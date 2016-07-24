# Matrix基础篇


## 前言

前面讲了四篇 Path 相关的内容，经过长时间的拖更，终于要到了大家期盼已久的 Matrix， 一个与黑客帝国同名的东东，看名字就知道很不凡，接下来我们就看看这个 Matrix 到底是何方神圣。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)


******

## 一、Matrix简介

如题，本篇的主角是 Matrix，其实和黑客帝国并没有太大关系，它看起来大概是下面这样:

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

### Matrix概念： Matrix的翻译过来是矩阵，模型。和其释义相同，Matrix是一个矩阵，也一个控制视图状态的模型，主要功能是坐标映射，数值转换。

我的的手机屏幕作为物理设备，其坐标系是从左上角开始的，但我们在开发的时候通常不会使用这一坐标系，而是使用内容区的坐标系。

以下图为例，我们的内容区和屏幕坐标系还相差一个通知栏加一个标题栏的距离，所以两者是不重合的，我们在内容区的坐标系中的内容最终绘制的时候肯定要转换为实际的屏幕坐标系来绘制，Matrix在此处的作用就是转换这些数值。

>
假设通知栏高度为20像素，导航栏高度为40像素,那么我们在内容区的(0，0)位置绘制一个点，最终就要转化为在实际坐标系中的(0，60)位置绘制一个点。


![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f624vi3eb6j30rs0goab5.jpg)

以上是仅以2D空间作为例子，然而我们在实际的软件开发过程中，为了有较好的空间层次感，可能都要求有一些3D效果，将3D效果的3维影像投影到2维的屏幕，也是依靠Matrix的转换。

## 二、Matrix基本原理

Matrix 本质是一个 3x3 的矩阵，里面有9个数值，分别用于控制视图的不同属性，大致如下:

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


序号 | 名称     | 对应单词    | 摘要
-----|----------|-------------|--------------
  0  | MSCALE_X | scale       | 控制X坐标 缩放，旋转
  1  | MSKEW_X  | skew        | 控制X坐标 错切，旋转
  2  | MTRANS_X | transfer    | 控制X坐标 位移
  3  | MSKEW_Y  | skew        | 控制Y坐标 错切，旋转
  4  | MSCALE_Y | scale       | 控制Y坐标 缩放，旋转
  5  | MTRANS_Y | transfer    | 控制Y坐标 位移
  6  | MPERSP_0 | perspective | 控制透视  (绕Y轴旋转)
  7  | MPERSP_1 | perspective | 控制透视  (绕X轴旋转)
  8  | MPERSP_2 | perspective | 控制透视  (齐次坐标标志位，通常为1)

从上表中可以看出一些内容，下面我们看一下2D画布中常用的四种操作(translate, scale, rotate, skew)都是由哪些参数控制的。

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

Matrix 是一个矩阵，肯定会涉及到一些比较麻烦的理论知识，我会尽量用通俗易懂的方式来帮助大家理解它。

我们先简单的理解几个概念和其作用。

### 1.齐次坐标系

* 作用: 方便区分坐标和向量，方便进行仿射变换。
* 摘要: 在数学中我们的点和向量都是这样表示的(x, y)，两者看起来一样，我们人可以根据上下文信息区分这是点还是向量，而计算机则无法区分，为此我们增加了一个标志位来让计算机也可以区分它们，增加之后看起来是这样: <br/>
**点(x : y : 1) － 向量(x : y : 0)**<br/>
你可能注意到了，我将分隔符换成了冒号，这是因为齐次坐标具有等比的性质，(2:3:1)、(4:6:2)...(2N,3N,N)表示的均是(2,3)这一个点。(**这也是为什么会产生将MPERSP_2解释为scale这一误解了**)

### 2.仿射变换

* 作用: **仿射变换其实是线性变换和平移变换的叠加。**我们之前了解过的 缩放、旋转、错切 本质上都属于线性变换。对于我们而言，仿射变换对应的就是常见的四种画布操作(缩放、旋转、错切、平移)。
* 摘要: 我们之前说过，Matrix主要作用就是坐标的映射，仿射变换主要就是做这个工作的，详情请继续往下看。

### 3.线性变换

线性变换主要有3种: 缩放(scale)、旋转(rotate) 和 错切(skew)。

线性变换均可用矩阵乘法完成(不了解矩阵乘法规则的参考下面维基百科)，下面我们就来讲讲这几种操作。

**[维基百科-矩阵乘法](https://zh.wikipedia.org/wiki/%E7%9F%A9%E9%99%A3%E4%B9%98%E6%B3%95)**

#### a.缩放

以点(10,10)为例，我们将x缩放到原来到0.5倍，y缩放到原来到2倍，我们可以轻易到算出结果:(10x0.5, 10x2) = (5, 20)

矩阵计算公式:

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


用计算示例:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
5 \\\\
20
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
0.5 & 0 \\\\
0 & 2 
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
10\\\\
10
\\end{1} 
\\right ]
$$)

#### b.错切





### 四大常用操作

我们之前在 [Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 中讲解过的四种画布操作(translate, scale, rotate, skew)，这些操作的核心就是改变Matrix的数值，接下来我们看看这四种操作都会影响到哪些数值。














## Matrix方法表

Matrix 有很多常用和不常用的方法，

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















