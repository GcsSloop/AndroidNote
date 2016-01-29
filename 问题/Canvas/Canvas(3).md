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

### drawPicture
drawPicture有三种方法：
```
public void drawPicture (Picture picture)

public void drawPicture (Picture picture, Rect dst)

public void drawPicture (Picture picture, RectF dst)
```
关于drawPicture还是挺让人费解的，不过嘛，我们慢慢研究一下他的用途。

既然是drawPicture就要了解一下什么是Picture。 顾名思义，Picture的意思是图片。

不过嘛，我觉得这么用图片这个名词解释Picture是不合适的，为何这么说？请看其官方文档对[Picture](http://developer.android.com/reference/android/graphics/Picture.html)的解释：

<i>
A Picture records drawing calls (via the canvas returned by beginRecording) and can then play them back into Canvas (via draw(Canvas) or drawPicture(Picture)).For most content (e.g. text, lines, rectangles), drawing a sequence from a picture can be faster than the equivalent API calls, since the picture performs its playback without incurring any method-call overhead.
</i>

好吧，我知道大部分人对这段鸟语是看不懂的，至于为什么要放在这里，仅仅是为了显得更加专业(偷笑)。

<b>那下面我就对这段不明觉厉的鸟语用通俗的话翻译一下:</b>

某一天小萌想在朋友面前显摆一下，于是在单杠上来了一个后空翻,动作姿势请参照下图：

朋友都说 恩，很不错。 想再看一遍(〃ω〃)。ヽ(〃∀〃)ﾉ。⁄(⁄ ⁄•⁄ω⁄•⁄ ⁄)⁄

于是小萌又来了一遍，如是几次之后，小萌累的吐血三升。

于是小萌机智的想，我何不用手机录下来呢，直接保存成为<b>后空翻.avi</b> 下次在想显摆的时候拿出手机，点一下播放就行了，省时省力。

小萌深深的被自己的机智折服了，然后Picture就诞生啦。(╯‵□′)╯︵┻━┻掀桌，坑爹呢，这剧情跳跃也忒大了吧。

<b>
好吧，言归正传，这次我们了解的Picture和上文中的录像功能是类似的，只不过我们Picture所录的是Canvas的内容。<br/>
我们把Canvas绘制点，线，矩形等诸多操作用Picture录制下来，下次需要的时候拿来就能用，使用Picture相比于再次调用绘图API，开销是比较小的。
</b>


