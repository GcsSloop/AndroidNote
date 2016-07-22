# Matrix基础


前面讲了四篇 Path 相关的内容，本次终于要到了大家期盼已久的黑客帝国!

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)

## 一、Matrix作用

如题，本篇的主角是 Matrix，看起来大概是下面这样:

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

它在我们在之前的很多文章中都提及过，但并没有仔细的介绍过，从本篇开始终于要正式介绍它了，这个在2D和3D绘图中十分重要的角色Matrix(_Android中有两个Matrix，分别属于OpenGL 和 graphics， 本篇主要讲述的是graphics中的Matrix_)，

本篇的重点不在于这些方法的讲解，而是帮助大家理解 Matrix 的一些基本概念。

### Matrix 的翻译过来是矩阵，模型。和其释义相同，Matrix是一个矩阵，其作用则是一个模型，一个控制视图状态的模型。

**Matrix 作为一个模型，主要功能就是数值转换，将一个坐标系中的点映射到另一个坐标系中，举个栗子:**

我的的手机屏幕作为物理设备，其坐标系肯定是从左上角开始的，而且是不会改变的，但我们在开发的时候通常不会使用这一坐标系，而是使用内容区的坐标系，通常情况下，我们的内容区和屏幕坐标系还相差一个通知栏加一个标题栏的距离，所以两者是不重合的，我们在内容区的坐标系中的内容最终绘制的时候肯定要转换为实际的屏幕坐标系来绘制，Matrix在此处的作用就是转换这些数值。

>
假设通知栏高度为20像素，导航栏高度为40像素,那么我们在内容区的(0，0)位置绘制一个点，最终就要转化为在实际坐标系中的(0，60)位置绘制一个点。

>
我们也知道，Android中点视图是可以嵌套的树形结构，每一个子View都有自己的坐标系，但实际绘制时都应当转换为实际的屏幕坐标系才能正确的绘制，所以让每一个子View都有一个Matrix，这样每一个View就都拥有了转换坐标的能力，可以非常方便的将各自的坐标系中的内容映射到屏幕坐标系中。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f624vi3eb6j30rs0goab5.jpg)


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


**数值作用的介绍:**

>
根据名称我们就能猜到其大概作用，但有些效果是需要多个参数组合控制的，如下。

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
  8  | MPERSP_2 | perspective | 控制透视  (通常为1)

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


## 三、Matrix优缺点

Matrix在view、canvas、path、bitmap等很多地方都有应用，可以说运用十分广泛，优点突出，但也存在一定但缺点，下面简要介绍一下Matrix相关优缺点。

### 优点

* 计算方便，利用矩阵的计算法则
* 方便复用，Matrix仅有九个数值，却包含了大部分画布的状态信息，如果有重复的需要相同状态的地方，只需要构造好一个Matrix，然后设置给不同的画布即可，无需给每一个画布都重新构造状态，同样，如果一张图片需要在不同的地方以状态(镜像，旋转)展示，可以只放一张资源图片，然后构造出不同的Matrix进行设置即可。
* 




Matrix 是一个矩阵，肯定会涉及到一些比较麻烦的理论知识，不过我会尽量用通俗易懂的方式来帮助大家理解它。

## 四大常用操作

我们之前在 [Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 中讲解过的四种画布操作(translate, scale, rotate, skew)，这些操作的核心就是改变Matrix的数值，接下来我们看看这四种操作都会影响到哪些数值。














## Matrix方法表

Matrix 有很多常用和不常用的方法，

方法类别   | 相关API                                                 | 摘要
-----------|---------------------------------------------------------|------------------------
基本方法   | equals hashCode toString toShortString                  | 比较、 获取哈希值、 转换为字符串
数值操作   | set reset setValues getValues                           | 设置、 重置、 设置数值、 获取数值                    
设置(set)  | setConcat setRotate setScale setSkew setTranslate       | 设置变换
前乘(pre)  | preConcat preRotate preScale preSkew preTranslate       | 前乘变换
后乘(post) | postConcat postRotate postScale postSkew postTranslate  | 后乘变换
数值计算   | mapPoints mapRadius mapRect mapVectors                  | 计算变换后的数值
特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos     | 一些特殊操作
矩阵相关   | invert isAffine isIdentity                              | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ...















