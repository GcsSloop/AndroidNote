# Canvas之画布操作

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView/README.md)

上一篇[Canvas之绘制基本形状](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B02%5DCanvas_BasicGraphics.md)中我们了解了如何使用Canvas绘制基本图形，本次了解一些基本的画布操作。

本来想把画布操作放到后面部分的，但是发现很多图形绘制都离不开画布操作，于是先讲解一下画布的基本操作方法。

## 一.Canvas的常用操作速查表

| 操作类型       | 相关API                                    | 备注                                       |
| ---------- | ---------------------------------------- | ---------------------------------------- |
| 绘制颜色       | drawColor, drawRGB, drawARGB             | 使用单一颜色填充整个画布                             |
| 绘制基本形状     | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧                  |
| 绘制图片       | drawBitmap, drawPicture                  | 绘制位图和图片                                  |
| 绘制文本       | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字          |
| 绘制路径       | drawPath                                 | 绘制路径，绘制贝塞尔曲线时也需要用到该函数                    |
| 顶点操作       | drawVertices, drawBitmapMesh             | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用 |
| 画布剪裁       | clipPath,    clipRect                    | 设置画布的显示区域                                |
| 画布快照       | save, restore, saveLayerXxx, restoreToCount, getSaveCount | 依次为 保存当前状态、 回滚到上一次保存的状态、 保存图层状态、 回滚到指定状态、 获取保存次数 |
| 画布变换       | translate, scale, rotate, skew           | 依次为 位移、缩放、 旋转、错切                         |
| Matrix(矩阵) | getMatrix, setMatrix, concat             | 实际上画布的位移，缩放等操作的都是图像矩阵Matrix， 只不过Matrix比较难以理解和使用，故封装了一些常用的方法。 |

******
## 二.Canvas基本操作

### 1.画布操作
#### 为什么要有画布操作？

画布操作可以帮助我们用更加容易理解的方式制作图形。

例如： 从坐标原点为起点，绘制一个长度为20dp，与水平线夹角为30度的线段怎么做？

按照我们通常的想法(*被常年训练出来的数学思维*)，就是先使用三角函数计算出线段结束点的坐标，然后调用drawLine即可。

然而这是否是被固有思维禁锢了？

假设我们先绘制一个长度为20dp的水平线，然后将这条水平线旋转30度，则最终看起来效果是相同的，而且不用进行三角函数计算，这样是否更加简单了一点呢？

**合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果，这也是画布操作存在的原因。**

**PS: 所有的画布操作都只影响后续的绘制，对之前已经绘制过的内容没有影响。**

*****
#### ⑴位移(translate)

 translate是坐标系的移动，可以为图形绘制选择一个合适的坐标系。
 **请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动**，如下：
``` java
        // 省略了创建画笔的代码
        
        // 在坐标原点绘制一个黑色圆形
        mPaint.setColor(Color.BLACK);
        canvas.translate(200,200);
        canvas.drawCircle(0,0,100,mPaint);

        // 在坐标原点绘制一个蓝色圆形
        mPaint.setColor(Color.BLUE);
        canvas.translate(200,200);
        canvas.drawCircle(0,0,100,mPaint);
```

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2f1ph46qaj30u01hcgm3.jpg" width="300"/>  

我们首先将坐标系移动一段距离绘制一个圆形，之后再移动一段距离绘制一个圆形，<b>两次移动是可叠加的</b>。

*****
#### ⑵缩放(scale)
缩放提供了两个方法，如下：
``` java
 public void scale (float sx, float sy)

 public final void scale (float sx, float sy, float px, float py)
```
这两个方法中前两个参数是相同的分别为x轴和y轴的缩放比例。而第二种方法比前一种多了两个参数，用来控制缩放中心位置的。

缩放比例(sx,sy)取值范围详解：

| 取值范围(n) | 说明                                           |
| ----------- | ---------------------------------------------- |
| (-∞, -1)    | 先根据缩放中心放大n倍，再根据中心轴进行翻转    |
| -1          | 根据缩放中心轴进行翻转                         |
| (-1, 0)     | 先根据缩放中心缩小到n，再根据中心轴进行翻转    |
| 0           | 不会显示，若sx为0，则宽度为0，不会显示，sy同理 |
| (0, 1)      | 根据缩放中心缩小到n                            |
| 1           | 没有变化                                       |
| (1, +∞)     | 根据缩放中心放大n倍                            |

如果在缩放时稍微注意一下就会发现<b>缩放的中心默认为坐标原点,而缩放中心轴就是坐标轴</b>，如下：

``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f);                // 画布缩放

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
(为了更加直观，我添加了一个坐标系，可以比较明显的看出，缩放中心就是坐标原点)

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2f1vphdjjj30u01hct9r.jpg" width="300" />  

接下来我们使用第二种方法让缩放中心位置稍微改变一下，如下：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.scale(0.5f,0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
(图中用箭头指示的就是缩放中心。)

<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f2f1w7kv8dj30u01hct9s.jpg" width="300" />  

前面两个示例缩放的数值都是正数，按照表格中的说明，**当缩放比例为负数的时候会根据缩放中心轴进行翻转**，下面我们就来实验一下：

``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);


        canvas.scale(-0.5f,-0.5f);          // 画布缩放

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
<img src="http://ww1.sinaimg.cn/large/005Xtdi2jw1f2f1x76o6qj30u01hc0tu.jpg" width="300" />  

> 为了效果明显，这次我不仅添加了坐标系而且对矩形中几个重要的点进行了标注，具有相同字母标注的点是一一对应的。

由于本次未对缩放中心进行偏移，所有默认的缩放中心就是坐标原点，中心轴就是x轴和y轴。

本次缩放可以看做是先根据缩放中心(坐标原点)缩放到原来的0.5倍，然后分别按照x轴和y轴进行翻转。

``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);


        canvas.scale(-0.5f,-0.5f,200,0);          // 画布缩放  <-- 缩放中心向右偏移了200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2f1xth4p6j30u01hc0u4.jpg" width="300" />  

> 添加了这么多的辅助内容，希望大家能够看懂。

本次对缩放中心点y轴坐标进行了偏移，故中心轴也向右偏移了。


<b>PS:和位移(translate)一样，缩放也是可以叠加的。</b>
``` java
   canvas.scale(0.5f,0.5f);
   canvas.scale(0.5f,0.1f);
```
调用两次缩放则 x轴实际缩放为0.5x0.5=0.25 y轴实际缩放为0.5x0.1=0.05

下面我们利用这一特性制作一个有趣的图形。

> 注意设置画笔模式为描边(STROKE)

``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(-400,-400,400,400);   // 矩形区域

        for (int i=0; i<=20; i++)
        {
            canvas.scale(0.9f,0.9f);
            canvas.drawRect(rect,mPaint);
        }
```

<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f2f1yfn22xj30u01hcta9.jpg" width="300" />  

*****
#### ⑶旋转(rotate)
旋转提供了两种方法：
``` java
  public void rotate (float degrees)
  
  public final void rotate (float degrees, float px, float py)
```
和缩放一样，第二种方法多出来的两个参数依旧是控制旋转中心点的。

默认的旋转中心依旧是坐标原点：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="http://ww2.sinaimg.cn/large/005Xtdi2jw1f2f1yws38nj30u01hcmy8.jpg" width="300" />  

改变旋转中心位置：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180,200,0);               // 旋转180度 <-- 旋转中心向右偏移200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="http://ww2.sinaimg.cn/large/005Xtdi2jw1f2f1zcmwb2j30u01hcmy9.jpg" width="300" />

<b>好吧，旋转也是可叠加的</b>
``` java
     canvas.rotate(180);
     canvas.rotate(20);
```
调用两次旋转，则实际的旋转角度为180+20=200度。

为了演示这一个效果，我做了一个不明觉厉的东西：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        canvas.drawCircle(0,0,400,mPaint);          // 绘制两个圆形
        canvas.drawCircle(0,0,380,mPaint);

        for (int i=0; i<=360; i+=10){               // 绘制圆形之间的连接线
            canvas.drawLine(0,380,0,400,mPaint);
            canvas.rotate(10);
        }
```
<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2f1zsnj00j30u01hc75a.jpg" width="300" />  

*****
#### ⑷错切(skew)

skew这里翻译为错切，错切是特殊类型的线性变换。

错切只提供了一种方法：
``` java
  public void skew (float sx, float sy)
```

<b>参数含义：<br/>
float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，<br/>
float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.</b>

变换后:
```
X = x + sx * y
Y = sy * x + y
```

示例：
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 水平错切 <- 45度

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f2f20h7i23j30u01hcdgq.jpg" width="300" />  

<b>如你所想，错切也是可叠加的，不过请注意，调用次序不同绘制结果也会不同</b>
``` java
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 水平错切
        canvas.skew(0,1);                       // 垂直错切

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2f20w0rffj30u01hcgm8.jpg" width="300" />

*****
#### ⑸快照(save)和回滚(restore)

<b>
Q: 为什么存在快照与回滚<br/>
A：画布的操作是不可逆的，而且很多画布操作会影响后续的步骤，例如第一个例子，两个圆形都是在坐标原点绘制的，而因为坐标系的移动绘制出来的实际位置不同。所以会对画布的一些状态进行保存和回滚。
</b>

<b>与之相关的API:</b>

| 相关API          | 简介                             |
| -------------- | ------------------------------ |
| save           | 把当前的画布的状态进行保存，然后放入特定的栈中        |
| saveLayerXxx   | 新建一个图层，并放入特定的栈中                |
| restore        | 把栈中最顶层的画布状态取出来，并按照这个状态恢复当前的画布  |
| restoreToCount | 弹出指定位置及其以上所有的状态，并按照指定位置的状态进行恢复 |
| getSaveCount   | 获取栈中内容的数量(即保存次数)               |

下面对其中的一些概念和方法进行分析：

##### 状态栈：
其实这个栈我也不知道叫什么名字，暂时叫做状态栈吧，它看起来像下面这样：

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f2gfqmu2sgj308c0dwmxw.jpg)

这个栈可以存储画布状态和图层状态。

<b>Q：什么是画布和图层？<br/>
A：实际上我们看到的画布是由多个图层构成的，如下图(图片来自网络)：<br/>

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f2gfrc6fdkj308c0dwglr.jpg)

实际上我们之前讲解的绘制操作和画布操作都是在默认图层上进行的。<br/>
在通常情况下，使用默认图层就可满足需求，但是如果需要绘制比较复杂的内容，如地图(地图可以有多个地图层叠加而成，比如：政区层，道路层，兴趣点层)等，则分图层绘制比较好一些。<br/>
你可以把这些图层看做是一层一层的玻璃板，你在每层的玻璃板上绘制内容，然后把这些玻璃板叠在一起看就是最终效果。
</b>

##### SaveFlags

| 数据类型 | 名称                         | 简介                                       |
| ---- | -------------------------- | ---------------------------------------- |
| int  | ALL_SAVE_FLAG              | 默认，保存全部状态                                |
| int  | CLIP_SAVE_FLAG             | 保存剪辑区                                    |
| int  | CLIP_TO_LAYER_SAVE_FLAG    | 剪裁区作为图层保存                                |
| int  | FULL_COLOR_LAYER_SAVE_FLAG | 保存图层的全部色彩通道                              |
| int  | HAS_ALPHA_LAYER_SAVE_FLAG  | 保存图层的alpha(不透明度)通道                       |
| int  | MATRIX_SAVE_FLAG           | 保存Matrix信息(translate, rotate, scale, skew) |

##### save
save 有两种方法：
``` java
  // 保存全部状态
  public int save ()
  
  // 根据saveFlags参数保存一部分状态
  public int save (int saveFlags)
```
可以看到第二种方法比第一种多了一个saveFlags参数，使用这个参数可以只保存一部分状态，更加灵活，这个saveFlags参数具体可参考上面表格中的内容。

每调用一次save方法，都会在栈顶添加一条状态信息，以上面状态栈图片为例，再调用一次save则会在第5次上面载添加一条状态。

#### saveLayerXxx
saveLayerXxx有比较多的方法：
``` java
// 无图层alpha(不透明度)通道
public int saveLayer (RectF bounds, Paint paint)
public int saveLayer (RectF bounds, Paint paint, int saveFlags)
public int saveLayer (float left, float top, float right, float bottom, Paint paint)
public int saveLayer (float left, float top, float right, float bottom, Paint paint, int saveFlags)

// 有图层alpha(不透明度)通道
public int saveLayerAlpha (RectF bounds, int alpha)
public int saveLayerAlpha (RectF bounds, int alpha, int saveFlags)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha)
public int saveLayerAlpha (float left, float top, float right, float bottom, int alpha, int saveFlags)
```
<b>注意：saveLayerXxx方法会让你花费更多的时间去渲染图像(图层多了相互之间叠加会导致计算量成倍增长)，使用前请谨慎，如果可能，尽量避免使用。</b>

使用saveLayerXxx方法，也会将图层状态也放入状态栈中，同样使用restore方法进行恢复。

这个暂时不过多讲述，如果以后用到详细讲解。(因为这里面东西也有不少啊QAQ)

##### restore
状态回滚，就是从栈顶取出一个状态然后根据内容进行恢复。

同样以上面状态栈图片为例，调用一次restore方法则将状态栈中第5次取出，根据里面保存的状态进行状态恢复。

##### restoreToCount
弹出指定位置以及以上所有状态，并根据指定位置状态进行恢复。

以上面状态栈图片为例，如果调用restoreToCount(2) 则会弹出 2 3 4 5 的状态，并根据第2次保存的状态进行恢复。

##### getSaveCount
获取保存的次数，即状态栈中保存状态的数量，以上面状态栈图片为例，使用该函数的返回值为5。

不过请注意，该函数的最小返回值为1，即使弹出了所有的状态，返回值依旧为1，代表默认状态。

##### 常用格式
虽然关于状态的保存和回滚啰嗦了不少，不过大多数情况下只需要记住下面的步骤就可以了：
``` java
   save();      //保存状态
   ...          //具体操作
   restore();   //回滚到之前的状态
```
这种方式也是最简单和最容易理解的使用方法。



******
## 三.总结

如本文一开始所说，合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果。

(,,• ₃ •,,)

<b>PS: 由于本人英文水平有限，某些地方可能存在误解或词语翻译不准确，如果你对此有疑问可以提交Issues进行反馈。</b>

## About Me
### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300"/> </a>

******
## 四.参考资料

[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)<br/>
[canvas变换与操作](http://blog.csdn.net/harvic880925/article/details/39080931)<br/>
[Canvas之translate、scale、rotate、skew方法讲解](http://blog.csdn.net/tianjian4592/article/details/45234419)<br/>
[Canvas的save(),saveLayer()和restore()浅谈](http://www.cnblogs.com/liangstudyhome/p/4143498.html)<br/>
[Graphics->Layers](http://www.programgo.com/article/72302404062/;jsessionid=8E62016408BFFB21D46F9C878A49D8EE)<br/>
[]()<br/>
[]()<br/>

