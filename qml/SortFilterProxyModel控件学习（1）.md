### SortFilterProxyModel基于QML的QSortFilterProxyModel控件学习（一）

#### 应用背景

现有的项目中已经实现了`model/view`,在此基础上，为了不对代码进行大规模的修改，第二由于本身对于重新去搭建新的SortFilterProxyModel，c++开发时间成本比较大。所以在GitHub上找到了这份基于`QML`实现QSortFilterProxyModel的控件进行一个学习和使用下载是[源代码地址](http://www.github.com/oKcerG/SortFilterProxyModel) [Dome地址](https://github.com/oKcerG/SFPMShowcase)

#### 引用方法

克隆下载存储库，在工程文件.pro中 添加 include (<path/to/SortFilterProxyModel>/SortFilterProxyModel.pri)
在使用的qml 中添加import SortFilterProxyModel 0.2 

#### 学习内容和实践内容

第一步：dome替换:将原来model修改，替换成新的看是否能成功（<font color = red >成功</font>）

```qml
model: ModelMgr.previewModel()
```

修改为

```javascript
model: sortfilterproxymodel
//
SortFilterProxyModel{
    id:sortfilterproxymodel
    sourceModel: ModelMgr.previewModel()

}
```

第二步，添加排序 只能实现默认排序（<font color = red >成功</font>）

```js
        sorters: [
            StringSorter{ roleName: "name_role" }
        ]
```

第三步：排序切换升序和降序（<font color = red >成功</font>）

```javascript
// header增加点击事件
MouseArea{
    anchors.fill: parent
    onClicked: {
        index = 0
        test_sort()
    }
}
//在js中去处理 
function test_sort()
{
    ascend = !ascend
    switch (index) {
        case 0 :
            id_sort0.sortOrder = ascend ? Qt.AscendingOrder : Qt.DescendingOrder
            break
        default:
        	break
    }
}
```

第四步 切换不同列的排序  （<font color = red >成功</font>）

```javascript
    /*方法 一 */ 这个方法的扩展性是很弱的，必须要找到更为合理的方法
    //通过enabled的设置 来变化设置sorts的有效性
    function test_sort()
    {
        ascend = !ascend
        switch (index) {
          case 0 :
              id_sort0.enabled = true;
              id_sort1.enabled = false
              id_sort0.sortOrder = ascend ? Qt.AscendingOrder : Qt.DescendingOrder
              break
          case 1 :
              id_sort0.enabled = false
              id_sort1.enabled = true
              id_sort1.sortOrder = ascend ? Qt.AscendingOrder : Qt.DescendingOrder

              break
          default:
              break
        }
    }
```

第五步 基本搜索功能（<font color = red >成功</font>）

```javascript
 Rectangle{
       width: id_table_view.width
       x:0
       y:0
       height:40
       color:"red"

       TextField{
          id:id_text
       }
    }
 //备注说明：之前将Rectangle 放在    Component 中的变化为什么不能关联到  filter里面呢？？这个我需要查一下  
filters: [
            AnyOf{
                RegExpFilter {
                    id:id_filtertime
                    roleName: "name_role"
                    pattern: "*" + id_text.displayText+"*"
                    caseSensitivity: Qt.CaseInsensitive
                    syntax: RegExpFilter.Wildcard
                }
            }
]   
```

第五步：  多条件是筛选 这个地方就是对于JS的正则表达式的使用 [学习地址](https://www.runoob.com/jsref/jsref-obj-regexp.html)

```qml
eg:(png$|jpg$) //筛选后缀为png 或者jpg的文件
```

第六步：区间值显示

```javascript
 RangeFilter {
     id:id_filtertRange
     roleName: "width_role"
     minimumValue:1000
     maximumValue:2000
     enabled: false
 }
```

第七步：获取筛选的数据的新数据

```javascript
        //在发生筛选的时候  这个事件会被触发
        //这个时候来遍历想获取的数据 在实际测试过程中这个地方会被多次调用，还在尝试对源码进行修改解决问题的过程中
        onCountChanged: {
            console.log("onCountChanged")
            console.log(data)
            for(var i = 0; i < count; i++){
               console.log(proxyModel.get(i).firstName)
            }
        }
```

在使用过程中会涉及数据源和筛选数据数据之间，不能一一对应的问题。我将在下一篇博客中进行解决和处理