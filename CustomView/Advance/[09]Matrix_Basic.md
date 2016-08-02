# Matrix原理

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView/README.md)


## 目录

- [前言](#qianyan)
- [Matrix简介](#jianjie)
- [Matrix基本原理](#jiben)
- [Matrix复合原理](#fuhe)
- [Matrix方法表](#fangfa)
- [总结](#zongjie)
- [关于作者](#about)
- [参考资料](#ziliao)

<p id="qianyan" /> 
## 前言

本文内容偏向理论，和 [画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 有重叠的部分，本文会让你更加深入的了解其中的原理。

本篇的主角Matrix，是一个一直在后台默默工作的劳动模范，虽然我们所有看到View背后都有着Matrix的功劳，但我们却很少见到它，本篇我们就看看它是何方神圣吧。

>
由于Google已经对这一部分已经做了很好的封装，所以跳过本部分对实际开发影响并不会太大，不想深究的粗略浏览即可，下一篇中将会详细讲解Matrix的具体用法和技巧。

******

<p id="jianjie" /> 
## Matrix简介

<p id="gaishu" />
**Matrix是一个矩阵，主要功能是坐标映射，数值转换。**

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

**Matrix作用就是坐标映射，那么为什么需要Matrix呢? 举一个简单的例子:**

我的的手机屏幕作为物理设备，其物理坐标系是从左上角开始的，但我们在开发的时候通常不会使用这一坐标系，而是使用内容区的坐标系。

以下图为例，我们的内容区和屏幕坐标系还相差一个通知栏加一个标题栏的距离，所以两者是不重合的，我们在内容区的坐标系中的内容最终绘制的时候肯定要转换为实际的物理坐标系来绘制，Matrix在此处的作用就是转换这些数值。

>
假设通知栏高度为20像素，导航栏高度为40像素,那么我们在内容区的(0，0)位置绘制一个点，最终就要转化为在实际坐标系中的(0，60)位置绘制一个点。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f624vi3eb6j30rs0goab5.jpg)

以上是仅作为一个简单的示例，实际上不论2D还是3D，我们要将图形显示在屏幕上，都离不开Matrix，所以说Matrix是一个在背后辛勤工作的劳模。



### Matrix特点

* 作用范围更广，Matrix在View，图片，动画效果等各个方面均有运用，相比与之前讲解等画布操作应用范围更广。
* 更加灵活，画布操作是对Matrix的封装，Matrix作为更接近底层的东西，必然要比画布操作更加灵活。
* 封装很好，Matrix本身对各个方法就做了很好的封装，让开发者可以很方便的操作Matrix。
* 
* 难以深入理解，很难理解中各个数值的意义，以及操作规律，如果不了解矩阵，也很难理解前乘，后乘。

<p id="wujie" /> 
### 常见误解

**1.认为Matrix最下面的一行的三个参数(MPERSP_0、MPERSP_1、MPERSP_2)没有什么太大的作用，在这里只是为了凑数。**

实际上最后一行参数在3D变换中有着至关重要的作用，这一点会在后面中Camera一文中详细介绍。

**2.最后一个参数MPERSP_2被解释为scale**

的确，更改MPERSP_2的值能够达到类似缩放的效果，但这是因为齐次坐标的缘故，并非这个参数的实际功能。

******

<p id="jiben" /> 
## Matrix基本原理

Matrix 是一个矩阵，最根本的作用就是坐标转换，下面我们就看看几种常见变换的原理:

> 我们所用到的变换均属于仿射变换，仿射变换是 线性变换(缩放，旋转，错切) 和 平移变换(平移) 的复合，由于这些概念对于我们作用并不大，此处不过多介绍，有兴趣可自行了解。

基本变换有4种: 平移(translate)、缩放(scale)、旋转(rotate) 和 错切(skew)。

下面我们看一下四种变换都是由哪些参数控制的。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f60gwrhlnyj30c008zdgy.jpg)
![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f633hvklfnj30c008zdge.jpg)

**从上图可以看到最后三个参数是控制透视的，这三个参数主要在3D效果中运用，通常为(0, 0, 1)，不在本篇讨论范围内，暂不过多叙述，会在之后对文章中详述其作用。**

由于我们以下大部分的计算都是基于矩阵乘法规则，如果你已经把线性代数还给了老师，请参考一下这里:
**[维基百科-矩阵乘法](https://zh.wikipedia.org/wiki/%E7%9F%A9%E9%99%A3%E4%B9%98%E6%B3%95)**

### 1.缩放(Scale)

![](http://latex.codecogs.com/png.latex?$$ x = k_1 x_0 $$)

![](http://latex.codecogs.com/png.latex?$$ y = k_2 y_0 $$)


用矩阵表示: 

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
k_1  &   0   &  0  \\\\
 0   &  k_2  &  0  \\\\
 0   &   0   &  1
\\end{1} 
\\right ] 
\\left [ 
\\begin{matrix} 
x_0 \\\\
y_0 \\\\
1
\\end{1} 
\\right ]
$$)

> 
你可能注意到了，我们坐标多了一个1，这是使用了齐次坐标系的缘故，在数学中我们的点和向量都是这样表示的(x, y)，两者看起来一样，计算机无法区分，为此让计算机也可以区分它们，增加了一个标志位，增加之后看起来是这样: <br/>
>
(x, y, 1) - 点<br/>
(x, y, 0) - 向量<br/>
>
另外，齐次坐标具有等比的性质，(2,3,1)、(4,6,2)...(2N,3N,N)表示的均是(2,3)这一个点。(**将MPERSP_2解释为scale这一误解就源于此**)。

图例：

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f6cnk02zy9j308c0dwwej.jpg)

### 2.错切(Skew)

错切存在两种特殊错切，水平错切(平行X轴)和垂直错切(平行Y轴)。

#### 水平错切

![](http://latex.codecogs.com/png.latex?$$ x = x_0 + ky_0 $$)

![](http://latex.codecogs.com/png.latex?$$ y = y_0 $$)

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix}  
 1   &  k  &  0 \\\\
 0   &  1   &  0 \\\\
 0   &  0   &  1
\\end{1} 
\\right ] 
\\left [ 
\\begin{matrix} 
x_0\\\\
y_0\\\\
1
\\end{1} 
\\right ]
$$)

图例:

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f6cniifb0sj308c0dw3yz.jpg)

#### 垂直错切

![](http://latex.codecogs.com/png.latex?$$ x = x_0 $$)

![](http://latex.codecogs.com/png.latex?$$ y = kx_0 + y_0 $$)

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix}  
 1   &  0  &  0 \\\\
 k   &  1   &  0 \\\\
 0   &  0   &  1
\\end{1} 
\\right ] 
\\left [ 
\\begin{matrix} 
x_0\\\\
y_0\\\\
1
\\end{1} 
\\right ]
$$)

图例:

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f6cnkwyksij308c0dwq3f.jpg)

#### 复合错切

> 水平错切和垂直错切的复合。

![](http://latex.codecogs.com/png.latex?$$ x = x_0 + k_1 y_0 $$)

![](http://latex.codecogs.com/png.latex?$$ y = k_2 x_0 + y_0 $$)

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix}  
 1   &  k_1 &  0 \\\\
 k_2 &  1   &  0 \\\\
 0   &  0   &  1
\\end{1} 
\\right ] 
\\left [ 
\\begin{matrix} 
x_0\\\\
y_0\\\\
1
\\end{1} 
\\right ]
$$)

图例:

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f6cqdu6olfj308c0dwdgi.jpg)

### 3.旋转(Rotate)

假定一个点 A(x<sub>0</sub>, y<sub>0</sub>) ,距离原点距离为 r, 与水平轴夹角为 α 度, 绕原点旋转 θ 度, 旋转后为点 B(x, y) 如下:

![](http://latex.codecogs.com/png.latex? $$ x_0 = r \\cdot cos \\alpha $$)

![](http://latex.codecogs.com/png.latex? $$ y_0 = r \\cdot sin \\alpha $$)

![](http://latex.codecogs.com/png.latex?
$$
x = r \\cdot cos( \\alpha + \\theta) 
= r \\cdot cos \\alpha \\cdot cos \\theta - r \\cdot sin \\alpha \\cdot sin \\theta 
= x_0 \\cdot cos \\theta - y_0 \\cdot sin \\theta
$$)

![](http://latex.codecogs.com/png.latex?
$$
y = r \\cdot sin( \\alpha + \\theta) 
= r \\cdot sin \\alpha \\cdot cos \\theta + r \\cdot cos \\alpha \\cdot sin \\theta 
= y_0 \\cdot cos \\theta + x_0 \\cdot sin \\theta
$$)

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
cos(\\theta) & -sin(\\theta) &  0 \\\\
sin(\\theta) & cos(\\theta)  &  0 \\\\
      0      &       0       &  1
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
x_0\\\\
y_0\\\\
1
\\end{1} 
\\right ]
$$)

图例:

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f6cpp174twj308c0dwt8s.jpg)


### 4.平移(Translate)

> 
此处也是使用齐次坐标的优点体现之一，实际上前面的三个操作使用 2x2 的矩阵也能满足需求，但是使用 2x2 的矩阵，无法将平移操作加入其中，而将坐标扩展为齐次坐标后，将矩阵扩展为 3x3 就可以将算法统一，四种算法均可以使用矩阵乘法完成。

![](http://latex.codecogs.com/png.latex?$$ x = x_0 + \\Delta x $$)

![](http://latex.codecogs.com/png.latex?$$ y = y_0 + \\Delta y $$)

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
x\\\\
y\\\\
1
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
1 & 0 & \\Delta x \\\\
0 & 1 & \\Delta y \\\\
0 & 0 & 1
\\end{1} 
\\right ] 
 . 
\\left [ 
\\begin{matrix} 
x_0\\\\
y_0\\\\
1
\\end{1} 
\\right ]
$$)

图例:

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f6dqiw20xoj308c0dw0su.jpg)


<p id="fuhe" /> 
## Matrix复合原理

其实Matrix的多种复合操作都是使用矩阵乘法实现的，从原理上理解很简单，但是，使用矩阵乘法也有其弱点，后面的操作可能会影响到前面到操作，所以在构造Matrix时顺序很重要。

我们常用的四大变换操作，每一种操作在Matrix均有三类,前乘(pre)，后乘(post)和设置(set)，可以参见文末对[Matrix方法表](#fangfa)，由于矩阵乘法不满足交换律，所以前乘(pre)，后乘(post)和设置(set)的区别还是很大的。

### 前乘(pre)

前乘相当于矩阵的右乘：
![](http://latex.codecogs.com/png.latex?$$ M' =  M \\cdot S $$)

> 这表示一个矩阵与一个特殊矩阵前乘后构造出结果矩阵。

### 后乘(post)

前乘相当于矩阵的左乘：
![](http://latex.codecogs.com/png.latex?$$ M' =  S \\cdot M $$)

> 这表示一个矩阵与一个特殊矩阵后乘后构造出结果矩阵。

### 设置(set)

设置使用的不是矩阵乘法，而是直接覆盖掉原来的数值，所以，**使用设置可能会导致之前的操作失效**。

## 组合

我们使用Matrix最终目的就是让视图显示为我们想要的状态，为此我们可能需要多种操作结合使用。

我发现很多讲解Matrix的文章喜欢用绕某一个点缩放(旋转)的示例来讲解，如下:


>
    那么我们如果想让它基于图片中心缩放，应该该怎么办？要用到组合变换，
      1）先将图片由中心平移到原点，这是应用变换 T
      2）对图应用缩放变换 S 
      3）再将图片平移回到中心，应用变换 -T
      
> 
    对应代码:
      matrix.postScale(0.5f, 0.5f);  
      matrix.preTranslate(-pivotX, -pivotY);  
      matrix.postTranslate(pivotX, pivotY);  
>
    PS: 此段文字引用自其它文章。

首先，**这个思路是没有任何问题的，也是实现绕某一点操作的核心原理**，但这可能会对一部分小白造成误解，认为只能这样实现，然而查看一下Matrix的方法表就能知道四大操作都可以指定中心点，所以，上面的三行代码用一行就能完成:

```java
matrix.postScale(0.5f, 0.5f, pivotX, pivotY);
```

**组合操作构造Matrix时，个人建议尽量全部使用后乘或者全部使用前乘，这样操作顺序容易确定，出现问题也比较容易排查。<br/>当然，由于矩阵乘法不满足交换律，前乘和后乘的结果是不同的，使用时应结合具体情景分析使用。**

### Pre与Post的区别

主要区别其实就是矩阵的乘法顺序不同，pre相当于矩阵的右乘，而post相当于矩阵的左乘。

以下观点存在歧义，故做删除标注:

<s>
在图像处理中，越靠近右边的矩阵越先执行，所以pre操作会先执行，而post操作会后执行。
</s>

在实际操作中，我们每一步操作都会得出准确的计算结果，但是为什么还会用存在先后的说法? 难道真的能够用pre和post影响计算顺序? 实则不然，下面我们用一个例子说明:

> 
```java
Matrix matrix = new Matrix();
matrix.postScale(0.5f, 0.8f);
matrix.preTranslate(1000, 1000);
Log.e(TAG, "MatrixTest:3" + matrix.toShortString());
```
>
在上面的操作中，如果按照正常的思路，先缩放，后平移，缩放操作执行在前，不会影响到后续的平移操作，但是执行结果却发现平移距离变成了(500， 800)。

> 在上面例子中，计算顺序是没有问题的，先计算的缩放，然后计算的平移，而缩放影响到平移则是因为前一步缩放后的结果矩阵右乘了平移矩阵，这是符合矩阵乘法的运算规律的，也就是说缩放操作虽然在前却影响到了平移操作，**相当于先执行了平移操作，然后执行的缩放操作，因此才有pre操作会先执行，而post操作会后执行这一说法**。


### 下面我们用不同对方式来构造一个矩阵:

**假设我们需要先缩放再平移。**

注意: 
* 1.由于矩阵乘法不满足交换律，请保证使用初始矩阵(Initial Matrix)，否则可能导致运算结果不同。
* 2.注意构造顺序，顺序是会影响结果的。
* 3.Initial Matrix是指new出来的新矩阵，或者reset后的矩阵，是一个单位矩阵。


#### 1.仅用pre：

``` java
Matrix m ＝ new Matrix();
m.reset();
m.preTranslate(tx, ty); //使用pre，越靠后越先执行。
m.preScale(sx, sy);
```

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
 & &\\\\
 & Result Matrix &\\\\
 & &
\\end{1} 
\\right ] 
 = 
 \\left [ 
\\begin{matrix} 
 & &\\\\
 & Initial Matrix &\\\\
 & &
\\end{1} 
\\right ] 
\\cdot 
\\left [ 
\\begin{matrix} 
1 & 0 & \\Delta x \\\\
0 & 1 & \\Delta y \\\\
0 & 0 & 1
\\end{1} 
\\right ] 
\\cdot 
\\left [ 
\\begin{matrix} 
sx & 0 & 0\\\\
0 & sy & 0\\\\
0 & 0 & 1
\\end{1} 
\\right ]
$$)

#### 2.仅用post:

``` java
Matrix m ＝ new Matrix();
m.reset();
m.postScale(sx, sy);  //使用post，越靠前越先执行。
m.postTranslate(tx, ty);
```

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
 & &\\\\
 & Result Matrix &\\\\
 & &
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
1 & 0 & \\Delta x \\\\
0 & 1 & \\Delta y \\\\
0 & 0 & 1
\\end{1} 
\\right ] 
\\cdot 
\\left [ 
\\begin{matrix} 
sx & 0 & 0\\\\
0 & sy & 0\\\\
0 & 0 & 1
\\end{1} 
\\right ]
\\cdot 
 \\left [ 
\\begin{matrix} 
 & &\\\\
 & Initial Matrix &\\\\
 & &
\\end{1} 
\\right ] 
$$)

#### 3.混合:

``` java
Matrix m ＝ new Matrix();
m.reset();
m.preScale(sx, sy);  
m.postTranslate(tx, ty);
```

或:

``` java
Matrix m ＝ new Matrix();
m.reset();
m.postTranslate(tx, ty);
m.preScale(sx, sy);  
```

> 由于此处只有两步操作，且指定了先后，所以代码上交换并不会影响结果。

用矩阵表示:

![](http://latex.codecogs.com/png.latex?
$$
\\left [ 
\\begin{matrix} 
 & &\\\\
 & Result Matrix &\\\\
 & &
\\end{1} 
\\right ] 
 = 
\\left [ 
\\begin{matrix} 
1 & 0 & \\Delta x \\\\
0 & 1 & \\Delta y \\\\
0 & 0 & 1
\\end{1} 
\\right ] 
\\cdot 
 \\left [ 
\\begin{matrix} 
 & &\\\\
 & Initial Matrix &\\\\
 & &
\\end{1} 
\\right ] 
\\cdot 
\\left [ 
\\begin{matrix} 
sx & 0 & 0\\\\
0 & sy & 0\\\\
0 & 0 & 1
\\end{1} 
\\right ]
$$)


**注意: 由于矩阵乘法不满足交换律，请保证初始矩阵为空，如果初始矩阵不为空，则导致运算结果不同。**


<p id="fangfa" /> 
## Matrix方法表

这个方法表，暂时放到这里让大家看看，方法的使用讲解放在下一篇文章中。

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

<p id="zongjie" />
## 总结

对于Matrix重在理解，理解了其中的原理之后用起来将会更加得心应手。

**学完了本篇之后，推荐配合鸿洋大大的视频课程 [
打造个性的图片预览与多点触控](http://www.imooc.com/learn/239) 食用，定然能够让你对Matrix对理解更上一层楼。**

<p id="about" />
## About Me

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300 height=100 /> </a>

<p id="ziliao" />
## 参考资料

[Matrix](https://developer.android.com/reference/android/graphics/Matrix.html)<br/>
[Android中图像变换Matrix的原理、代码验证和应用](http://biandroid.iteye.com/blog/1399462)<br/>
[Android中关于矩阵（Matrix）前乘后乘的一些认识](http://blog.csdn.net/linmiansheng/article/details/18820599)<br/>
[维基百科-仿射变换](https://zh.wikipedia.org/wiki/%E4%BB%BF%E5%B0%84%E5%8F%98%E6%8D%A2)<br/>
[维基百科-齐次坐标](https://zh.wikipedia.org/wiki/%E9%BD%90%E6%AC%A1%E5%9D%90%E6%A0%87)<br/>
[维基百科-线性映射](https://zh.wikipedia.org/wiki/%E7%BA%BF%E6%80%A7%E6%98%A0%E5%B0%84)<br/>
[齐次坐标系入门级思考](https://oncemore2020.github.io/blog/homogeneous/)<br/>
[仿射变换与齐次坐标](https://guangchun.wordpress.com/2011/10/12/affineandhomogeneous/)<br/>
[]()<br/>
