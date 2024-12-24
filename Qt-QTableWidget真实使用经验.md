## 有着几点以前不知道
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
