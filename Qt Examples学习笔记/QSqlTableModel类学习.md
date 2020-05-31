### QSqlTableModel类学习

[本地源码地址](%qtpath%\Examples\Qt-5.14.2\sql\sql.pro)

#### 创建

```c++
model = new QSqlTableModel(this); //创建
```

#### 基本设置

```c++
void initializeModel(QSqlTableModel *model)
{
    //设置数据库表 不从表中选择数据，而是获取其字段信息。
    model->setTable("person"); 
    //描述在数据库中编辑值时选择的策略。
    //QSqlTableModel::OnFieldChange:对模型的所有更改都将立即应用到数据库。
    //QSqlTableModel::OnRowChange:当用户选择另一行时，将应用对该行的更改。
    //QSqlTableModel::OnManualSubmit:所有更改都将缓存在模型中，直到调用submitAll()或revertAll()。
    model->setEditStrategy(QSqlTableModel::OnManualSubmit);
    /*使用指定的筛选器和排序条件，使用通过setTable()设置的表中的数据填充模型，如果成功，则返回true;否则返回false。*/
    model->select();
    
    //setHeaderData 继承QSqlQueryModel::setHeaderData方法——重新实现从QAbstractItemModel: setHeaderData ()。
    //注意，这个函数不能用于修改数据库中的值，因为模型是只读的。
    model->setHeaderData(0, Qt::Horizontal, QObject::tr("ID"));
    model->setHeaderData(1, Qt::Horizontal, QObject::tr("First name"));
    model->setHeaderData(2, Qt::Horizontal, QObject::tr("Last name"));
}
```

#### 利用事务提交数据

```c++
  void TableEditor::submit()
 {
     //QSqlDatabase::transaction() 开启事务
     model->database().transaction();
     //提交所有挂起的更改并在成功时返回true。 
      if (model->submitAll()) {
          model->database().commit(); //提交
      } else {
          model->database().rollback();//回滚
          QMessageBox::warning(this, tr("Cached Table"),
                               tr("The database reported an error: %1")
                               .arg(model->lastError().text()));
      }
  }
```

#### 使用筛选功能

```c++
void MainWindow::changeArtist(int row)
{
    if (row > 0) {
        //relationModel(2)返回的是 QSqlRelationalTableModel
        QModelIndex index = model->relationModel(2)->index(row, 1);
        // 将当前筛选器设置为筛选。
        // 过滤器是一个SQL WHERE子句，没有关键字WHERE(例如，name='Josephine')。
        model->setFilter("artist = '" + index.data().toString() + '\'') ;
        showArtistProfile(index);
    } else if (row == 0) {
        model->setFilter(QString());
        showImageLabel();
    } else {
        return;
    }
}
```

使用排序

```c++
model->sort(1，Qt::AscendingOrder) //设置第一列 升序 排序
```

