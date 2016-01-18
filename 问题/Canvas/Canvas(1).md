# Canvas
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

上一次我们了解自定义View的流程，以及几个重要的相关函数，不过，这些东西依旧还是理论，只能用来了解一下原理，并不能<b>拿来(zhuang)用(B)</b>, 这一次我们就了解一些<b>能(zhaung)用(B)</b>的东西。
关于本文，我们先了解Canvas的基本用法，再学习一个能拿来装逼(就是看起来很牛逼，但是没卵用)的东西。

## 一.Canvas简介
Canvas可以说是一大利器(用于2D绘图)，因为涉及到的东西比较基础。

<b>比较基础的东西一般有两大特点:<br/>
  1.可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。<br/>
  2.比较难用：各种方法太过基础，想要比较完美的组合起来有一定难度，并非不会这些操作，而是不知如何组织这些操作。</b>

不过不必担心，下面不仅会介绍到Canvas的操作方法，还会简单介绍一些设计思路和技巧。

## 二.Canvas基础
Canvas我们可以称之为画布，在上面绘制各种东西。

绘制的<b>基本形状</b>由<b>Canvas</b>确定，但绘制出来的<b>颜色,具体效果</b>则由<b>Paint</b>确定。

本次内容仅简单讲解Canvas所能绘制的基本内容，对Paint有一点涉及但不会涉及太多，关于Paint的更多用法会在以后详细想介绍，下面开始正题：

### Canvas的基本内容：

Canvas的常用操作如下：

操作类型 | 相关API | 备注
--- | --- | ---
绘制颜色 | drawColor, drawRGB, drawARGB | [安卓自定义View基础-颜色](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E9%A2%9C%E8%89%B2/%E9%A2%9C%E8%89%B2.md)
绘制图片 | drawBitmap, drawPicture, drawBitmapMesh | drawBitmapMesh为带形变的绘制Bitmap
绘制文本 | drawText, 	drawPosText, drawTextOnPath, drawTextRun | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字、
绘制基本形状 | drawPoint, drawLine, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制路径 | drawPath | 绘制贝塞尔曲线需要用到该函数
绘制定点 | drawVertices |  
剪裁画布 | clipPath, 	clipRect, clipRegion(API21时废弃) | 设置画布的显示区域

变换Canvas：save、restore、translate、scale、rotate、concat(Matrix matrix)、setMatrix(Matrix matrix)






