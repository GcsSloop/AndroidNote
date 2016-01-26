# Canvas基础二
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

上一次我们了解了一些Canvas的基础知识，这次依旧是Canvas相关的一些内容。

相比于上一次绘制简单的图形，这次的内容会稍微有趣一点点，相信大家看完之后会对Canvas更加了解,能够制作一些<b>更(geng)加(neng)炫(zhuang)酷(B)</b>的东西出来。

## 一.Canvas常用操作
为了方便，我把上次的表格搬过来了。

操作类型 | 相关API | 备注
--- | --- | ---
绘制颜色 | drawColor, drawRGB, drawARGB | 使用单一颜色填充整个画布<br/> 相关 : [安卓自定义View基础-颜色](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E9%A2%9C%E8%89%B2/%E9%A2%9C%E8%89%B2.md)
绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片 | drawBitmap, drawPicture | 绘制位图和图片
绘制文本 | drawText,    drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径 | drawPath | 绘制路径，绘制贝塞尔曲线时也需要用到该函数
顶点操作 | drawVertices, drawBitmapMesh | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁 | clipPath,    clipRect | 设置画布的显示区域
画布快照 | save, restore | 两者一般成对使用，save是保存当前状态，restore是回滚到上一次保存的状态
画布变换 | translate, scale, rotate, skew | 依次为 位移、缩放、 旋转、倾斜<br/> 相关： [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md)    [角度与弧度](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6.md)
Matrix(矩阵) | getMatrix, setMatrix, concat | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

******
## 二.Canvas基本操作
上次呢我们了解了绘制颜色和绘制基本形状，这次会了解画布变换，绘制图片，绘制文字和绘制路径。

*****
### 1.画布操作
#### 为什么要有画布操作？
画布操作可以帮助我们用更加容易理解的方式制作图形，如果你之前看过绘制太极和Canvas(1)最后的实例，你就会发现其中就有对translate的运用。

translate 是干什么用的呢？

上面表格中写的是"位移"，但"位移"的词义很是模糊，到底位移的是什么？那换种说法，translate是坐标系的移动，为图形绘制选择一个合适的坐标系，看下图：

<i>PS:请不要在意坐标系数值和图像的倾斜，制作软件出了一点问题，如果你有比较好的制作数学图形的软件可以推荐一下。</i>

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/translate.png)

不同的坐标系对于描述一个图形的难度实际上是不同的，就拿上图来说，假设蓝色矩形为屏幕，左侧状态为默认的坐标系，右侧为将坐标原点移动到屏幕中心的坐标系，要在屏幕中心绘制一个圆形，在左侧的坐标系中，你需要计算出圆心的位置和半径，而右侧只需知道半径即可。

<b>合理的使用画布操作可以帮助你用更容易理解的方式创作你想要的效果，这也是画布操作存在的原因。</b>

下面对几种画布操作详细讲解。

*****
#### ⑴位移(translate)
 <b>请注意，位移是基于当前位置移动，而不是每次基于屏幕左上角的(0,0)点移动</b>，如下：
```
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

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/translate.jpg" width = "270" height = "480" alt="title" align=center />  

我们首先将坐标系移动一段距离绘制一个圆形，之后再移动一段距离绘制一个圆形，<b>两次移动是叠加的</b>。

*****
#### ⑵缩放(scale)
缩放提供了两个方法，如下：
```
 public void scale (float sx, float sy)

 public final void scale (float sx, float sy, float px, float py)
```
这两个方法中前两个参数是相同的分别为x轴和y轴的缩放比例(1.0代表不变，大于1代表放大，小于1代表缩小)，而第二种方法比前一种多了两个参数。

多出来的两个参数是干什么的？ 当然是控制缩放中心位置的。

如果在缩放时稍微注意一下就会发现<b>缩放的中心默认为坐标原点</b>，如下：
```
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

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale1.jpg" width = "270" height = "480" alt="title" align=center />  

接下来我们使用第二种方法让缩放中心位置稍微改变一下，如下：
```
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

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale2.jpg" width = "270" height = "480" alt="title" align=center />  

<b>PS:和位移(translate)一样，缩放也是可以叠加的。</b>
```
   canvas.scale(0.5f,0.5f);
   canvas.scale(0.5f,0.5f);
```
调用两次缩放到0.5则实际缩放为0.5x0.5=0.25

下面我们利用叠加效果制作一个有趣的图形。
```
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(-400,-400,400,400);   // 矩形区域

        for (int i=0; i<=20; i++)
        {
            canvas.scale(0.9f,0.9f);
            canvas.drawRect(rect,mPaint);
        }
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/scale3.jpg" width = "270" height = "480" alt="title" align=center />  

*****
#### ⑶旋转(rotate)
和缩放一样，旋转同样提供了两种方法。
```
  public void rotate (float degrees)
  
  public final void rotate (float degrees, float px, float py)
```
和缩放一样，第二种方法多出来的两个参数依旧是控制旋转中心点的。

默认的旋转中心依旧是坐标原点：
```
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180);                     // 旋转180度 <-- 默认旋转中心为原点

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/rotate1.jpg" width = "270" height = "480" alt="title" align=center />  

改变旋转中心位置：
```
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,-400,400,0);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.rotate(180,200,0);               // 旋转180度 <-- 旋转中心向右偏移200个单位

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/rotate2.jpg" width = "270" height = "480" alt="title" align=center />  


*****
#### ⑷倾斜(skew)
skew这里翻译为倾斜，有的地方也叫错切。

倾斜只提供了一种方法：
```
  public void skew (float sx, float sy)
```
<b>参数含义：<br/>
float sx:将画布在x方向上倾斜相应的角度，sx倾斜角度的tan值，<br/>
float sy:将画布在y轴方向上倾斜相应的角度，sy为倾斜角度的tan值.</b>

示例：
```
        // 将坐标系原点移动到画布正中心
        canvas.translate(mWidth / 2, mHeight / 2);

        RectF rect = new RectF(0,0,200,200);   // 矩形区域

        mPaint.setColor(Color.BLACK);           // 绘制黑色矩形
        canvas.drawRect(rect,mPaint);

        canvas.skew(1,0);                       // 在x轴倾斜45度 <-- tan45 = 1

        mPaint.setColor(Color.BLUE);            // 绘制蓝色矩形
        canvas.drawRect(rect,mPaint);
```
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/skew.jpg" width = "270" height = "480" alt="title" align=center />  

*****
#### ⑸快照(save)和回滚(restore)

