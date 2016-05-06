# drawText

由于个人的疏忽以及对问题的想当然，没有进行验证，在之前这篇文章 [【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 中出现了一个严重的错误，有不少眼睛雪亮的网友都发现了该错误并给予了反馈，非常感谢这些网友的帮助。

由于这一部分涉及的知识也比较多，特此用一篇文章来修正错误以及详细讲解其中的各个知识点，如果之前的文章让你你造成了误解，在此向你致以诚挚的歉意。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f3i3wsruf7j306o06o3yg.jpg)

## 错误原因

这个错误是drawText方法中坐标的一个问题，就一般的绘图而言，坐标一般都是指定左上角，然而drawText默认并不是左上角，甚至不是大家测试后认为的左下角，而是一个**由Paint中Align决定的对齐基准线**，该API注释原文如下：

> Draw the text, with origin at (x,y), using the specified paint. **The origin is interpreted based on the Align setting in the paint.**

在默认情况下基线如下图所示：

> PS：基准线(以下简称基线)有两条，是用户指定的坐标确定的，在默认情况下，基线X在字符串左侧，基线y在字符串下方(但并不是最低的部分)。

![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3jppylp6zj30dw08c74v.jpg)

## 分析这样设计的原因

**Q: 为何基线y要放到文字下面，而不是上面？**

> A : 既然采用这种对齐方式，必然有其缘由，至少不是为了坑我这种小白。据本人猜测可能有以下原因：
* 1.符合人类书写习惯，不论是汉字还是英文或是其他语言，我们在书写对齐的时候都是以下面为基准线的，而不是上面，（**我们默认的基准线类似于四线格中的第三条线**）。<br/>
![](http://ww4.sinaimg.cn/large/005Xtdi2gw1f3knbp87ttj30dw08cq38.jpg)<br/>
* 2.字的特点，字的显示不仅有有中文汉字，还有一些特殊字符，并且大部分是根据下面对齐的，如果把对齐的基线放到上面并使其显示整齐，设计太麻烦，如下：<br/>
![](http://ww1.sinaimg.cn/large/005Xtdi2gw1f3knvptqsrj30dw08cmxw.jpg)<br/>
* 3.字体的特点，我们都知道，字有很多的字体，不同字体的大小是不同的，如果以以上面为基准线而让其底部对齐，这设计难度简直不敢想象：<br/>
![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3ko9oln9lj30dw08cmxo.jpg)<br/>
**综上所述，基线y放到下面不仅符合人的书写习惯，而且更加便于设计。**

## drawText从入门到懵逼

drawText是Canvas提供的一个方法，使用起来也比较简单，但是想玩出花样，单单的会使用这些方法是远远不够的。

**drawText方法的使用可以参见这一篇文章：[【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这里就不多啰嗦了，接下来我们将会了解一些更加好玩的东西。**

### drawText & Paint

虽然在 [图片文字](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这篇文章中有简单的了解部分方法，但并没有深入的去讲解，本次将会将与文本相关的部分全部摘出来讲解，至于其他内容以后再讲。

先了解Paint中与文本相关的内部类或者枚举：

名称           | 类型   | 主要作用
---------------|:------:|------------------------------------------------------
Align          | 枚举   | 指定基准线与文本的相对关系(包含 左对齐 右对齐 居中)
Style          | 枚举   | 设置样式，但不仅仅是为文本服务(包含 描边 填充 描边加填充)
FontMetrics    | 内部类 | 描述给定的文本大小，字体，间距等各种度量值(度量结果类型为float)
FontMetricsInt | 内部类 | 作用同上，但度量结果返回值是int型的

#### Align

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
        // 平移画布
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

#### Style

Style的意思是样式，这个在 [Canvas之绘制基本形状](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B2%5DCanvas_BasicGraphics.md) 这篇文章中讲过，它有三种状态：

枚举类型        | 作用
----------------|----------------
FILL            | 填充
STROKE          | 描边
FILL_AND_STROKE | 填充加描边

Style 对应的方法如下：

``` java

```







