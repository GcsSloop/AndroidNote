# 番外篇 - 修正drawText的错误

由于个人的疏忽以及对问题的想当然，没有进行验证，在之前这篇文章 [【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 中出现了一个严重的错误，有不少眼睛雪亮的网友都发现了该错误并给予了反馈，非常感谢这些网友的帮助。

由于错误涉及的知识比较多，特此用一篇文章来修正错误，如果之前的文章让你你造成了误解，在此向你致以诚挚的歉意。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f3i3wsruf7j306o06o3yg.jpg)

## 错误原因

这个错误是drawText方法中坐标的一个问题，就一般的绘图而言，坐标一般都是指定左上角，然而drawText默认并不是左上角，甚至不是大家测试后认为的左下角，而是一个**由Paint中Align决定的对齐基准线**，该API原文如下：

> Draw the text, with origin at (x,y), using the specified paint. **The origin is interpreted based on the Align setting in the paint.**

在默认情况下基线如下图所示：

## 分析这样设计的原因

**Q: 为何要采用底部基线对齐方式？**

> A : 既然采用这种对齐方式，必然有其缘由，至少不是为了坑我这种小白。据本人猜测可能有以下原因：
* 1.符合人类书写习惯，不论是汉字还是英文或是其他语言，我们在只有横线的纸上书写的时候都是以下面的一条线为基准线的，而不是上面那条线。
* 2.字体的特点，字的显示不仅有有中文汉字，还有一些特殊字符，如果把对齐的基线放到上面要么显示整齐，要么设计太麻烦。






