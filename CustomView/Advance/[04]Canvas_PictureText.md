# Canvas之图片文字

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView/README.md)

在上一篇文章[Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B03%5DCanvas_Convert.md)中我们了解了画布的一些基本操作方法，本次了解一些绘制图片文字相关的内容。如果你对前几篇文章讲述的内容熟练掌握的话，那么恭喜你，本篇结束之后，大部分的自定义View已经难不倒你了，当然了，这并不是终点，接下来还会有更加炫酷的技能。

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
# 二.Canvas基本操作详解

## 1.绘制图片

绘制有两种方法，drawPicture(矢量图) 和 drawBitmap(位图),接下来我们一一了解。

### (1)drawPicture

**使用Picture前请关闭硬件加速，以免引起不必要的问题！<br/>使用Picture前请关闭硬件加速，以免引起不必要的问题！<br/>使用Picture前请关闭硬件加速，以免引起不必要的问题！**

**在AndroidMenifest文件中application节点下添上 android:hardwareAccelerated="false"以关闭整个应用的硬件加速。 <br/>更多请参考这里：[Android的硬件加速及可能导致的问题](https://github.com/GcsSloop/AndroidNote/issues/7)**

关于drawPicture一开始还是挺让人费解的，不过嘛，我们接下来慢慢研究一下它的用途。

既然是drawPicture就要了解一下什么是Picture。 顾名思义，Picture的意思是图片。

不过嘛，我觉得这么用图片这个名词解释Picture是不合适的，为何这么说？请看其官方文档对[Picture](http://developer.android.com/reference/android/graphics/Picture.html)的解释：

<i>
A Picture records drawing calls (via the canvas returned by beginRecording) and can then play them back into Canvas (via draw(Canvas) or drawPicture(Picture)).For most content (e.g. text, lines, rectangles), drawing a sequence from a picture can be faster than the equivalent API calls, since the picture performs its playback without incurring any method-call overhead.
</i>

好吧，我知道很多人对这段鸟语是看不懂的，至于为什么要放在这里，仅仅是为了显得更加专业(偷笑)。

**下面我就对这段不明觉厉的鸟语用通俗的话翻译一下:**

某一天小萌想在朋友面前显摆一下，于是在单杠上来了一个后空翻,动作姿势请参照下图：

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2kua0sxg0j30bo0b4aba.jpg" width=300 />

朋友都说 恩，很不错。 想再看一遍 (〃ω〃)。ヽ(〃∀〃)ﾉ。⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄

于是小萌又来了一遍，如是几次之后，小萌累的吐血三升。

于是小萌机智的想，我何不能用手机将我是飒爽英姿录下来呢，直接保存成为**后空翻.avi** 下次想显摆的时候直接拿出手机，点一下播放就行了，省时省力。

小萌被自己的机智深深的折服了，然后Picture就诞生啦。(╯‵□′)╯︵┻━┻掀桌，坑爹呢，这剧情跳跃也忒大了吧。


**好吧，言归正传，这次我们了解的Picture和上文中的录像功能是类似的，只不过我们Picture录的是Canvas中绘制的内容。**

我们把Canvas绘制点，线，矩形等诸多操作用Picture录制下来，下次需要的时候拿来就能用，使用Picture相比于再次调用绘图API，开销是比较小的，也就是说对于重复的操作可以更加省时省力。

**PS：你可以把Picture看作是一个录制Canvas操作的录像机。**

了解了Picture的概念之后，我们再了解一下Picture的相关方法。

| 相关方法                                     | 简介                                       |
| ---------------------------------------- | ---------------------------------------- |
| public int getWidth ()                   | 获取宽度                                     |
| public int getHeight ()                  | 获取高度                                     |
| public Canvas beginRecording (int width, int height) | 开始录制 (返回一个Canvas，在Canvas中所有的绘制都会存储在Picture中) |
| public void endRecording ()              | 结束录制                                     |
| public void draw (Canvas canvas)         | 将Picture中内容绘制到Canvas中                    |
| public static Picture createFromStream (InputStream stream) | (已废弃)通过输入流创建一个Picture                    |
| public void writeToStream (OutputStream stream) | (已废弃)将Picture中内容写出到输出流中                  |

上面表格中基本上已经列出了Picture的所有方法，其中getWidth和getHeight没什么好说的，最后两个已经废弃也自然就不用关注了，排除了这些方法之后，只剩三个方法了，接下来我们就比较详细的了解一下：

**很明显，beginRecording 和 endRecording 是成对使用的，一个开始录制，一个是结束录制，两者之间的操作将会存储在Picture中。**

#### 使用示例：

**准备工作:**

录制内容，即将一些Canvas操作用Picture存储起来，录制的内容是不会直接显示在屏幕上的，只是存储起来了而已。

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

Picture虽然方法就那么几个，但是具体使用起来还是分很多情况的，由于录制的内容不会直接显示，就像存储的视频不点击播放不会自动播放一样，同样，想要将Picture中的内容显示出来就需要手动调用播放(绘制)，将Picture中的内容绘制出来可以有以下几种方法：

| 序号   | 简介                                       |
| ---- | ---------------------------------------- |
| 1    | 使用Picture提供的draw方法绘制。                    |
| 2    | 使用Canvas提供的drawPicture方法绘制。              |
| 3    | 将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。 |


以上几种方法主要区别：

| 主要区别         | 分类                    | 简介                                   |
| ------------ | --------------------- | ------------------------------------ |
| 是否对Canvas有影响 | 1有影响<br/>2,3不影响       | 此处指绘制完成后是否会影响Canvas的状态(Matrix clip等) |
| 可操作性强弱       | 1可操作性较弱<br/>2,3可操作性较强 | 此处的可操作性可以简单理解为对绘制结果可控程度。             |

几种方法简介和主要区别基本就这么多了，接下来对于各种使用方法一一详细介绍：

**1.使用Picture提供的draw方法绘制:**
``` java
        // 将Picture中的内容绘制在Canvas上
        mPicture.draw(canvas);  
```

<img src="http://ww1.sinaimg.cn/large/005Xtdi2jw1f2kwz9956lj30u01hcdg9.jpg" width = "300" />  

**PS：这种方法在比较低版本的系统上绘制后可能会影响Canvas状态，所以这种方法一般不会使用。**

**2.使用Canvas提供的drawPicture方法绘制**

drawPicture有三种方法：
``` java
public void drawPicture (Picture picture)

public void drawPicture (Picture picture, Rect dst)

public void drawPicture (Picture picture, RectF dst)
```
和使用Picture的draw方法不同，Canvas的drawPicture不会影响Canvas状态。

**简单示例:**

``` java
    canvas.drawPicture(mPicture,new RectF(0,0,mPicture.getWidth(),200));
```

<img src="http://ww4.sinaimg.cn/large/005Xtdi2jw1f2kwzseqawj30u01hc74o.jpg" width = "300"/>  

 **PS:对照上一张图片，可以比较明显的看出，绘制的内容根据选区进行了缩放。 **

**3.将Picture包装成为PictureDrawable，使用PictureDrawable的draw方法绘制。**

```
        // 包装成为Drawable
        PictureDrawable drawable = new PictureDrawable(mPicture);
        // 设置绘制区域 -- 注意此处所绘制的实际内容不会缩放
        drawable.setBounds(0,0,250,mPicture.getHeight());
        // 绘制
        drawable.draw(canvas);
```

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f2kx0bquw3j30u01hcdg8.jpg" width = "300" />  

**PS:此处setBounds是设置在画布上的绘制区域，并非根据该区域进行缩放，也不是剪裁Picture，每次都从Picture的左上角开始绘制。**

> **注意：在使用Picture之前请关闭硬件加速，以免引起不必要的问题，如何关闭请参考这里： [Android的硬件加速及可能导致的问题](https://github.com/GcsSloop/AndroidNote/issues/7)**

### (2)drawBitmap

 > 其实一开始知道要讲Bitmap我是拒绝的，为什么呢？因为Bitmap就是很多问题的根源啊有木有，Bitmap可能导致内存不足，内存泄露，ListView中的复用混乱等诸多问题。想完美的掌控Bitmap还真不是一件容易的事情。限于篇幅**本文对于Bitmap不会过多的展开，只讲解一些常用的功能**，关于Bitmap详细内容，以后开专题讲解QAQ。

 既然要绘制Bitmap，就要先获取一个Bitmap，那么如何获取呢？

 **获取Bitmap方式:**

| 序号   | 获取方式               | 备注                                       |
| ---- | ------------------ | ---------------------------------------- |
| 1    | 通过Bitmap创建         | 复制一个已有的Bitmap(_新Bitmap状态和原有的一致_) 或者 创建一个空白的Bitmap(_内容可改变_) |
| 2    | 通过BitmapDrawable获取 | 从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap    |
| 3    | 通过BitmapFactory获取  | 从资源文件 内存卡 网络等地方获取一张图片并转换为内容不可变的Bitmap    |

**通常来说，我们绘制Bitmap都是读取已有的图片转换为Bitmap绘制到Canvas上。**<br/>
很明显，第1种方式不能满足我们的要求，暂时排除。<br/>
第2种方式虽然也可满足我们的要求，但是我不推荐使用这种方式，至于为什么在后续详细讲解Drawable的时候会说明,暂时排除。<br/>
第3种方法我们会比较详细的说明一下如何从各个位置获取图片。<br/>

#### 通过BitmapFactory从不同位置获取Bitmap:

**资源文件(drawable/mipmap/raw):**
``` java
        Bitmap bitmap = BitmapFactory.decodeResource(mContext.getResources(),R.raw.bitmap);
```
**资源文件(assets):**
``` java
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
``` java
    Bitmap bitmap = BitmapFactory.decodeFile("/sdcard/bitmap.png");
```

**网络文件:**
``` java
        // 此处省略了获取网络输入流的代码
        Bitmap bitmap = BitmapFactory.decodeStream(is);
        is.close();
```

既然已经获得到了Bitmap，那么就开始本文的重点了，将Bitmap绘制到画布上。

#### 绘制Bitmap：

依照惯例先预览一下drawBitmap的常用方法：
``` java
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

``` java
    canvas.drawBitmap(bitmap,new Matrix(),new Paint());
```

> 关于Matrix和Paint暂时略过吧，一展开又是啰啰嗦嗦一大段，反正挖坑已经是常态了，大家应该也习惯了(PAP).

<img src="http://ww3.sinaimg.cn/large/005Xtdi2gw1f4ixnjrn83j30u01hcabc.jpg" width = "300" /> 

第二种方法就是在绘制时指定了图片左上角的坐标(距离坐标原点的距离)：

> **注意：此处指定的是与坐标原点的距离，并非是与屏幕顶部和左侧的距离, 虽然默认状态下两者是重合的，但是也请注意分别两者的不同。**

``` java
    canvas.drawBitmap(bitmap,200,500,new Paint());
```

<img src="http://ww3.sinaimg.cn/large/005Xtdi2gw1f4ixoug2x8j30u01hcgn4.jpg" width = "300" /> 

第三种方法比较有意思，上面多了两个矩形区域(src,dst),这两个矩形选区是干什么用的？

| 名称                  | 作用                |
| ------------------- | ----------------- |
| Rect src            | 指定绘制图片的区域         |
| Rect dst 或RectF dst | 指定图片在屏幕上显示(绘制)的区域 |

示例：
``` java
        // 将画布坐标系移动到画布中央
        canvas.translate(mWidth/2,mHeight/2);

        // 指定图片绘制区域(左上角的四分之一)
        Rect src = new Rect(0,0,bitmap.getWidth()/2,bitmap.getHeight()/2);

        // 指定图片在屏幕上显示的区域
        Rect dst = new Rect(0,0,200,400);

        // 绘制图片
        canvas.drawBitmap(bitmap,src,dst,null);
```
<img src="http://ww2.sinaimg.cn/large/005Xtdi2gw1f4ixqgk8rwj30u01hc756.jpg" width = "300" /> 

**详解：**

上面是以绘制该图为例，用src指定了图片绘制部分的区域，即下图中红色方框标注的区域。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f2kx2daw1qj305k05kq39.jpg)

然后用dst指定了绘制在屏幕上的绘制，即下图中蓝色方框标注的区域，图片宽高会根据指定的区域自动进行缩放。

<img src="http://ww2.sinaimg.cn/large/005Xtdi2gw1f4ixr3skcjj30u01hc3za.jpg" width = "300" /> 

从上面可知，第三种方法可以绘制图片的一部分到画布上，这有什么用呢？

如果你看过某些游戏的资源文件，你可能会看到如下的图片(图片来自网络)：

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f2kx3lk1ucj30kg04omz7.jpg)

用一张图片包含了大量的素材，在绘制的时候每次只截取一部分进行绘制，这样可以大大的减少素材数量，而且素材管理起来也很方便。

然而这和我们有什么关系呢？我们又不做游戏开发。

确实，我们不做游戏开发，但是在某些时候我们需要制作一些炫酷的效果，这些效果因为太复杂了用代码很难实现或者渲染效率不高。这时候很多人就会想起帧动画，将动画分解成一张一张的图片然后使用帧动画制作出来，这种实现方式的确比较简单，但是一个动画效果的图片有十几到几十张，一个应用里面来几个这样炫酷的动画效果就会导致资源文件出现一大堆，想找其中的某一张资源图片简直就是灾难啊有木有。但是把同一个动画效果的所有资源图片整理到一张图片上，会大大的**减少资源文件数量，方便管理**，妈妈再也不怕我找不到资源文件了，**同时也节省了图片文件头、文件结束块以及调色板等占用的空间。**

**下面是利用drawBitmap第三种方法制作的一个简单示例：**

资源文件如下：

![](https://raw.githubusercontent.com/GcsSloop/AndroidNote/master/CustomView/Advance/Res/Checkmark.png)

最终效果如下：

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f2kx67fkkog306g0b43yn.gif)

源码如下：

> PS:由于是示例代码，做的很粗糙，仅作为学习示例，不建议在任何实际项目中使用。

[_点击此处查看源码_](https://github.com/GcsSloop/AndroidNote/issues/10)

## 2.绘制文字
依旧预览一下相关常用方法：
``` java
    // 第一类
    public void drawText (String text, float x, float y, Paint paint)
    public void drawText (String text, int start, int end, float x, float y, Paint paint)
    public void drawText (CharSequence text, int start, int end, float x, float y, Paint paint)
    public void drawText (char[] text, int index, int count, float x, float y, Paint paint)
    
    // 第二类
    public void drawPosText (String text, float[] pos, Paint paint)
    public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)
    
    // 第三类
    public void drawTextOnPath (String text, Path path, float hOffset, float vOffset, Paint paint)
    public void drawTextOnPath (char[] text, int index, int count, Path path, float hOffset, float vOffset, Paint paint)
```
> PS 其中的CharSequence和String的区别可以到这里看看. [->戳这里<-](https://github.com/GcsSloop/AndroidNote/issues/16)

绘制文字部分大致可以分为三类：

第一类只能指定文本基线位置(基线x默认在字符串左侧，基线y默认在字符串下方)。<br/>
第二类可以分别指定每个文字的位置。<br/>
第三类是指定一个路径，根据路径绘制文字。<br/>

通过上面常用方法的参数也可看出，绘制文字也是需要画笔的，而且文字的大小,颜色,字体,对齐方式都是由画笔控制的。

不过嘛这里仅简单介绍几种常用方法(反正挖坑多了也不怕)，具体在讲解Paint时再详细讲解。

**Paint文本相关常用方法表**

| 标题   | 相关方法                      | 备注                                       |
| ---- | ------------------------- | ---------------------------------------- |
| 色彩   | setColor setARGB setAlpha | 设置颜色，透明度                                 |
| 大小   | setTextSize               | 设置文本字体大小                                 |
| 字体   | setTypeface               | 设置或清除字体样式                                |
| 样式   | setStyle                  | 填充(FILL),描边(STROKE),填充加描边(FILL_AND_STROKE) |
| 对齐   | setTextAlign              | 左对齐(LEFT),居中对齐(CENTER),右对齐(RIGHT)        |
| 测量   | measureText               | 测量文本大小(注意，请在设置完文本各项参数后调用)                |

为了绘制文本，我们先创建一个文本画笔：
``` java
        Paint textPaint = new Paint();          // 创建画笔
        textPaint.setColor(Color.BLACK);        // 设置颜色
        textPaint.setStyle(Paint.Style.FILL);   // 设置样式
        textPaint.setTextSize(50);              // 设置字体大小
```

### 第一类(drawText)
第一类可以指定文本开始的位置，可以截取文本中部分内容进行绘制。

其中x，y两个参数是指定文本绘制两个基线,示例：
``` java

        // 文本(要绘制的内容)
        String str = "ABCDEFGHIJK";

        // 参数分别为 (文本 基线x 基线y 画笔)
        canvas.drawText(str,200,500,textPaint);
```
<img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f3njfsi0l4j30dw0nuwey.jpg" width = "300" />  

> PS: 图中字符串下方的红线是基线y，基线x未在图中画出。

当然啦，除了能指定绘制文本的起始位置，还能只取出文本中的一部分内容进行绘制。

截取文本中的一部分，对于String和CharSequence来说只指定字符串下标start和end位置(**注意：0<= start < end < str.length()**)

以上一个例子使用的字符串为例，它的下标是这样的(wait，我为啥要说这个，算了，不管了，就这样吧(๑•́ ₃ •̀๑)):

| 字符   | A    | B    | C    | D    | E    | F    | G    | H    | I    | J    | K    |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 下标   | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |

假设我们指定start为1，end为3，那么最终截取的字符串就是"BC"。

一般来说，**使用start和end指定的区间是前闭后开的，即包含start指定的下标，而不包含end指定的下标**，故[1,3)最后获取到的下标只有 下标1 和 下标2 的字符，就是"BC"。

示例：
``` java 
        // 文本(要绘制的内容)
        String str = "ABCDEFGHIJK";
        // 参数分别为 (字符串 开始截取位置 结束截取位置 基线x 基线y 画笔)
        canvas.drawText(str,1,3,200,500,textPaint);
```
<img src="http://ww3.sinaimg.cn/large/005Xtdi2gw1f3njh66018j30dw0nuq3b.jpg" width = "300" />  

另外，对于字符数组char[]我们截取字符串使用起始位置(index)和长度(count)来确定。

同样，我们指定index为1，count为3，那么最终截取到的字符串是"BCD"。

其实就是从下标位置为1处向后数3位就是截取到的字符串，示例：
``` java
        // 字符数组(要绘制的内容)
        char[] chars = "ABCDEFGHIJK".toCharArray();
        
        // 参数为 (字符数组 起始坐标 截取长度 基线x 基线y 画笔)
        canvas.drawText(chars,1,3,200,500,textPaint);
```
<img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f3njhnldb5j30dw0nu74o.jpg" width = "300" />  

### 第二类(drawPosText)

通过和第一类比较，我们可以发现，第二类中没有指定x，y坐标的参数，而是出现了这样一个参数**float[] pos**。

好吧，这个名为pos的浮点型数组就是指定坐标的，至于为啥要用数组嘛，因为这家伙野心比较大，想给每个字符都指定一个位置。

示例：
``` java
        String str = "SLOOP";

        canvas.drawPosText(str,new float[]{
                100,100,    // 第一个字符位置
                200,200,    // 第二个字符位置
                300,300,    // ...
                400,400,
                500,500
        },textPaint);
```

<img src="http://ww2.sinaimg.cn/large/005Xtdi2jw1f2kx9fl8c8j30u01hcglz.jpg" width = "300" />  

不过嘛，虽然虽然这个方法也比较容易理解，但是关于这个方法我个人是不推荐使用的，因为坑比较多，主要有一下几点：

| 序号   | 反对理由                        |
| ---- | --------------------------- |
| 1    | 必须指定所有字符位置，否则直接crash掉，反人类设计 |
| 2    | 性能不佳，在大量使用的时候可能导致卡顿         |
| 3    | 不支持emoji等特殊字符，不支持字形组合与分解    |

关于第二类的第二种方法：

``` java
public void drawPosText (char[] text, int index, int count, float[] pos, Paint paint)
```

和上面一样，就是从字符数组中切出来一段进行绘制，相信以诸位看官的聪明才智一眼就看出来了，我这里就不多说了，真的不是我偷懒啊(ˉ▽￣～) ~~

### 第三类(drawTextOnPath)

第三类要用到path这个大杀器，作为一个厉害角色怎么能这么轻易露脸呢，先保持一下神秘感，也就是说，下回再和大家见面喽。

# 三.总结

学会了图片和文字绘制，对于大部分自定义View都能制作了，可以去看看这位大神制作的作品，尝试模仿一下[一个绚丽的loading动效分析与实现！](http://blog.csdn.net/tianjian4592/article/details/44538605)

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f2kxbki0wtg308c069mzr.gif)

(,,• ₃ •,,)

<b>PS: 由于本人英文水平有限，某些地方可能存在误解或词语翻译不准确，如果你对此有疑问可以提交Issues进行反馈。</b>

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300/> </a>

## 参考资料

[Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)<br/>
[Bitmap](http://developer.android.com/reference/android/graphics/Bitmap.html)<br/>
[Paint](http://developer.android.com/reference/android/graphics/Paint.html)<br/>
[Android ApiDemo 笔记（一）Content与Graphics](http://blog.csdn.net/wufenglong/article/details/5596402)<br/>

