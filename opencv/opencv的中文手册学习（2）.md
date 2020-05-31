### opencv的中文手册学习（二）

#### 学习内容和目标

参见 [2_2_视频入门](http://woshicver.com/ThirdSection/2_2_视频入门/) ，原文总比我这种断章取义讲的更加的清楚，所以我附上学习的原文地址。今天的学习课题没有我喜欢的练习题，表示很遗憾。只能进行一些代码的复制和修改，再做一些知识点的补充了，只能说这篇博客是方便我以后自己查看。

```python
# import numpy as np
import cv2 as cv
cap = cv.VideoCapture(0)
if not cap.isOpened():
    print("Cannot open camera")
    exit()

while True:
    # 逐帧捕获
    ret, frame = cap.read()
    # 如果正确读取帧，ret为True
    if not ret:
        print("Can't receive frame (stream end?). Exiting ...")
        break
    # 我们在框架上的操作到这里
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    # 显示结果帧e
    cv.imshow('frame', gray)
    if cv.waitKey(1) == ord('q'):
        break
# 完成所有操作后，释放捕获器
cap.release()
cv.destroyAllWindows()

```

为什么要复制这段代码呢，原因很简单，我一般都是先把代码复制到我的`vscode`中，然后直接运行查看效果，当发现没有运行通过的时候，就通过解决这些不能通过的问题以达到学习的目的，当我运行这段代码的时候发现的第一个问题尽然是我的摄像头被我禁用了，在这个地方我还是简单的介绍一下 `win10` 怎么打开摄像头的操作（设置->隐私->相机-> 打开访问权限就可以了），之前是因为我为了降低系统的消耗，特地关闭了的。

第二个问题就是发现怎么，打开后的图像是灰色的，发现这个地方必须要解释一下下面这行代码

```python
# cvtColor函数是OpenCV里用于图像颜色空间转换，可以实现RGB颜色、HSV颜色、HSI颜色、lab颜色、YUV颜色等转换，也可以
# 彩色和灰度图互转。这个地方的cv.COLOR_BGR2GRAY就是BGR颜色到 GRAY（灰度图）的转化
gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY) 
cv.imshow('frame', gray)
# 然后我这边直接修改一下显示的参数  就变成彩色的了
cv.imshow('frame', frame) 
```

[cvtColor函数的具体学习地址](https://zouzhongliang.com/index.php/2019/08/19/opencv-yansekongjianzhuanhuanhanshucvtcoloryunyong/) 里面可以查看更为的参数使用方式。

```python
import numpy as np
import cv2 as cv
cap = cv.VideoCapture(0)
# 定义编解码器并创建VideoWriter对象
fourcc = cv.VideoWriter_fourcc(*'XVID')
out = cv.VideoWriter('output.avi', fourcc, 20.0, (640,  480))
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        print("Can't receive frame (stream end?). Exiting ...")
        break
    frame = cv.flip(frame, 0)
    # 写翻转的框架
    out.write(frame)
    cv.imshow('frame', frame)
    if cv.waitKey(1) == ord('q'):
        break
# 完成工作后释放所有内容
cap.release()
out.release()
cv.destroyAllWindows()
```

这段代码中，想先说明一下的就是`fourcc`（四字符代码）[对照表地址](https://www.fourcc.org/codecs.php) ，在学习这个之前，建议如果是新学习者先去查阅两个概念 ` 容器格式`和 `编码格式` 的概念。在实际真正涉及到这两块的内容，我会在我的博客中详细的介绍。如果有兴趣的同学可以先通过搜索引擎解决自己的疑惑。比较常用的`fourcc`我在这个地方列举几个：XVID，MPG4，DIV2，FLV1 ...... 

只说以要说一下`fourcc`，是因为我们代码中涉及到`cv.VideoWriter` 视频的输出，第一个参数中的后缀名字，avi就是 容器格式，`XVID`就是说保存视频的编码格式。

最后提一下这个函数`cv::flip`()支持图像的翻转（上下翻转、左右翻转，以及同时均可）

```python
horizontal = cv.flip(img,1) 水平镜像
vertical = cv.flip(img,0) 垂直镜像
cross = cv.flip(img,-1) 对角镜像
```

