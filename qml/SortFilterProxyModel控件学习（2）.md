### SortFilterProxyModel基于QML的QSortFilterProxyModel控件学习（二）

#### 目标问题

在qml实现了排序和筛选过程后，新的需求需要将筛选后的数据进行导出。首先是在数据模型中新增索引，原来的索引不能一一对应，在QSortFilterProxyModel中有事件可以将在count变化的时候发出信号，但存在的问题是，这个countChanged事件会根据每次减少的数量，或者增加的数量而反复执行。所以只能再源码中进行新的信号定义。

```c++
//CPreviewDataModel.cpp 建立索引关系
void CPreviewDataModel::clearShowlistindex(){
    m_showlistindex.clear();
}

void CPreviewDataModel::addShowlistindex(int p){
    m_showlistindex.insert(m_showlistindex.length(),p);
}

int CPreviewDataModel::getShowlistindex(int index) const{
    return  m_showlistindex.value(index,-1);
}

//countchanged  会由于删除的数据多少，来多次执行槽函数，如果我们这个槽函数中进行处理，会对性能有很大的损害
onCountChanged: {
	ModelMgr.previewModel().clearShowlistindex();
    for(var i=0;i<count;i++){
          ModelMgr.previewModel().addShowlistindex(get(i).dataindex_role)
    }
    console.log("onCountChanged" + count);
}

//原因是：源码中 由于信号与 与槽的绑定有一些问题 把每次rows的插入和删除 信号都传递给countchanged
connect(this, &QAbstractItemModel::rowsInserted, this, &QQmlSortFilterProxyModel::countChanged);
connect(this, &QAbstractItemModel::rowsRemoved, this, &QQmlSortFilterProxyModel::countChanged);

所以我们需要定义新的信号来处理这个问题
void filterEnding()//声明
    
在测试过程中发现，删选条件执行后  这三个函数都会被执行到
void queueInvalidateFilter();
void invalidateFilter();
Q_INVOKABLE int mapFromSource(int sourceRow) const; //常成员函数 不能对成员的变量进行修改


```

如果需要在mapFromSource中调用，需要将信号定义为一个const信号

```c++
void queueInvalidateFilter() const; //之前测试发现调用这个信号就报错，其实是函数内部访问数据的问题，信号可以发送，但是范围model数据被引起崩溃，具体的原因还没有调查明白，希望懂的同学告诉我一下。
```

#### 知识点学习

const 函数（常成员函数）：在常成员函数中不得修改类中的任何[数据成员](https://baike.baidu.com/item/数据成员/217930)的值，常成员函数我所理解的使用场景最好是在遍历访问堆中的指针变量的时候，最好用const函数。避免过程对数据进行修改，导致相互方位报错，和数据不一致等的bug。

qt中的信号是以什么样的方式存储到内存中的？[参见地址](https://blog.csdn.net/linux_wgl/article/details/33419409)

元对象编译器moc（meta object compiler）对C++文件中的类声明进行分析并产生用于初始化元对象的C++代码，**元对象包含全部信号和槽的名字以及指向这些函数的指针。** 这句话的意思，所信号与槽的本质是函数，我们声明了一个函数指针？？

搜索一圈发现，会用信号与槽的人很多，真正能讲清楚原理的人还是很少，抽空我在找本书来好好研究一下信号与槽的原理。并做一篇专门的博客！