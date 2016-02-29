# Canvas基础四
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

本次会了解到path(路径)这个神器，有了这个神器，就能创造出更多**炫(zhuang)酷(B)**的东东了。

******

# 一.Path常用方法表
> 为了兼容性(_偷懒_) 本表格中去除了在API21(即安卓版本5.0)以上才添加的方法。忍不住吐槽一下，为啥看起来有些顺手就能写的重载方法要等到API21才添加上啊。宝宝此刻内心也是崩溃的。

作用 | 相关方法 | 备注
--- | --- | ---
移动起点 | moveTo | 移动下一次操作的起点位置
设置终点 | setLastPoint | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同
连接直线 | lineTo | 添加上一个点到当前点之间的直线到Path
闭合路径 | close  | 连接第一个点连接到最后一个点，形成一个闭合区域
重置路径 | reset, rewind | 清除Path中的内容(**reset相当于重置到new Path阶段，rewind会保留Path的数据结构**)
添加内容 | addRect, addRoundRect,  addOval, addCircle, 	addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)
是否为空 | isEmpty | 判断Path是否为空
替换路径 | set | 用新的路径替换到当前路径所有内容
偏移路径 | offset | 对当前路径之前的操作进行偏移(不会影响之后的操作)
计算边界 | computeBounds | 计算Path的边界
贝塞尔曲线 | quadTo cubicTo | 分别为二次和三次贝塞尔曲线的方法
rXxx方法 | rMoveTo rLineTo rQuadTo rCubicTo | **不带r的方法是基于原点的坐标系(偏移量)，rXxx方法是基于当前点坐标系(偏移量)**
填充模式 | setFillType, getFillType, isInverseFillType, toggleInverseFillType| 设置,获取,判断和切换填充模式
提示方法 | incReserve | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)**
布尔操作(API19) | op | 对两个Path进行布尔运算(即取交集、并集等操作)
矩阵操作 | transform | 矩阵变换


# 二.Path详解

## Path作用
本次特地开了一篇详细讲解Path，为什么要单独摘出来呢，这是因为Path在2D绘图中是一个很重要的东西。

在前面我们讲解的所有绘制都是简单图形(如 矩形 圆 圆弧等)，而对于那些复杂一点的图形则没法去绘制(如绘制一个心形 正多边形 五角星等)，而**使用Path不仅能够绘制简单图形，也可以绘制这些比较复杂的图形。另外，根据路径绘制文本和剪裁画布都会用到Path。**

关于Path的作用先简单地说这么多，具体的我们接下来慢慢研究。

## Path含义

**官方介绍：**

_The Path class encapsulates compound (multiple contour) geometric paths consisting of straight line segments, quadratic curves, and cubic curves. It can be drawn with canvas.drawPath(path, paint), either filled or stroked (based on the paint's Style), or it can be used for clipping or to draw text on a path._

> 嗯，没错依旧是拿来装逼的，如果你看不懂的话，不用担心，其实并没有什么卵用。

**通俗解释(sloop个人版)：**

**Path是封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。你能用Canvas中的drawPath来把这条路径画出来(同样支持Paint的不同绘制模式)，也可以用于剪裁画布和根据路径绘制文字。我们有时会用Path来描述一个图像的轮廓，所以也会称为轮廓线(轮廓线仅是Path的一种使用方法，两者并不等价)**


另外路径有开放和封闭的区别。
  
图像 | 名称 | 备注
 --- | --- | ---
 ![](http://ww4.sinaimg.cn/thumbnail/005Xtdi2jw1f0zx9g9gggj30f00aiwek.jpg) | 封闭路径 | 首尾相接形成了一个封闭区域
 ![](http://ww1.sinaimg.cn/thumbnail/005Xtdi2jw1f0zxg8ilpxj30f00aimx8.jpg) | 开放路径 | 没有首位相接形成封闭区域
  
> 这个是我随便画的，仅为展示一下区别，请无视我灵魂画师一般的绘图水准。

**与Path相关的还有一些比较神奇的概念，不过暂且不说，等接下来需要用到的时候再详细说明。**

## Path使用方法详解

前面扯了一大堆概念性的东西。接下来就开始实战了，请诸位看官准备好瓜子、花生、爆米花，坐下来慢慢观看。

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f19mncfcirj305i02j744.jpg)

### 第1组: moveTo、 setLastPoint、 lineTo 和 close
  
由于Path的有些知识点无法单独来讲，所以本次采取了一次讲一组方法。
  
按照惯例，先创建画笔：
  
``` java
        Paint mPaint = new Paint();             // 创建画笔
        mPaint.setColor(Color.BLACK);           // 画笔颜色 - 黑色
        mPaint.setStyle(Paint.Style.STROKE);    // 填充模式 - 描边
        mPaint.setStrokeWidth(10);              // 边框宽度 - 10
```
  
#### lineTo：
  
  首先讲解的的LineTo，为啥先讲解这个呢？
  
  是因为moveTo、 setLastPoint、 close都无法直接看到效果，借助有具现化效果的lineTo才能让这些方法现出原形。
  
方法预览：

```
public void lineTo (float x, float y)
```

lineTo很简单，只有一个方法，作用也很容易理解，line嘛，顾名思义就是一条线。

俗话(数学书上)说，两点确定一条直线，但是看参数明显只给了一个点的坐标吧(这不按常理出牌啊)。

再仔细一看，这个lineTo除了line外还有一个to呢，to翻译过来就是“至”，到某个地方的意思，**lineTo难道是指从某个点到参数坐标点之间连一条线？**

没错，你猜对了，但是这某个点又是哪里呢？

前面我们提到过Path可以用来描述一个图像的轮廓，图像的轮廓通常都是用一条线构成的，所以这里的某个点就是上次操作结束的点，如果没有进行过操作则默认点为坐标原点。

那么我们就来试一下：

``` java
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心(宽高数据在onSizeChanged中获取)

        Path path = new Path();                     // 创建Path

        path.lineTo(200, 200);                      // lineTo
        path.lineTo(200,0);

        canvas.drawPath(path, mPaint);              // 绘制Path
```


<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f1ap1tu0w9j30u01hcjse.jpg" width = "270" height = "480"/>  

在示例中我们调用了两次lineTo，**第一次由于之前没有过操作，所以默认点就是坐标原点O，结果就是坐标原点O到A(200,200)之间连直线(用蓝色圈1标注)。**

**第二次lineTo的时候，由于上次的结束位置是A(200,200),所以就是A(200,200)到B(200,0)之间的连线(用蓝色圈2标注)。**

#### moveTo 和 setLastPoint：

这两个方法虽然在作用上有相似之处，但实际上却是完全不同的两个东东，具体参照下表：

方法名 | 简介  | 是否影响之前的操作 | 是否影响之后操作
--- | --- | --- | ---
moveTo | 移动下一次操作的起点位置 | 否 | 是
setLastPoint | 设置之前操作的最后一个点位置 | 是 | 是

废话不多说，直接上代码：
``` java
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心

        Path path = new Path();                     // 创建Path

        path.lineTo(200, 200);                      // lineTo

        path.moveTo(200,100);                       // moveTo

        path.lineTo(200,0);                         // lineTo

        canvas.drawPath(path, mPaint);              // 绘制Path
```
<img src="http://ww3.sinaimg.cn/large/005Xtdi2gw1f1aqjptdtjj30u01hct9t.jpg" width = "270" height = "480"/>  

这个和上面演示lineTo的方法类似，只不过在两个lineTo之间添加了一个moveTo。

**moveTo只改变下次操作的起点，在执行完第一次LineTo的时候，本来的默认点位置是A(200,200),但是moveTo将其改变成为了C(200,100),所以在第二次调用lineTo的时候就是连接C(200,100) 到 B(200,0) 之间的直线(用蓝色圈2标注)。**

下面是setLastPoint的示例：

``` java 
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心

        Path path = new Path();                     // 创建Path

        path.lineTo(200, 200);                      // lineTo

        path.setLastPoint(200,100);                 // setLastPoint

        path.lineTo(200,0);                         // lineTo

        canvas.drawPath(path, mPaint);              // 绘制Path
```
<img src="http://ww1.sinaimg.cn/large/005Xtdi2gw1f1ari1l9g8j30u01hcab5.jpg" width = "270" height = "480"/> 

**setLastPoint是重置上一次操作的最后一个点，在执行完第一次的lineTo的时候，最后一个点是A(200,20),而setLastPoint更改最后一个点为C(200,100),所以在实际执行的时候，第一次的lineTo就不是从原点O到A(200,200)的连线了，而变成了从原点O到C(200,100)之间的连线了。**

**在执行完第一次lineTo和setLastPoint后，最后一个点的位置是C(200,100),所以在第二次调用lineTo的时候就是C(200,100) 到 B(200,0) 之间的连线(用蓝色圈2标注)。**

#### close

close方法用于连接当前最后一个点和最初的一个点(如果两个点不重合的话)，最终形成一个封闭的图形。

``` java
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心

        Path path = new Path();                     // 创建Path

        path.lineTo(200, 200);                      // lineTo

        path.lineTo(200,0);                         // lineTo

        path.close();                               // close

        canvas.drawPath(path, mPaint);              // 绘制Path
```
<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f1axmfeojzj30u01hcwfi.jpg" width = "270" height = "480"/> 

很明显，两个lineTo分别代表第1和第2条线，而close在此处的作用就算连接了B(200,0)点和圆的O之间的第3条线，使之形成一个封闭的图形。

**注意：close的作用是封闭路径，与当前最后一个点和第一个点并不等价。如果连接了最后一个点和第一个点仍然无法形成封闭图形，则close什么 也不做。**

### 第2组: addXxx与arcTo

这次内容主要是在Path中添加基本图形，重点区别addArc与arcTo的区别。

首先进行方法预览：
``` java 
// 第一类(基本形状)
    // 圆形
    public void addCircle (float x, float y, float radius, Path.Direction dir)
    // 椭圆
    public void addOval (RectF oval, Path.Direction dir)
    // 矩形
    public void addRect (float left, float top, float right, float bottom, Path.Direction dir)
    public void addRect (RectF rect, Path.Direction dir)
    // 圆角矩形
    public void addRoundRect (RectF rect, float[] radii, Path.Direction dir)
    public void addRoundRect (RectF rect, float rx, float ry, Path.Direction dir)

// 第二类(Path)
    // path
    public void addPath (Path src)
    public void addPath (Path src, float dx, float dy)
    public void addPath (Path src, Matrix matrix)

// 第三类(addArc与arcTo)
    // addArc
    public void addArc (RectF oval, float startAngle, float sweepAngle)
    // arcTo
    public void arcTo (RectF oval, float startAngle, float sweepAngle)
    public void arcTo (RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)

```

#### 第一类(基本形状)
这一类就是在path中添加一个基本形状，基本形状部分和前面所讲的绘制基本形状并无太大差别，详情参考[Canvas(1)颜色与基本形状](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Canvas(1).md), 本次只将其中不同的部分摘出来详细讲解一下。

**仔细观察一下第一类是方法，无一例外，在最后都有一个_Path.Direction_，这是一个什么神奇的东东？**

Direction的意思是 方向，趋势。 点进去看一下会发现Direction是一个枚举(Enum)类型，里面只有两个枚举常量，如下：

类型 | 解释 | 翻译
--- | --- | ---
CW | clockwise | 顺时针
CCW | counter-clockwise | 逆时针

> **瞬间懵逼，我只是想添加一个基本的形状啊，搞什么顺时针和逆时针, (╯‵□′)╯︵┻━┻**

**稍安勿躁，┬─┬ ノ( ' - 'ノ) {摆好摆好） 既然存在肯定是有用的，先偷偷剧透一下这个顺时针和逆时针的作用。**

序号 | 作用 
--- | --- 
 1 | 在添加图形时确定闭合顺序
 2 | 在选择填充模式时，对自相交图像最终渲染结果可能有影响

这个先剧透这么多，至于对闭合顺序有啥影响，自相交图形的渲染等问题等请慢慢看下去

咱们先研究确定闭合顺序的问题，添加一个矩形试试看：

``` java
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心

        Path path = new Path();

        path.addRect(-200,-200,200,200, Path.Direction.CW);

        canvas.drawPath(path,mPaint);
```
<img src="http://ww1.sinaimg.cn/large/005Xtdi2gw1f1cmvjtuxcj30u01hcwgm.jpg" width = "270" height = "480"/> 

**将上面代码的CW改为CCW再运行一次。接下来就是见证奇迹的时刻，两次运行结果一模一样，有木有很神奇！**

> **(╯°Д°)╯︵ ┻━┻(再TM掀一次) 
坑人也不带这样的啊，一毛一样要它干嘛。**

**其实啊，这个东东是自带隐身技能的，想要让它现出原形，就要用到咱们刚刚学到的setLastPoint(重置当前最后一个点的位置)。**

```
        canvas.translate(mWidth / 2, mHeight / 2);  // 移动坐标系到屏幕中心
        canvas.scale(1,-1);                         // <-- 注意 scale特殊运用：翻转y坐标轴

        Path path = new Path();

        path.addRect(-200,-200,200,200, Path.Direction.CW);

        path.setLastPoint(-300,300);                // <-- 移动最后一个点的位置

        canvas.drawPath(path,mPaint);
```

<img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1ea3qc3j6j30u01hcabi.jpg" width = "270" height = "480"/> 

**请注意：为了演示方便，本次将坐标系调整到了和我们常见的数学坐标系相同，请留意代码的变动，否则效果刚好是反着的，会更加难以理解。**

> 如果你对坐标系有疑问，请参考之前文章 [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md) <br/>
>如果你对翻转坐标轴有疑问，请参考之前文章 [Canvas(2)画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Canvas(2).md)

可以明显看到，图形发生了奇怪的变化。为何会如此呢？

我们先分析一下，绘制一个矩形(仅绘制边线)，实际上只需要进行四次lineTo操作就行了，也就是说，只需要知道4个点的坐标，然后使用moveTo到第一个点，之后依次lineTo就行了(从上面的测试可以看出，在实际绘制中也确实是这么干的)。

可是为什么要这么做呢？确定一个矩形最少需要两个点(对角线的两个点)，根据这两个点的坐标直接算出四条边然后画出来不就行了，干嘛还要先计算出四个点坐标，之后再连直线呢？

这个就要涉及一些path的存储问题了，前面在path中的定义中说过，Path是封装了由直线和曲线(二次，三次贝塞尔曲线)构成的几何路径。其中曲线部分用的是贝塞尔曲线，稍后再讲。 然而除了曲线部分就只剩下直线了，对于直线的存储最简单的就是记录坐标点，然后直接连接各个点就行了。而虽然记录矩形只需要两个点，但是如果只用两个点来记录一个矩形的话，就要额外增加一个标志位来记录这是一个矩形，显然对于存储和解析都是很不划算的事情，将矩形转换为直线，为的就是存储记录方便。






## 贝塞尔曲线

在展示的图像中的Path由**直线**和**曲线**构成，我们知道对于**直线还是比较好描述的，两点确定一条直线，而不规则的曲线我们一般会用到贝塞尔曲线**，关于贝塞尔曲线这个的概念你们还是参考一下这里[**维基百科-贝塞尔曲线**](https://zh.wikipedia.org/wiki/%E8%B2%9D%E8%8C%B2%E6%9B%B2%E7%B7%9A)，或者自己Google一下，不过嘛你们可以在下面预览一下贝塞尔曲线：

贝塞尔曲线 | 结构 | 演示动画
 --- | --- | ---
 一阶曲线<br/>(线性曲线) | ![](https://upload.wikimedia.org/wikipedia/commons/0/00/B%C3%A9zier_1_big.gif) | ![](https://upload.wikimedia.org/wikipedia/commons/0/00/B%C3%A9zier_1_big.gif)
 二阶曲线 | ![](https://upload.wikimedia.org/wikipedia/commons/6/6b/B%C3%A9zier_2_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/3/3d/B%C3%A9zier_2_big.gif)
三阶曲线 |  ![](https://upload.wikimedia.org/wikipedia/commons/8/89/B%C3%A9zier_3_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/d/db/B%C3%A9zier_3_big.gif)
四阶曲线 | ![](https://upload.wikimedia.org/wikipedia/commons/b/bf/B%C3%A9zier_4_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/a/a4/B%C3%A9zier_4_big.gif)


看了上面的一到四阶贝塞尔曲线的效果图，有木有不明觉厉，感觉好高端，好厉害的样子，除了一阶的还知道怎么回事，二阶以上就懵逼了。

其实贝塞尔曲线都是靠一些不明觉厉的公式计算出来的，它的通用公式如下：

![](https://upload.wikimedia.org/math/8/f/4/8f4c915ef475b93fc0f8374f378e436f.png)

额，看完之后更加懵逼了，这么复杂的公式臣妾真心不会算啊。

不过不要紧，这肯定不是手算的，在本文后面内容中会教大家一些取巧的方法，这里先讲Path，讲到曲线的部分再详细的说明。





## 贝塞尔曲线详解


# 总结
# 参考资料
