# Canvas基础四
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

本次会了解到path(路径)这个神器，有了这个神器，就能创造出更多炫酷的东东了。

******

# 一.Path常用方法表

作用 | 相关方法 | 备注
--- | --- | ---
移动起点 | moveTo | 移动下一次绘制的起点坐标
连接直线 | lineTo | 添加上一个点到当前点之间的直线到Path
贝塞尔曲线 | quadTo cubicTo | 分别为二次和三次贝塞尔曲线的方法
rXxx方法 | rMoveTo rLineTo rQuadTo rCubicTo | 和上面方法类似，**上面方法是基于原点的坐标系，rXxx方法是基于当前点偏移**
添加图形 | addXxx | 添加基本形状(矩形 圆形 圆弧等)到Path
布尔操作 | op | 对两个Path进行布尔运算(即取集、并集等操作)
填充模式 | setFillType getFillType | 设置和获取填充模式
闭合路径 | close  | 连接第一个点连接到最后一个点，形成一个闭合区域
重置Path | reset rewind | 清除Path中的内容
矩阵操作 | transform | 矩阵变换


# 二.Path详解

## Path作用
本次特地开了一篇详细讲解Path，为什么要单独摘出来呢，这是因为Path在2D绘图中是一个很重要的东西。

在前面我们讲解的所有绘制都是简单图形(如 矩形 圆 圆弧等)，而对于那些复杂一点的图形则没法去绘制(如绘制一个心形 正多边形 五角星等)，而**使用Path不仅能够绘制简单图形，也可以绘制这些比较复杂的图形。另外，根据路径绘制文本和剪裁画布都会用到Path。**

关于Path的作用先简单地说这么多，具体的我们接下来慢慢研究。

## Path含义(sloop个人版, 非官方)

  简单介绍完path的作用，按照套路当然要讲一下Path是什么了，Path在中文中一般翻译做**路径**，但是路径到底是什么鬼，在不同的地方定义是完全是不同的，在网络中指从起点到终点经过的所有路由，在文件系统中指文件从根目录到当前文件的文件结构，在生活中指经过的道路。在绘图中指用绘图工具创建的任意形状曲线。
  
**那这里的Path到底是什么呢？**
  
  Path是一个大神器，逼格也比较高，干脆就叫做**逼格拉斯之剑**吧，(这TM是什么鬼(￣▽￣)")<br/>
  
  不开玩笑了，咱们正式的定义一下什么是Path：<br/>
    **Path是一条曲线，我们经常用这条曲线勾勒图形图像的轮廓，所以我们也称之为“轮廓线”。Path有开闭之分，封闭的路径只所有曲线最后首位相接连接成一个封闭的区域，而开放的路径就是没有闭合形成封闭区域。**
  
封闭路径 | 开放路径
 --- | ---
 ![](http://ww4.sinaimg.cn/thumbnail/005Xtdi2jw1f0zx9g9gggj30f00aiwek.jpg) | ![](http://ww1.sinaimg.cn/thumbnail/005Xtdi2jw1f0zxg8ilpxj30f00aimx8.jpg)
  
> 这个是我随便画的，仅为展示一下区别，请无视我灵魂画师一般的绘图水准。

## Path使用方法详解


  
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
