# Canvas基础三
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

Canvas的内容确实有点多，这次接着了解其他相关内容。

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
# 二.Canvas基本操作详解

上次讲解了画布相关的一些操作，这次讲解绘制图片 文字 路径等一些基本操作。

## 1.绘制图片

绘制有两种方法，drawPicture 和 drawBitmap,接下来我们一一了解。

### (1)drawPicture

关于drawPicture一开始还是挺让人费解的，不过嘛，我们接下来慢慢研究一下它的用途。

既然是drawPicture就要了解一下什么是**Picture**。 顾名思义，Picture的意思是图片。

不过嘛，我觉得这么用图片这个名词解释Picture是不合适的，为何这么说？请看其官方文档对[Picture](http://developer.android.com/reference/android/graphics/Picture.html)的解释：

<i>
A Picture records drawing calls (via the canvas returned by beginRecording) and can then play them back into Canvas (via draw(Canvas) or drawPicture(Picture)).For most content (e.g. text, lines, rectangles), drawing a sequence from a picture can be faster than the equivalent API calls, since the picture performs its playback without incurring any method-call overhead.
</i>

好吧，我知道大部分人对这段鸟语是看不懂的，至于为什么要放在这里，仅仅是为了显得更加专业(偷笑)。

<b>下面我就对这段不明觉厉的鸟语用通俗的话翻译一下:</b>

某一天小萌想在朋友面前显摆一下，于是在单杠上来了一个后空翻,动作姿势请参照下图：

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/Picture1.jpg)

朋友都说 恩，很不错。 想再看一遍 (〃ω〃)。ヽ(〃∀〃)ﾉ。⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄

于是小萌又来了一遍，如是几次之后，小萌累的吐血三升。

于是小萌机智的想，我何不用手机录下来呢，直接保存成为<b>后空翻.avi</b> 下次想显摆的时候直接拿出手机，点一下播放就行了，省时省力。

小萌被自己的机智深深的折服了，然后Picture就诞生啦。(╯‵□′)╯︵┻━┻掀桌，坑爹呢，这剧情跳跃也忒大了吧。

<b>
好吧，言归正传，这次我们了解的Picture和上文中的录像功能是类似的，只不过我们Picture录的是Canvas中绘制的内容。<br/>
我们把Canvas绘制点，线，矩形等诸多操作用Picture录制下来，下次需要的时候拿来就能用，使用Picture相比于再次调用绘图API，开销是比较小的，也就是说对于重复的操作可以更加省时省力。
</b>

<b>PS：你可以把Picture看作是一个录制Canvas操作的录像机。</b>

了解了Picture的概念之后，我们再了解一下Picture的相关方法。

相关方法 | 简介
--- | ---
public int getWidth () | 获取宽度
public int getHeight () | 获取高度
public Canvas beginRecording (int width, int height) | 开始录制 (返回一个Canvas，在Canvas中所有的绘制都会存储在Picture中)
public void endRecording () | 结束录制
public void draw (Canvas canvas) | 将Picture中内容绘制到Canvas中
public static Picture createFromStream (InputStream stream) | (已废弃)通过输入流创建一个Picture
public void writeToStream (OutputStream stream) | (已废弃)将Picture中内容写出到输出流中

上面表格中基本上已经列出了Picture的所有方法，其中getWidth和getHeight没什么好说的，最后两个已经废弃也自然就不用关注了，排除了这些方法之后，只剩三个方法了，接下来我们就比较详细的了解一下：

**很明显，beginRecording 和 endRecording 是成对使用的，一个开始录制，一个是结束录制，两者之间的操作将会存储在Picture中。**

#### 使用示例：

**准备工作:**
``` java 
    // 1.创建Picture
    private Picture mPicture = new Picture();
    
---------------------------------------------------------------
    
    // 2.录制内容方法
    private void recording() {
        // 开始录制 (接收返回值Canvas)
        Canvas canvas = mPicture.beginRecording(500, 500);
        // 创建一个画笔
        Paint paint = new Paint();
        paint.setColor(Color.BLUE);
        paint.setStyle(Paint.Style.FILL);

        // 在Canvas中具体操作
        // 位移
        canvas.translate(250,250);
        // 绘制一个圆
        canvas.drawCircle(0,0,100,paint);

        mPicture.endRecording();
    }
    
---------------------------------------------------------------

    // 3.在使用前调用(我在构造函数中调用了)
      public Canvas3(Context context, AttributeSet attrs) {
        super(context, attrs);
        
        recording();    // 调用录制
    }

```

**具体使用:**

Picture虽然方法就那么几个，但是具体使用起来还是分很多情况的，总体上可以分为以下几种：

序号 | 简介
--- | ---
1 | 使用Picture提供的draw方法绘制。
2 | 使用Canvas提供的drawPicture方法绘制。
3 | 将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。


以上几种方法主要区别：

主要区别 | 分类 | 简介
--- | --- | ---
是否对Canvas有影响 | 1有影响<br/>2,3不影响 | 此处指绘制完成后是否会影响Canvas的状态(Matrix clip等)
可操作性强弱 | 1可操作性较弱<br/>2,3可操作性较强 | 此处的可操作性可以简单理解为对绘制结果可控程度。

几种方法简介和主要区别我知道的就这么多了，接下来对于各种使用方法一一详细介绍：

**1.使用Picture提供的draw方法绘制:**
```
        // 将Picture中的内容绘制在Canvas上
        mPicture.draw(canvas);  
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawPicture1.jpg" width = "270" height = "480"/>  

**PS：这种方法在比较低版本的系统上绘制后可能会影响Canvas状态，所以这种方法一般不会使用。**

**2.使用Canvas提供的drawPicture方法绘制**

drawPicture有三种方法：
```
public void drawPicture (Picture picture)

public void drawPicture (Picture picture, Rect dst)

public void drawPicture (Picture picture, RectF dst)
```
和使用Picture的draw方法不同，Canvas的drawPicture不会影响Canvas状态。

**简单示例:**
``` java
    canvas.drawPicture(mPicture,new RectF(0,0,mPicture.getWidth(),200));
```
 
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawPicture2.jpg" width = "270" height = "480"/>  
 
 **PS:对照上一张图片，可以比较明显的看出，绘制的内容根据选区进行了缩放。 **

**3.将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。**
```
        // 包装成为Drawable
        PictureDrawable drawable = new PictureDrawable(mPicture);
        // 设置绘制区域 -- 注意此处所绘制的实际内容不会缩放，而是相当于剪裁
        drawable.setBounds(0,0,250,mPicture.getHeight());
        // 绘制
        drawable.draw(canvas);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawPicture3.jpg" width = "270" height = "480"/>  

**PS:此处setBounds相当于剪裁显示区域，并非根据该区域进行缩放。**

### (2)drawBitmap
如果你了解过矢量图和位图，你就会发现，其实上面讲的Picture和矢量图非常类似，而Bitmap就是位图，两者区别如下：

类型 | 简介 
--- | ---
 **矢量图:** | 也叫做向量图，由坐标和运算得出，缩放不失真。
 **位图:** | 也叫做点阵图，删格图象，像素图，最小单位由象素构成，缩放会失真。
 
 

