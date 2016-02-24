# Canvas基础四
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

本次会了解到path(路径)这个神器，有了这个神器，就能创造出更多**炫(zhuang)酷(B)**的东东了。

******

# 一.Path常用方法表
> 为了兼容性(_偷懒_) 本表格中去除了在API21(即安卓版本5.0)以上才添加的方法。忍不住吐槽一下，为啥看起来有些顺手就能写的重载方法要等到API21才添加上啊。宝宝此刻内心也是崩溃的。

作用 | 相关方法 | 备注
--- | --- | ---
移动起点 | moveTo | 移动下一次绘制的起点坐标
设置终点 | setLastPoint | 设置之前绘制的最后一个点位置，如果在绘制之前调用，效果和moveTo相同
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
  
**lineTo：**
  
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

        canvas.drawPath(path, mPaint);              // 绘制Path
```

  
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
