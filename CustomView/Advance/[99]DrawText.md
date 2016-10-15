# drawText之坐标、居中、绘制多行

本文用于讲解Canvas中关于DrawText相关的一些常见问题，如坐标，居中，和绘制多行。

之前由于个人的疏忽以及对问题的想当然，没有进行验证，在 [【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这篇文章中出现了一个错误，有不少眼睛雪亮的网友都发现了该错误并给予了反馈，非常感谢这些网友的帮助。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f3i3wsruf7j306o06o3yg.jpg)

## 错误原因

这个错误是drawText方法中坐标的一个问题，就一般的绘图而言，坐标一般都是指定左上角，然而drawText默认并不是左上角，甚至不是大家测试后认为的左下角，而是一个**由Paint中Align决定的对齐基准线**，该API注释原文如下：

> Draw the text, with origin at (x,y), using the specified paint. **The origin is interpreted based on the Align setting in the paint.**

在默认情况下基线如下图所示：

> PS：基线(BaseLine)有两条，是用户指定的坐标确定的，在默认情况下，基线X在字符串左侧，基线y在字符串下方(但并不是最低的部分)。

![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3jppylp6zj30dw08c74v.jpg)

## 分析这样设计的原因

**Q: 为何基线y要放到文字下面，而不是上面？**

> A : 既然采用这种对齐方式，必然有其缘由，至少不是为了坑我这种小白。据本人猜测可能有以下原因：
* 1.符合人类书写习惯，不论是汉字还是英文或是其他语言，我们在书写对齐的时候都是以下面为基准线的，而不是上面，（**我们默认的基准线类似于四线格中的第三条线**）。<br/>
![](http://ww4.sinaimg.cn/large/005Xtdi2gw1f3knbp87ttj30dw08cq38.jpg)<br/>
* 2.字的特点，字的显示不仅有有中文汉字，还有一些特殊字符，并且大部分是根据下面对齐的，如果把对齐的基线放到上面并使其显示整齐，设计太麻烦，如下：<br/>
![](http://ww1.sinaimg.cn/large/005Xtdi2gw1f3knvptqsrj30dw08cmxw.jpg)<br/>
* 3.字体的特点，我们都知道，字有很多的字体风格，不同字体风格的大小是不同的，如果以以上面为基准线而让其底部对齐，这设计难度简直不敢想象：<br/>
![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3ko9oln9lj30dw08cmxo.jpg)<br/>
**综上所述，基线y放到下面不仅符合人的书写习惯，而且更加便于设计。**

## drawText从入门到懵逼

虽然之前在 [图片文字](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这篇文章中已经简单的了解部分关于文字的方法，但Paint中关于文字的方法还有很多，本文会了解一些我们比较关心的一些内容，例如绘制居中的文本，多行文本等，在此之前我们先了解一下Paint中与文本相关的内部类或者枚举：

名称           | 类型   | 主要作用
---------------|:------:|------------------------------------------------------
Align          | 枚举   | 指定基准线与文本的相对关系(包含 左对齐 右对齐 居中)
Style          | 枚举   | 设置样式，但不仅仅是为文本服务(包含 描边 填充 描边加填充)
FontMetrics    | 内部类 | 描述给定的文本大小，字体，间距等各种度量值(度量结果类型为float)
FontMetricsInt | 内部类 | 作用同上，但度量结果返回值是int型的

### Align

Align中文意思是对齐，其作用正式如此，我们使用过 Word 的人都知道文本有 *左对齐、居中、右对齐、两端对齐、分散对齐* 五种模式，Align作用就是设置文本的对齐模式，但是并没有这么多，仅有 **左对齐、居中、右对齐** 三种模式，如下：

> 吐槽：就因为没有两端对齐和分散对齐，导致长文本，尤其是中英混合的长文本在手机上显示效果奇差，相信做阅读类软件的程序员深有体会，最终还要自定义View解决。

枚举类型 | 作用
---------|-------------------------------------------------------------
LEFT     | 左对齐   基线X在文本左侧，基线y在文本下方，文本出现在给定坐标的右侧，是默认的对齐方式
RIGHT    | 右对齐   基线x在文本右侧，基线y在文本下方，文本出现在给定坐标的左侧
CENTER   | 居中对齐 基线x在文本中间，基线y在文本下方，文本出现在给定坐标的两侧

Align对应的方法如下：

``` java
  public Paint.Align getTextAlign ()              // 获取对齐方式
  
  public void setTextAlign (Paint.Align align)    // 设置对齐方式
```

在实际运用中基线与模式之间的关系则如下图所示：

![](http://ww2.sinaimg.cn/large/005Xtdi2gw1f3l2d6gw0nj30dw08c74q.jpg)

> PS 该图片是截屏加工所得，其中红色的先是各自的基准线。注意汉字有一部分是在基准线下面的。

其核心代码如下：

``` java
        // 平移画布 - 如有疑问请参考画布操作这篇文章
        canvas.translate(mCenterX,0);   // mCenterX 是屏幕宽度的一半，在onSizeChanged中获取

        // 设置字体颜色
        mPaint.setColor(Color.rgb(0x06,0xaf,0xcd)); 

        // 在不同模式下绘制文字
        mPaint.setTextAlign(Paint.Align.LEFT);
        canvas.drawText("左对齐", 0, 200, mPaint);

        mPaint.setTextAlign(Paint.Align.CENTER);
        canvas.drawText("居中对齐", 0, 400, mPaint);

        mPaint.setTextAlign(Paint.Align.RIGHT);
        canvas.drawText("右对齐", 0, 600, mPaint);
```

### Style

Style的意思是样式，这个在 [Canvas之绘制基本形状](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B2%5DCanvas_BasicGraphics.md) 这篇文章中讲过，它有三种状态：

枚举类型        | 作用
----------------|----------------
FILL            | 填充
STROKE          | 描边
FILL_AND_STROKE | 填充加描边

Style 对应的方法如下：

``` java
public Paint.Style getStyle ()              // 获取当然样式

public void setStyle (Paint.Style style)    // 设置样式
```

效果如下：

![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3nh9r8ih0j30dw08c3z4.jpg)

核心代码：

``` java
        // 平移画布 - 如有疑问请参考画布操作这篇文章
        canvas.translate(mCenterX,0);   // mCenterX 是屏幕宽度的一半，在onSizeChanged中获取

        mPaint.setTextAlign(Paint.Align.CENTER);
        mPaint.setColor(Color.rgb(0x06,0xaf,0xcd));

        // 设置不同的样式

        // 填充
        mPaint.setStyle(Paint.Style.FILL);
        canvas.drawText("GcsSloop 中文", 0, 200, mPaint);

        // 描边
        mPaint.setStyle(Paint.Style.STROKE);
        canvas.drawText("GcsSloop 中文", 0, 400, mPaint);

        // 描边加填充
        mPaint.setStyle(Paint.Style.FILL_AND_STROKE);
        canvas.drawText("GcsSloop 中文", 0, 600, mPaint);
```

### FontMetrics

FontMetrics 是 Paint 的一个内部类，根据名字我们可以大致知道这是一个描述**字体规格**相关的类，关于这个类的描述原文是这样的：

> Class that describes the various metrics for a font at a given text size. Remember, Y values increase going down, so those values will be positive, and values that measure distances going up will be negative. This class is returned by getFontMetrics().

翻译一下：

> 简单来说，FontMetrics包含了当前文本相关的各项参数，你可以通过 Paint中 _getFontMetrics()_ 这个方法来获取到这个类。
(另外，原文中还特地强调了一下关于坐标正负的问题。）

##### FontMetrics包含了以下几个数值：

> 如果没有特殊说明，一下文章中“基线”默认代表“基线Y”。

名称    | 正负| 释义
--------|:---:|--------------------------------------------------------
top     | -   | 在指定字形和大小的情况下，字符最高点与基线之间的距离
ascent  | -   | 单个字符的最高点与基线距离的推荐值(不包括字母上面的符号)
descent | +   | 单个字符的最低点与基线距离的推荐值
bottom  | +   | 在指定字形和大小的情况下，字符最低点与基线之间的距离
leading	| +   | 行间距,当前行bottom与下一行top之间的距离的推荐值 (通常为0，因为top与ascent，bottom与leading之间的距离足够作为行间距了)

![](http://ww1.sinaimg.cn/large/005Xtdi2gw1f3syr8fbd1j30dw08c755.jpg)

看了上面啰啰嗦嗦讲了一堆，你可能会产生一些疑问，这里我简单解释一下：

0、 FontMetrics到底有什么用？

> FontMetrics 是字体规格，比较精确的测量出文字相关的各种数据，在很多地方都是用得到的，比较常见的就是我们的音乐播放器中的歌词显示，需要实时的变字体位置，这里就需要比较精确是数值来进行计算以确保不会出现两行文字重叠等问题。

1、为什么表格中最高点距离基线的值是负值，而最低点反而是正值？

> 这是因为手机屏幕坐标系的特殊性，在数学中常见的坐标系y轴是向上的，而对于手机而言，y轴默认是向下的，所以数值越小越靠近屏幕顶端，即最高点的数值比基线小，最高点减去基线结果自然为负数。 更多参考  [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Base/%5B1%5DCoordinateSystem.md) 这篇文章。

2、为什么表格中很多都是推荐值？

> 这是因为字的规格不仅受字大小的影响，而且受字体风格的影响，不同的字体风格差别很大，其结果可能会有偏差，故大部分都是推荐值。

3、 具体绘制的文本不同会影响 FontMetrics 中的数值吗？

> 不会， **FontMetrics中的数值受字体大小(FontSize) 和 字体风格(Typeface) 影响**， 而不会因为具体绘制的内容不同而改变，给Paint设置完字体大小和字体风格后就能获取到正确的FontMetrics。

**获取FontMetrics的方法**
``` java
    Paint.FontMetrics mMetrics = mPaint.getFontMetrics();
    
    Log.e("TAG", "top=" + mMetrics.top);
    Log.e("TAG", "ascent=" + mMetrics.ascent);
    Log.e("TAG", "descent=" + mMetrics.descent);
    Log.e("TAG", "bottom=" + mMetrics.bottom);
    Log.e("TAG", "leading=" + mMetrics.leading);
```
结果：
```
05-15 21:18:13.315 13112-13112/com.gcssloop.canvas E/TAG: top=-152.98828
05-15 21:18:13.315 13112-13112/com.gcssloop.canvas E/TAG: ascent=-129.88281
05-15 21:18:13.315 13112-13112/com.gcssloop.canvas E/TAG: descent=34.179688
05-15 21:18:13.315 13112-13112/com.gcssloop.canvas E/TAG: bottom=37.939453
05-15 21:18:13.315 13112-13112/com.gcssloop.canvas E/TAG: leading=0.0
```

**由于FontMetrics中的 ascent 与 descent 比较常用，所以可以直接通过Paint获取：**
``` java
    Log.e("TAG", "P.ascent=" + mPaint.ascent());
    Log.e("TAG", "P.descent=" + mPaint.descent());
```
结果，可以看到与通过FontMetrics获得的结果相同。
``` 
05-15 21:24:18.950 13112-13112/com.gcssloop.canvas E/TAG: P.ascent=-129.88281
05-15 21:24:18.955 13112-13112/com.gcssloop.canvas E/TAG: P.descent=34.179688
```


## 文字居中

对于绘制居中的文本来说，我们可以封装一个方法用中心点作为绘制坐标，在绘制的时候转换为实际坐标。

根据前面的知识可知，想让文字水平居中很容易，只需要设置 TextAlign 为 CENTER，那么基线X(BaseLineX)自然就是这个字符串的中心了。

而让文字垂直居中则有些麻烦了，因为基线Y(BaseLineY)既不是顶部，底部，也不是中心，所以文本居中最难的内容就是计算BaseLineY的位置。

我们已知：

> 1. 中心位置坐标，centerX, centerY
2. 文本高度：height ＝ descent－ascent
3. descent的坐标：descentY ＝ centerY ＋ 1/2height
4. baseLineY坐标：baseLineY ＝ descentY－descent

通过上面内容可以推算出如下公式：

> baseLineY ＝ centerY － 1/2(ascent + descent)

**根据公式我们可以封装一个方法：**

``` java
    /**
     * 居中绘制文本
     *
     * @param text
     * @param canvas
     * @param paint
     */
    public void drawTextByCenter(String text, float x, float y, Canvas canvas, Paint paint) {
        Paint tempPaint = new Paint(paint);             // 创建一个临时画笔,防止影响原来画笔的状态
        tempPaint.setTextAlign(Paint.Align.CENTER);     // 设置文本对齐方式为居中

        // 通过y计算出baseline的位置
        float baseline = y - (tempPaint.descent() + tempPaint.ascent()) / 2;   

        canvas.drawText(text, x, baseline, tempPaint);  //绘制文本
    }
```

**测试方法是否正确**

``` java
    canvas.translate(mCenterX, mCenterY);

    String text = "ASSA";
    mPaint.setColor(Color.BLACK);

    drawTextByCenter(text, 0, 0, canvas, mPaint);
```

结果：

<img src="http://ww3.sinaimg.cn/large/005Xtdi2jw1f3wlau69aaj30u01hcgmd.jpg" width=300 />

## 绘制多行

前面讲的所有绘制文本都是绘制单行文本，而对于长文本就束手无策了，但是我们偶尔也需要绘制一些长文本，例如绘制一段文本，这样前面的就无法满足需求了。

我们先思考一下如何才能绘制一段长文本让其可以自动换行，如自动测量文本长度，然后根据测量的内容截开成n行，然后一行一行的绘制。









