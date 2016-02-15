# Canvas基础四
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

本次会了解到path(路径)这个神器，有了这个神器，就能创造出更多炫酷的东东了。

# 一.Canvas常用操作表

操作类型 | 相关API | 备注
--- | --- | ---
绘制颜色 | drawColor, drawRGB, drawARGB | 使用单一颜色填充整个画布<br/> 相关 : [安卓自定义View基础-颜色](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E9%A2%9C%E8%89%B2/%E9%A2%9C%E8%89%B2.md)
绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片 | drawBitmap, drawPicture | 绘制位图和图片
绘制文本 | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径 | drawPath | 绘制路径，绘制贝塞尔曲线时也需要用到该函数
顶点操作 | drawVertices, drawBitmapMesh | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁 | clipPath,    clipRect | 设置画布的显示区域
画布快照 | save, restore, saveLayerXxx, restoreToCount, getSaveCount | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 会滚到指定状态、 获取保存次数
画布变换 | translate, scale, rotate, skew | 依次为 位移、缩放、 旋转、倾斜<br/> 相关： [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md)    [角度与弧度](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6.md)
Matrix(矩阵) | getMatrix, setMatrix, concat | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

******

# 二.Path详解

## Path说明
本次特地开了一篇详细讲解Path，为什么要单独摘出来呢，这是因为Path在2D绘图中是一个很重要的东西。

在前面我们讲解的所有绘制都是简单图形(如 矩形 圆 圆弧等)，而对于那些复杂一点的图形则没法去绘制(如绘制一个心形 正多边形 五角星等)，而使用Path则可以帮助我们绘制这些复杂的图形。

关于Path的作用先简单地说这么多，具体的我们接下来慢慢研究。

## Path定义(sloop个人版, 非官方)

  简单介绍完path的作用，按照套路当然要讲一下Path是什么了，Path在中文中一般翻译做**路径**，但是路径到底是什么鬼，在不同的地方定义是完全是不同的，在网络中指从起点到终点经过的所有路由，在文件系统中指文件从根目录到当前文件的文件结构，在生活中指经过的道路。在绘图中指用绘图工具创建的任意形状曲线。
  
**那这里的Path到底是什么呢？**
  
  Path是一个大神器，逼格也比较高，干脆就叫做**逼格拉斯之剑**吧，(这TM是什么鬼(￣▽￣)")<br/>
  
  不开玩笑了，咱们正式的定义一下什么是Path：<br/>
    **Path是一条曲线，我们一般用这条曲线勾勒图形图像的轮廓，所以我们一般称之为“轮廓线”。有开闭之分，封闭的轮廓线只所有曲线最后首位相接连接成一个封闭的区域，而开放的轮廓线就是没有闭合形成封闭区域的轮廓线。**
  
封闭 | 开放
 --- | ---
 ![](http://ww4.sinaimg.cn/thumbnail/005Xtdi2jw1f0zx9g9gggj30f00aiwek.jpg) | ![](http://ww1.sinaimg.cn/thumbnail/005Xtdi2jw1f0zxg8ilpxj30f00aimx8.jpg)
  
> 这个是我随便画的，仅为展示一下区别，请无视我灵魂画师一般的绘图水准。
  
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

## Path常用方法表

