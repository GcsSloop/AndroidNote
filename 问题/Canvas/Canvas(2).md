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

### 1.画布操作
#### 为什么要有画布操作
画布操作可以帮助我们用更加容易理解的方式制作图形，如果你之前看过绘制太极和Canvas(1)最后的实例，你就会发现其中就有对translate的运用。

translate 是干什么用的呢？

上面表格中写的是"位移"，但"位移"的词义很是模糊，到底位移的是什么？那我换种说法，你应该就明白了，translate是坐标系的移动，为图形绘制选择一个合适的坐标系，看下图：

<i>PS:请不要在意坐标系数值和图像的倾斜，制作软件出了一点问题，如果你有比较好的制作数学图形的软件可以推荐一下。</i>

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art2/translate.png)

不同的坐标系对于描述一个图形的难度实际上是不同的，就拿上图来说，假设蓝色矩形为屏幕，左侧状态为默认的坐标系，右侧为将坐标原点移动到屏幕中心的坐标系，要在屏幕中心绘制一个圆形，在左侧的坐标系中，你需要计算出圆心的位置和半径，而右侧只需知道半径即可。

合理的使用画布操作可以帮助你用更容易理解的方式更加迅速的创作你想要的效果，这也是画布操作存在的原因。

下面对几种画布操作详细讲解。

#### 位移(translate)
 <b>请注意，位移是基于当前位置移动，而不是基于屏幕左上角的物理位置移动</b>，如下：
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

我们首先将坐标系移动一段距离绘制一个圆形，之后再移动一段距离绘制一个圆形，两次移动是叠加的。

#### 缩放(scale)

#### 旋转(rotate)

#### 倾斜(skew)

### 快照(save)和回滚(restore)

