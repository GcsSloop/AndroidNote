# Path之贝塞尔曲线

### 作者微博: [@攻城师sloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView)

在上一篇文章[Path之基本图形](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_BasicGraphics.md)中我们了解了Path的基本使用方法，本次了解Path中非常重要的内容-贝塞尔曲线。


******

# 一.Path常用方法表
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

# 二.Path详解

上一次除了一些常用函数之外，讲解的基本上都是直线，本次需要了解其中的曲线部分,说到曲线，就不得不提大名鼎鼎的贝塞尔曲线。它的发明者是下面这个人(法国数学家PierreBézier)。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f1ky5bw28pg305k07h3yo.gif)

**在Google中搜索贝塞尔曲线，我们看到的基本就是下面这些不明觉厉的图形：**

贝塞尔曲线 | 结构 | 演示动画
 --- | --- | ---
 一阶曲线<br/>(线性曲线) | ![](https://upload.wikimedia.org/wikipedia/commons/0/00/B%C3%A9zier_1_big.gif) | ![](https://upload.wikimedia.org/wikipedia/commons/0/00/B%C3%A9zier_1_big.gif)
 二阶曲线 | ![](https://upload.wikimedia.org/wikipedia/commons/6/6b/B%C3%A9zier_2_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/3/3d/B%C3%A9zier_2_big.gif)
三阶曲线 |  ![](https://upload.wikimedia.org/wikipedia/commons/8/89/B%C3%A9zier_3_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/d/db/B%C3%A9zier_3_big.gif)
四阶曲线 | ![](https://upload.wikimedia.org/wikipedia/commons/b/bf/B%C3%A9zier_4_big.svg) | ![](https://upload.wikimedia.org/wikipedia/commons/a/a4/B%C3%A9zier_4_big.gif)

**或者类似于这样的不明觉厉的公式：**

![](https://upload.wikimedia.org/math/8/f/4/8f4c915ef475b93fc0f8374f378e436f.png)

**基本上，大部分人看到这里已经懵逼了，话说这根本就是数学家干的活吧，让我一个写程序(_差点挂在高数上下不来_)的弄这个？**

> **不用担心，本次内容中不会拿这些不明觉厉的东西去忽悠大家，会尽量用通俗易懂接地气的方式描述贝塞尔曲线，下面正式开始。**

## 贝塞尔曲线能干什么？

贝塞尔曲线的运用是十分广泛的，可以说**贝塞尔曲线奠定了计算机绘图的基础(_因为它可以将任何复杂的图形用精确的数学语言进行描述_)**，在你不经意间就已经使用过它了。

你会使用Photoshop的话，你可能会注意到里面有一个**钢笔工具**。这是一个令人十分费解的东西，我在一开始尝试使用的时候总会制造出一些和我预期不同的曲线，各种歪歪扭扭。这个钢笔工具就是贝塞尔曲线的一种运用。

你说你不会PS？ 没关系，你如果看过前面的文章或者用过2D绘图，肯定绘制过圆，圆弧，圆角矩形等这些东西。这里面的圆弧部分全部都是贝塞尔曲线的运用。

实际上在前面我们所绘制的**圆是用四段贝塞尔曲线拼接出来的**，有木有觉得很不可思议？

咱们可以做一个实验，**教大家如何用圆分分钟做出一个吃豆人(PackMan)**:

上代码：
``` java

```


#### 贝塞尔曲线作用十分广泛，简单举个的栗子: QQ小红点拖拽效果，一些炫酷的下拉刷新控件，阅读软件的翻书效果，一些平滑的折线图的制作等等...除此之外，在动画方面贝塞尔也是功不可没，很多炫酷的动画效果都有贝塞尔是身影出没。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f2wqwi6b4aj305406owef.jpg)

## 论如何避免接触公式学好贝塞尔曲线




# 三.总结

# 四.参考资料
