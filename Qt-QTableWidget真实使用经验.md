## 有着几点以前不知道
- 如何设置表头
  ```c++
    QStringList headerLabels;
    headerLabels << "State" << "Start" << "Stop" << "Points" << "IFBW";
    setHorizontalHeaderLabels(headerLabels);
  ```
- 表格可以规定一次编辑几个单元格
  ```c++
    setSelectionMode(QAbstractItemView::SingleSelection);
    setSelectionBehavior(QAbstractItemView::SelectItems);
  ```
- 表格的QTableWidgetItem可以设置为QCheckBox替代setWidget(new QCheckBox) 方法：setFlags
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
