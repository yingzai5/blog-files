### opencv的中文手册学习（一）

#### 环境介绍

win10 + vscode + python3.7.6 + opencv 4.2.0 对于环境配置网上的介绍太多了，用python来搭建opencv的学习环境，来说是我觉得最为简单了。另外一方面也说明，我对内部的这些调用逻辑还是没有深入的理解，在以后有了扎实基础的时候这个地方我可能还需要重新去做一次

#### 解决问题

练习题：当你尝试在OpenCV中加载彩色图像并将其显示在Matplotlib中时，存在一些问题。阅读此讨论：http://stackoverflow.com/a/15074748/1134940)并理解它。

```python
import cv2 as cv
from matplotlib import pyplot as plt
img = cv.imread('image.jpg', 0)
plt.imshow(img, cmap='gray', interpolation='bicubic')
plt.xticks([]), plt.yticks([])  # 隐藏 x 轴和 y 轴上的刻度值
plt.show()

```

运行的时候回报如下的错误：TypeError: Image data of dtype object cannot be converted to float

原因在OpenCV和Matplotlib中，像素顺序略有不同。OpenCV遵循BGR顺序，而matplotlib可能遵循RGB顺序。

```python
b, g, r = cv.split(img)
img2 = cv.merge([r, g, b])
```

报错：ValueError: not enough values to unpack (expected 3, got 0)

网上解释说是由于没有使用绝对路径的问题 我将图片的路径修改为绝对路径后（D:\\code\\Python_Code\\opencv\\image\\1.jpg） 出现新的问题

ValueError: not enough values to unpack (expected 3, got 1)

将cv在读取的时候 将后面的一个参数去掉 代码变成如下就能正常运行

```python
import cv2 as cv
from matplotlib import pyplot as plt
img = cv.imread('D:/code/Python_Code/opencv/image/1.jpg')
[b, g, r] = cv.split(img)
img2 = cv.merge([r, g, b])
plt.imshow(img2, cmap='gray', interpolation='bicubic')
plt.xticks([]), plt.yticks([])  # 隐藏 x 轴和 y 轴上的刻度值
plt.show()
```

这到练习题主要涉及两个问题的处理

+ cv.imread() 添加的第二个参数的意义，和返回数据的影响（参加原理解释一）
+ OpenCV和Matplotlib中，像素顺序略有不同

#### 原理解释一

>Flags指定了所读取图片的颜色类型
>CV_LOAD_IMAGE_ANYDEPTH返回图像的深度不变。
>CV_LOAD_IMAGE_COLOR总是返回一个彩色图。
>CV_LOAD_IMAGE_GRAYSCALE总是返回一个灰度图。
>0返回3通道彩色图
>注意：alpha 通道将被忽略，如果需要alpha 通道，请使用负值
>=0返回灰度图
><0返回原图（带alpha 通道）

在之前的代码中，我们返回的是一张灰度图，然后我们去获取他的bgr值肯定有问题的。这个地方我们需要理解两个概念
灰度图：参见 [百度百科灰度计算方法](https://baike.baidu.com/item/灰度图/8105733?fr=aladdin#3)
通道分离（split函数的功能）：参见 [图像模式 ](https://baike.baidu.com/item/%E5%9B%BE%E5%83%8F%E6%A8%A1%E5%BC%8F)去理解 

