### Views 样例学习笔记

[Examples地址](<安装路径下的examples>\quick\views)

#### QML如何动态添加条目到Listview中

定义ListModel

```qml
//值得注意的是这个里面有两层 数据关系 一个是`fruitmodel` 一个是`attributes`
ListModel {     
        id: fruitModel
        ListElement {
            name: "Apple"; cost: 2.45
            attributes: [
                ListElement { description: "Core" },
                ListElement { description: "Deciduous" }
            ]
        }
}
```

动态添加一个条目 直接在定义的ListModel 中使用append方法,按照之前定义的`ListElement`模板将参数填入进去 

```qml
fruitModel.append({
                    "name": "Pizza Margarita",
                    "cost": 5.95,
                    "attributes": [{"description": "Cheese"}, {"description": "Tomato"}]
                })
               
 fruitModel.clear() //可以清空所有的条目
 fruitModel.move(index, index-1, 1) //移动操作 向上移动一个
 fruitModel.move(index, index+1, 1) //向下移动一下
 
```

针对上面的`listmodel`二级数据结构，通过`Repeater`来进行数据分层，通过`setProperty`设置属性，从而达到需要实现的具体内容的修改

```qml
Row {
    anchors.horizontalCenter: parent.horizontalCenter
    spacing: 5
    Repeater { //
        model: attributes /这个地方处理了数据源的第二次数据
        Text { text: description; color: "red" }
    }
}
//修改listmodel中原来数据的值   “显示的，和实际内存的是否都被修改呢？？”
fruitModel.setProperty(index, "cost", Math.max(0,cost-0.25))             
```

#### `pathview`沿着特定路线显示`items`

 Path 的属性 startX 、 startY 描述路径起点。 pathElements 属性是个列表，是默认属性，它保存组成路径的多个路径元素，常见的路径元素有 PathLine 、 PathQuad 、 PathCubic 、 PathArc 、 PathCurve 、 PathSvg 。路径上最后一个路径元素的终点就是整个路径的终点，如果终点与起点重合，那么 Path 的 closed 属性就为 true 。

```qml
path: Path {
    startX: 10
    startY: 50
    PathAttribute { name: "iconScale"; value: 0.5 }
    PathQuad { x: 200; y: 150; controlX: 50; controlY: 200 }
    PathAttribute { name: "iconScale"; value: 1.0 }
    PathQuad { x: 390; y: 50; controlX: 350; controlY: 200 }
    PathAttribute { name: "iconScale"; value: 0.5 }
}
// PathQuad 元素定义一条二次贝塞尔曲线作为路径段。它的起点为上一个路径元素的终点（或者路径的起点），终点由 x 、 y 或 relativeX 、 relativeY 定义，控制点由 controlX 、 controlY 或 relativeControlX 、 relativeControlY 来定义.
//PathAttribute 定义一些属性，它的声明语法类似下面：
  PathAttribute { name: "zOrder"; value: 0.2; }
  name 属性指定待定义属性的名字， real 类型的 value 属性的值为待定义的属性的值。

```

[参见链接]( https://blog.csdn.net/foruok/article/details/38060495 )

`Pathline`代码实现

```qml
 Path {
        startX: 0;
        startY: 0;
        PathQuad {
            x: root.width - 1;
            y: root.height - 1;
            controlX: 0;
            controlY: root.height - 1;
        }
 }
 
 //第二段
 Path {
     startX: 10;
     startY: 100;
     PathLine {
         x: root.width/2 - 40;
         y: 100;	
     }		
     PathPercent { value: 0.28; }
     PathLine {
         relativeX: root.width/2 - 60;
         relativeY: 0;
     }
 }
  PathPercent 放在组成路径的元素后面，比如放在 PathLine 后面，指明它前面的那部分路径（通常由一个或多个 Path 元素组成）所放置的 item 数量占整个路径上所有 item 数量的比率。
    PathPercent 的 value 属性为 real 值，范围 0.0 至 1.0 。需要注意的是，在一个 Path 中使用 PathPercent ，PathPercent 元素的 value 值是递增的，某一段路径如果在两个 PathPercent 之间，那么这段路径上面放置的 item 数量占路径上总 item 数量的比率，是后面的 PathPercent 与 前面的 PathPercent 的 value 之差。
```

<font color=red>扩展一个相关与图像处理的博客地址，下次有时间的时候后在深度学习这篇博客[图像处理链接]( https://blog.csdn.net/foruok/article/details/37740583 )</font>

PathCubic : PathCubic 定义一个三次方贝塞尔曲线，它有两个控制点。 

PathArc： 路径元素定义一条弧线，它的起点为上一个路径元素的终点（或者路径的起点），终点由 x 、 y 或 relativeX 、 relativeY 定义。弧线的半径由 radiusX 、 radiusY 定义。 

PathCurve ：提供了一种简单的方法来指定一条直接通过一组点的曲线。通常在一个系列中使用多个路径曲线 

####  model/view 学习

在SmallTalk中有一个经典的设计模式-MVC。即模型-视图-控制器，在qml中将control改成了delegate(委托)，也就是现在的Model-View-Delegate.换了个说法，Model还是负责数据，View管着视图输出，Delegate呢就是一个介于视图和数据之间的桥梁。 