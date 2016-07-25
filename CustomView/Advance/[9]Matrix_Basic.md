# Matrix基础篇


## 前言

前面讲了四篇 Path 相关的内容，经过长时间的拖更，终于要到了大家期盼已久的 Matrix， 一个与黑客帝国同名的东东，看名字就知道很不凡，接下来我们就看看这个 Matrix 到底是何方神圣。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)

******

## 一、Matrix简介

**如题，本篇的主角是 Matrix，其实和黑客帝国并没有太大关系, Matrix是一个矩阵，也一个控制视图状态的模型，主要功能是坐标映射，数值转换。**

它看起来大概是下面这样:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
MSCALE\\_X & MSKEW\\_X & MTRANS\\_X \\\\
\\\\
MSKEW\\_Y & MSCALE\\_Y & MTRANS\\_Y \\\\
\\\\
MPERSP\\_0 & MPERSP\\_1 & MPERSP\\_2 
\\end{1} 
\\right ] 
$$)

**为了帮助大家理解Matrix的作用，在这里简单的举一个例子:**

我的的手机屏幕作为物理设备，其坐标系是从左上角开始的，但我们在开发的时候通常不会使用这一坐标系，而是使用内容区的坐标系。

以下图为例，我们的内容区和屏幕坐标系还相差一个通知栏加一个标题栏的距离，所以两者是不重合的，我们在内容区的坐标系中的内容最终绘制的时候肯定要转换为实际的屏幕坐标系来绘制，Matrix在此处的作用就是转换这些数值。

>
假设通知栏高度为20像素，导航栏高度为40像素,那么我们在内容区的(0，0)位置绘制一个点，最终就要转化为在实际坐标系中的(0，60)位置绘制一个点。


![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f624vi3eb6j30rs0goab5.jpg)

以上是仅以2D空间作为例子，然而我们在实际的软件开发过程中，为了有较好的空间层次感，可能都要求有一些3D效果，将3D效果的3维影像投影到2维的屏幕，也是依靠Matrix的转换。

下面我们看一下2D画布中常用的四种操作(translate, scale, rotate, skew)都是由哪些参数控制的。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f60gwrhlnyj30c008zdgy.jpg)
![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f633hvklfnj30c008zdge.jpg)

>
**从上图可以看到最后三个参数是控制透视的，这三个参数主要在3D效果中运用，通常为(0, 0, 1)，不在本篇范围内，暂不过多叙述，会在之后对文章中详述其作用。**

### 常见误解

在写本文之前，我翻阅很多介绍 Matrix 的文章和官方文档，但其中文的搜索结果令我很悲伤，大部分的中文文章对Matrix都存在误解,想当然的创造出一些错误的理论, 导致很多(抄袭)的文章都是这一错误的理论，不知道要坑害多少小白，常见的错误理论有:

**1.认为Matrix最下面的一行的三个参数(MPERSP_0、MPERSP_1、MPERSP_2)没有什么太大的作用，在这里只是为了凑数。**

> **实际上最后一行参数在3D变换中有着至关重要的作用，这一点会在后面中Camera一文中详细介绍。**

**2.最后一个参数MPERSP_2被解释为scale**

> **的确，更改MPERSP_2的值能够达到类似缩放的效果，但这是因为齐次坐标的缘故，并非这个参数的实际功能。**


## 三、Matrix详解

Matrix 是一个矩阵，最根本的作用就是坐标转换，下面我们就看看几种常见的变换原理:

常见的基本变换有4种: 平移(translate)、缩放(scale)、旋转(rotate) 和 错切(skew)。

由于我们以下大部分的计算都是基于矩阵乘法规则，如果你已经把线性代数还给了老师，请参考一下这里:
**[维基百科-矩阵乘法](https://zh.wikipedia.org/wiki/%E7%9F%A9%E9%99%A3%E4%B9%98%E6%B3%95)**

#### a.缩放

以点(10,10)为例，我们将x缩放到原来到0.5倍，y缩放到原来到2倍，我们可以轻易到算出结果:(10x0.5, 10x2) = (5, 20)

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
X\\\\
Y
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
Scale\\_X & 0 \\\\
0 & Scale\\_Y 
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
x\\\\
y
\\end{1} 
\\right ]
$$)


#### b.错切

错切有水平错切(平行X轴)和垂直错切(平行Y轴)，或者是两者叠加。

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
X\\\\
Y
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
1 & Skew\\_X \\\\
Skew\\_Y & 1 
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
x\\\\
y
\\end{1} 
\\right ]
$$)

#### b.旋转

逆时针旋转 a 度。

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
X\\\\
Y
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
cos(a) & -sin(a) \\\\
sin(a) & cos(a) 
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
x\\\\
y
\\end{1} 
\\right ]
$$)


### 4.平移变换

我们可以看到，在之前的示例中，用的都是 2 x 2 的矩阵，但我们实际的矩阵是 3 x 3 的，这是为什么呢？

前面的各种方法我们都是直接













**Matrix方法表**

方法类别   | 相关API                                                 | 摘要
-----------|---------------------------------------------------------|------------------------
基本方法   | equals hashCode toString toShortString                  | 比较、 获取哈希值、 转换为字符串
数值操作   | set reset setValues getValues                           | 设置、 重置、 设置数值、 获取数值
数值计算   | mapPoints mapRadius mapRect mapVectors                  | 计算变换后的数值
设置(set)  | setConcat setRotate setScale setSkew setTranslate       | 设置变换
前乘(pre)  | preConcat preRotate preScale preSkew preTranslate       | 前乘变换
后乘(post) | postConcat postRotate postScale postSkew postTranslate  | 后乘变换
特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos     | 一些特殊操作
矩阵相关   | invert isAffine isIdentity                              | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ...















