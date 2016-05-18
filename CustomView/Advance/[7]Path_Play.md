# Path之玩出花样

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView)

经历过前两篇 [Path之基本操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_Basic.md) 和 [Path之贝塞尔曲线](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B6%5DPath_Bezier.md) 的讲解，本篇正式进入Path的收尾篇，通过使用前面的内容和本文新学的内容，教大家如何将玩转Path，玩出花样。

******

## 一.Path常用方法表
> 为了兼容性(_偷懒_) 本表格中去除了在API21(即安卓版本5.0)以上才添加的方法。忍不住吐槽一下，为啥看起来有些顺手就能写的重载方法要等到API21才添加上啊。宝宝此刻内心也是崩溃的。

作用 | 相关方法 | 备注
--- | --- | ---
移动起点   | moveTo | 移动下一次操作的起点位置
设置终点   | setLastPoint | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同
连接直线   | lineTo | 添加上一个点到当前点之间的直线到Path
闭合路径   | close  | 连接第一个点连接到最后一个点，形成一个闭合区域
添加内容   | addRect, addRoundRect,  addOval, addCircle, 	addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)
是否为空   | isEmpty | 判断Path是否为空
是否为矩形 | isRect  | 判断path是否是一个矩形
替换路径   | set | 用新的路径替换到当前路径所有内容
偏移路径   | offset | 对当前路径之前的操作进行偏移(不会影响之后的操作)
贝塞尔曲线 | quadTo, cubicTo | 分别为二次和三次贝塞尔曲线的方法
rXxx方法   | rMoveTo, rLineTo, rQuadTo, rCubicTo | **不带r的方法是基于原点的坐标系(偏移量)，rXxx方法是基于当前点坐标系(偏移量)**
填充模式   | setFillType, getFillType, isInverseFillType, toggleInverseFillType| 设置,获取,判断和切换填充模式
提示方法   | incReserve | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)**
布尔操作(API19) | op | 对两个Path进行布尔运算(即取交集、并集等操作)
计算边界   | computeBounds | 计算Path的边界
重置路径   | reset, rewind | 清除Path中的内容(**reset相当于重置到new Path阶段，rewind会保留Path的数据结构**)
矩阵操作   | transform | 矩阵变换


## 二、Path方法详解

### rXxx方法

此类方法可以看到和前面的一些方法看起来很像，只是在前面多了一个r，那么这个rXxx和前面的一些方法有什么区别呢？

> **rXxx方法的坐标使用的是相对位置(基于当前点的位移)，而之前方法的坐标是绝对位置(基于当前坐标系的坐标)。**

**举个例子:**

``` java
    Path path = new Path();

    path.moveTo(100,100);
    path.lineTo(100,200);

    canvas.drawPath(path,mDeafultPaint);
```
<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f403p6ftn9j30u01hc74m.jpg" width=300 />

在这个例子中，先移动点到坐标(100，100)处，之后再连接 _点(100，100)_ 到 _(100，200)_ 之间点直线,非常简单，画出来就是一条竖直的线，那接下来看下一个例子：

``` java
    Path path = new Path();

    path.moveTo(100,100);
    path.rLineTo(100,200);

    canvas.drawPath(path,mDeafultPaint);
```
<img src ="http://ww1.sinaimg.cn/large/005Xtdi2jw1f403zs9qeej30u01hcdg7.jpg" width=300 />

这个例子中，将 lineTo 换成了 rLineTo 可以看到在屏幕上原本是竖直的线变成了倾斜的线。这是因为最终我们连接的是 _(100,100)_ 和 _(200, 300)_ 之间的线段。

在使用rLineTo之前，当前点的位置在 (100,100) ， 使用了 rLineTo(100,200) 之后，下一个点的位置是在当前点的基础上加上偏移量得到的，即 (100+100, 100+200) 这个位置，故最终结果如上所示。

**PS: 此处仅以 rLineTo 为例，只要理解 “绝对坐标” 和 “相对坐标” 的区别，其他方法类比即可。**

### 填充模式

跳票了两次终于讲到自相交图形问题了，在讲解之前我们先了解一下什么是自相交图形。

**自相交图形定义：**

> 多边形在平面内除了顶点之外还有其他公共点。

### 布尔操作(API19)

### 计算边界

### 重置路径






