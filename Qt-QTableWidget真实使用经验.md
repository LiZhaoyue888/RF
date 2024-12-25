## QTableWidget有几个以前不知道的实用知识点
- 如何设置表头
  ```c++
    QStringList headerLabels;
    headerLabels << "State" << "Start" << "Stop" << "Points" << "IFBW";
    setHorizontalHeaderLabels(headerLabels);
  ```
- 规定一次只能选中一个单元格
  ```c++
    setSelectionMode(QAbstractItemView::SingleSelection);//一次只能选中一个单元格
    setSelectionBehavior(QAbstractItemView::SelectItems);//不能通过点击行列选中单元格
  ```
- 表格的QTableWidgetItem可以设置为CheckBox，唯一缺点无法居中
  ```c++
    QTableWidgetItem* checkItem = new QTableWidgetItem();
    checkItem->setFlags(Qt::ItemIsUserCheckable | Qt::ItemIsEnabled);
    checkItem->setCheckState(seg.state ? Qt::Checked : Qt::Unchecked);
    setItem(index, 0, checkItem);
  ```
- 清空表格所有行
  ```c++
    clearContents();
    setRowCount(0);
  ```

## 关于为什么没有选择MVD

**需求背景：**
在编辑表格时，手动修改表格中的cell时，这个cell显示扫描频率。当我要输入时不带单位，期望输入后自动补全单位。就比如按下输入1按回车键，表格应该改为1 Hz这样，输入1e9自动变为1 GHz  
**尝试过程：**  
- ChatGPT让我绑定itemChanged() 以为很蠢，因为这根本不是一个信号而是虚函数，并且不区分代码修改还是用户输入**后面会反转**  
- 然后尝试绑定commitData()这个信号，发现根本不是一个信号而是一个protected Q_SLOTS 虚函数,需要自己重写，遂放弃**后面会反转**
- 然后尝试用MVD,重写QStyledItemDelegate的setModelData方法，然后给QTableWidget设置Delegate。这个方法很有效，决定最终采用，但是把过程中的代码浮上来
当我用MVD尝试OK之后，回过头来写文档，要不在试试前两种方法，最终发现都可以。这其中有个关键的两点，就是itemChanged不是说编辑的同时发射信号，不会。第二个点就是，不用区分是代码修改值还是手动修改的，isSelected()判断一下是不是我正在修改的cell，然后把当前编辑的单元格的文本加单位就行。
然后还发现，重写commitData也能实现，不过要拿到currentIndex，然后判断isValid()然后再修改编辑器的值就行。那么顺序就是：编辑完成，先走commitData()然后再走itemChanged()，性能上肯定是commitData()好，因为itemChanged信号触发太过频繁，添加删除行也会触发，代码修改值也会触发，可能导致递归。
**一下是三种代码对比：**
- 绑定itemChanged信号
  ```c++
  connect(this, &QTableWidget::itemChanged, this, & SegmentTable::slot_temChanged);
  void SegmentTable::slot_temChanged(QTableWidgetItem* item)
  {
      if (item->isSelected())
      {
          QString text = item->text();
  
          if (item->column() == 1 || item->column() == 2)
          {
              double realValue = ModelService::getRealValueWithUnit(text);
              QString niceValue = ModelService::getValueWithNiceUnit_Freq(realValue);
              item->setText(niceValue);
          }
      }
  }
  ```
- 重写commitData
```c++
void commitData(QWidget* editor)override;
void SegmentTable::commitData(QWidget* editor)
{
    QModelIndex index = this->currentIndex();
    int column = index.column();
    if (index.isValid())
    {
        QString originalText = qobject_cast<QLineEdit*>(editor)->text();
        if (column == 1 || column == 2)
        {
            double realValue = ModelService::getRealValueWithUnit(originalText);
            QString niceValue = ModelService::getValueWithNiceUnit_Freq(realValue);
            qobject_cast<QLineEdit*>(editor)->setText(niceValue);
        }
    }
    QAbstractItemView::commitData(editor);
}
```
- 采用MVD  
头文件.h代码  
```c++

class SegmentDelegate : public QStyledItemDelegate
{
    Q_OBJECT
public:
    explicit SegmentDelegate(QObject* parent = nullptr);

    // 创建编辑器（默认QLineEdit或其他）
    QWidget* createEditor(QWidget* parent,
                          const QStyleOptionViewItem &option,
                          const QModelIndex &index) const override;

    // 将模型数据写入编辑器
    void setEditorData(QWidget* editor,
                       const QModelIndex &index) const override;

    // 将编辑器数据写回模型
    void setModelData(QWidget* editor,
                      QAbstractItemModel* model,
                      const QModelIndex &index) const override;

    // 设置编辑器几何
    void updateEditorGeometry(QWidget* editor,
                              const QStyleOptionViewItem &option,
                              const QModelIndex &index) const override;
};

```
源文件.cpp代码 重点关注 setModelData()  createEditor()
```c++

SegmentDelegate::SegmentDelegate(QObject* parent)
    : QStyledItemDelegate(parent)
{
}

QWidget* SegmentDelegate::createEditor(QWidget* parent,
                                       const QStyleOptionViewItem &option,
                                       const QModelIndex &index) const
{
    static QStringList buttonTextList = { "1MHz", "500KHz", "250KHz", "125KHz", "60KHz", "20KHz", "15KHz", "7KHz", "4KHz", "2KHz", "1KHz" };
    if (index.column() == 4)//ifbw
    {
        auto combox = new QComboBox(parent);
        combox->addItems(buttonTextList);
        return combox;
    }
    return QStyledItemDelegate::createEditor(parent, option, index);
}

void SegmentDelegate::setEditorData(QWidget* editor,
                                    const QModelIndex &index) const
{
    // 先让基类把模型数据设置到编辑器，如 QLineEdit
    QStyledItemDelegate::setEditorData(editor, index);
}
//修改编辑器，触发更新模型数据
void SegmentDelegate::setModelData(QWidget* editor,
                                   QAbstractItemModel* model,
                                   const QModelIndex &index) const
{
    QLineEdit* lineEdit = qobject_cast<QLineEdit*>(editor);
    if (!lineEdit)
    {
        QStyledItemDelegate::setModelData(editor, model, index);
        return;
    }

    QString text = lineEdit->text();
    if (index.column() == 1 || index.column() == 2)
    {
        double realValue = ModelService::getRealValueWithUnit(text);
        QString niceValue = ModelService::getValueWithNiceUnit_Freq(realValue);
        model->setData(index, niceValue, Qt::EditRole);
    }
    else
    {
        model->setData(index, text, Qt::EditRole);
    }
}

void SegmentDelegate::updateEditorGeometry(QWidget* editor,
        const QStyleOptionViewItem &option,
        const QModelIndex &index) const
{
    Q_UNUSED(index);
    editor->setGeometry(option.rect);
}
//构造函数，重设Delegate
SegmentTable::SegmentTable(QWidget* parent /*= nullptr*/)
    : QTableWidget(parent)
{
    initUI();
    buildConnection();

	SegmentDelegate* delegate = new SegmentDelegate(this);
	setItemDelegate(delegate);
}
```
### 一个巧妙地Qt设定
当我用Delegate的createEditor给QTableWidget的某一列editor改为QComboBox时，我只改了editor，没有修改setEditorData和setModelData，神奇的事情发生了。  
表格中的数据可以根据QComboBox的选择自动更新，我本来还打算往setEditorData中添加代码，所以ChatGPT的愚蠢不能全信，即便是ChatGPT-o1。  
这个只修改createEditor不修改其他项的神奇之处就在于以下代码，其中关键是：editor->metaObject()->userProperty().name()
```c++
void QStyledItemDelegate::setEditorData(QWidget *editor, const QModelIndex &index) const
{
#ifdef QT_NO_PROPERTIES
    Q_UNUSED(editor);
    Q_UNUSED(index);
#else
    QVariant v = index.data(Qt::EditRole);
    QByteArray n = editor->metaObject()->userProperty().name();

    if (!n.isEmpty()) {
        if (!v.isValid())
            v = QVariant(editor->property(n).userType(), (const void *)0);
        editor->setProperty(n, v);
    }
#endif
}
```
