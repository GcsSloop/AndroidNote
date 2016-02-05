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

> **注意：在使用Picture之前请关闭硬件加速，以免引起不必要的问题，如何关闭请参考这里： [Android的硬件加速及可能导致的问题](https://github.com/GcsSloop/AndroidNote/issues/7)**

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

> **注意：在使用Picture之前请关闭硬件加速，以免引起不必要的问题，如何关闭请参考这里： [Android的硬件加速及可能导致的问题](https://github.com/GcsSloop/AndroidNote/issues/7)**

### (2)drawBitmap

 > 其实一开始知道要讲Bitmap我是拒绝的，为什么呢？因为Bitmap就是很多问题的根源啊有木有，Bitmap可能导致内存不足，内存泄露，ListView中的复用混乱等诸多问题。想完美的掌控Bitmap还真不是一件容易的事情。限于篇幅**本文对于Bitmap不会过多的展开，只讲解一些常用的功能**，关于Bitmap详细内容，以后开专题讲解QAQ。

 既然要绘制Bitmap，就要先获取一个Bitmap，那么如何获取呢？
 
 **获取Bitmap方式:**
 
 序号 | 获取方式 | 备注
 --- | --- | ---
  1 | 通过Bitmap创建            | 复制一个已有的Bitmap(_新Bitmap状态和原有的一致_) 或者 创建一个空白的Bitmap(_内容可改变_)
  2 | 通过BitmapDrawable获取    | 从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
  3 | 通过BitmapFactory获取     | 从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap
  
**通常来说，我们绘制Bitmap都是读取已有的图片转换为Bitmap绘制到Canvas上。**<br/>
很明显，第1种方式不能满足我们的要求，暂时排除。<br/>
第2种方式虽然也可满足我们的要求，但是我不推荐使用这种方式，至于为什么在后续详细讲解Drawable的时候会说明,暂时排除。<br/>
第3种方法我们会比较详细的说明一下如何从各个位置获取图片。<br/>

#### 通过BitmapFactory从不同位置获取Bitmap:

**资源文件(drawable/mipmap/raw):**
```
        Bitmap bitmap = BitmapFactory.decodeResource(mContext.getResources(),R.raw.bitmap);
```
**资源文件(assets):**
```
        Bitmap bitmap=null;
        try {
            InputStream is = mContext.getAssets().open("bitmap.png");
            bitmap = BitmapFactory.decodeStream(is);
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
```

**内存卡文件:**
```
    Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/bitmap.png");
```

**网络文件:**
```
        // 此处省略了获取网络输入流的代码
        Bitmap bitmap = BitmapFactory.decodeStream(is);
        is.close();
```

既然已经获得到了Bitmap，那么就开始本文的重点了，将Bitmap绘制到画布上。
#### 绘制Bitmap：
依照惯例先预览一下drawBitmap的常用方法：
```
    // 第一种
    public void drawBitmap (Bitmap bitmap, Matrix matrix, Paint paint)
    
    // 第二种
    public void drawBitmap (Bitmap bitmap, float left, float top, Paint paint)
    
    // 第三种
    public void drawBitmap (Bitmap bitmap, Rect src, Rect dst, Paint paint)
    public void drawBitmap (Bitmap bitmap, Rect src, RectF dst, Paint paint)
```

第一种方法中后两个参数(matrix, paint)是在绘制的时候对图片进行一些改变，如果只是需要将图片内容绘制出来只需要如下操作就可以了：

PS:图片左上角位置默认为坐标原点。
```
    canvas.drawBitmap(bitmap,new Matrix(),new Paint());
```
> 关于Matrix和Paint暂时略过吧，一展开又是啰啰嗦嗦一大段，反正挖坑已经是常态了，大家应该也习惯了(PAP).

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawBitmap1.jpg" width = "270" height = "480"/> 

第二种方法就是在绘制时指定了图片左上角距离坐标原点的距离：

> **注意：此处指定的是与坐标原点的距离，并非是与屏幕顶部和左侧的距离。**
```
    canvas.drawBitmap(bitmap,200,500,new Paint());
```
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawBitmap2.jpg" width = "270" height = "480"/> 

第三种方法比较有意思，上面多了两个矩形区域(src,dst),这两个矩形选区是干什么用的？

名称 | 作用
--- | ---
Rect src | 指定绘制图片的区域
Rect dst <br/>或RectF dst | 指定图片在屏幕上显示(绘制)的区域

示例：
```
        // 将画布坐标系移动到画布中央
        canvas.translate(mWidth/2,mHeight/2);

        // 指定图片绘制区域(左上角的四分之一)
        Rect src = new Rect(0,0,bitmap.getWidth()/2,bitmap.getHeight()/2);

        // 指定图片在屏幕上显示的区域
        Rect dst = new Rect(0,0,200,400);

        // 绘制图片
        canvas.drawBitmap(bitmap,src,dst,null);
```
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawBitmap3.jpg" width = "270" height = "480"/> 

**详解：**

上面是以绘制该图为例，用src指定了图片绘制部分的区域，即下图中红色方框标注的区域。

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawBitmap4.jpg)

然后用dst指定了绘制在屏幕上的绘制，即下图中蓝色方框标注的区域，图片宽高会根据指定的区域自动进行缩放。
<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art3/drawBitmap5.jpg" width = "270" height = "480"/> 

从上面可知，第三种方法可以绘制图片的一部分到画布上，这有什么用呢？

如果你看过某些游戏的资源文件，你可能会看到如下的图片：


用一张图片包含了大量的素材，在绘制的时候每次只截取一部分进行绘制，这样可以大大的减少素材数量，而且素材管理起来也很方便。

然而这和我们有什么关系呢？我们又不做游戏开发。

确实，虽然我们不做游戏开发，但是在某些时候我们需要制作一些炫酷的效果，这些效果因为太复杂了用代码很难实现或者渲染效率不高。这时候很多人就会想起帧动画，将动画分解成一张一张的图片然后使用帧动画制作出来，这种实现方式的确比较简单，但是一个动画效果的图片有十几到几十张，一个应用里面来几个这样炫酷的动画效果就会导致资源文件出现一大堆，想找其中的某一张资源图片简直就是灾难啊有木有。但是把同一个动画效果的所有资源图片整理到一张图片上，会大大的减少资源文件数量，方便管理，妈妈再也不怕我找不到资源文件了。

下面是利用drawBitmap方法制作的一个简单示例。

资源文件如下：

最终效果如下：

源码如下：




## 2.绘制文字

## 3.绘制路径

