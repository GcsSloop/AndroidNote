# 番外篇 - 修正drawText的错误

由于个人的疏忽以及对问题的想当然，没有进行验证，在之前这篇文章 [【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 中出现了一个严重的错误，有不少眼睛雪亮的网友都发现了该错误并给予了反馈，非常感谢这些网友的帮助。

由于错误涉及的知识比较多，特此用一篇文章来修正错误，如果之前的文章让你你造成了误解，在此向你致以诚挚的歉意。

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
* 3.字体的特点，我们都知道，字有很多的字体，不同字体的大小是不同的，如果以以上面为基准线，这设计难度简直不敢想象：<br/>
![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f3ko9oln9lj30dw08cmxo.jpg)<br/>
**综上所述，基线y放到下面不仅符合人的书写习惯，而且更加便于设计。**

## drawText从入门到懵逼

drawText是Canvas提供的一个方法，使用起来也比较简单，但是想玩出花样，单单的会使用这些方法是远远不够的。

**drawText方法的使用可以参见这一篇文章：[【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这里就不多啰嗦了，接下来我们将会了解一些更加好玩的东西。**

### drawText & Paint

虽然在 [图片文字](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 这篇文章中有简单的了解部分方法，但并没有深入的去讲解，本次将会详细的讲解其中的各个方法。









