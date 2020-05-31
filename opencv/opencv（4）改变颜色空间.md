### opencv的颜色空间相互转化

HSV：色调（H），饱和度（S），明度（V） RGB：红色（R）， 绿色（G），蓝色（B）
CMY：青（Cyan）、洋红（或品红）（Magenta）和黄（Yellow）加上黑色Black 为 CMYK

RGB和CMY颜色模型都是面向硬件的，而HSV（Hue Saturation Value）颜色模型是面向用户的

#### RGB转化到HSV的算法

```c++
# COLOR_BGR2HSV 
max=max(R,G,B)；
min=min(R,G,B)；
V=max(R,G,B)；
S=(max-min)/max；
if (R = max) H =(G-B)/(max-min)* 60；
if (G = max) H = 120+(B-R)/(max-min)* 60；
if (B = max) H = 240 +(R-G)/(max-min)* 60；
if (H < 0) H = H+ 360；
```

#### HSV转化到RGB的算法

```c++
# COLOR_HSV2BGR
if (s = 0)
	R=G=B=V;
else
	H /= 60;
i = INTEGER(H);
f = H - i;
a = V * ( 1 - s );
b = V * ( 1 - s * f );
c = V * ( 1 - s * (1 - f ) );
switch(i)
	case 0: R = V; G = c; B = a;
	case 1: R = b; G = v; B = a;
	case 2: R = a; G = v; B = c;
	case 3: R = a; G = b; B = v;
	case 4: R = c; G = a; B = v;
	case 5: R = v; G = a; B = b;
```

#### RGB和GRAY的相互转化方法

```python
# COLOR_BGR2GRAY
GRAY = B * 0.114 + G * 0.587 + R * 0.299
# COLOR_GRAY2BGR
R = G = B = GRAY; A = 0;
```
#### 在matplotlib 和cv.imread的使用不同的颜色空间

matplotlib中使用的是RGB空间颜色，而cv.imread默认读取的是颜色空间为BGR

```python
COLOR_BGR2RGB
COLOR_RGB2BGR
```

