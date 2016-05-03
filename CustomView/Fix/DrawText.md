# 番外篇 - 修正drawText的错误

由于个人的疏忽以及对问题的想当然，没有进行验证，在之前这篇文章 [【安卓自定义View进阶 - 图片文字】](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B4%5DCanvas_PictureText.md) 中出现了一个严重的错误，有不少眼睛雪亮的网友都发现了该错误并给予了反馈，非常感谢这些网友的帮助。

由于错误涉及的知识比较多，特此用一篇文章来修正错误，如果之前的文章让你你造成了误解，在此向你致以诚挚的歉意。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f3i3wsruf7j306o06o3yg.jpg)

## 错误原因

这个错误是drawText方法中坐标的一个问题，就一般的绘图而言，坐标一般都是指定左上角，然而drawText默认并不是左上角，甚至不是大家测试后认为的左下角，而是一个由paint中Align决定的对齐基准线，该API原文如下：

> Draw the text, with origin at (x,y), using the specified paint. **The origin is interpreted based on the Align setting in the paint.**



