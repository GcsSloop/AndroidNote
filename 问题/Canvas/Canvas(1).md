# Canvas基础一
### 讲解安卓中的Canvas基础内容。
### 作者微博: [@攻城师sloop](http://weibo.com/5459430586)

上一次我们了解自定义View的流程，以及几个重要的相关函数，不过，这些东西依旧还是理论，并不能<b>拿来(zhuang)用(B)</b>, 这一次我们就了解一些<b>能(zhaung)用(B)</b>的东西。
关于本文，我们先了解Canvas的基本用法，再学习一个能拿来装逼(就是看起来很牛逼，但是没卵用)的东西。

## 一.Canvas简介
Canvas可以说是一大利器(在2D绘图方面)，因为涉及到的东西比较基础。

<b>比较基础的东西一般有两大特点:<br/>
  1.可操作性强：由于这些是构成上层的基础，所以可操作性必然十分强大。<br/>
  2.比较难用：各种方法太过基础，想要比较完美的组合起来有一定难度，并非不会这些操作，而是不知如何组织这些操作。</b>

不过不必担心，下面不仅会介绍到Canvas的操作方法，还会简单介绍一些设计思路和技巧。

## 二.Canvas基础
Canvas我们可以称之为画布，在上面绘制各种东西。

绘制的<b>基本形状</b>由<b>Canvas</b>确定，但绘制出来的<b>颜色,具体效果</b>则由<b>Paint</b>确定。

本次内容仅简单讲解Canvas所能绘制的基本内容，对Paint有一点涉及但不会涉及太多，关于Paint的更多用法会在以后详细想介绍，下面开始正题：

### Canvas的常用操作：

操作类型 | 相关API | 备注
--- | --- | ---
绘制颜色 | drawColor, drawRGB, drawARGB | 使用单一颜色填充整个画布<br/> 相关 : [安卓自定义View基础-颜色](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E9%A2%9C%E8%89%B2/%E9%A2%9C%E8%89%B2.md)
绘制基本形状 | drawPoint, drawPoints, drawLine, drawLines, drawRect, drawRoundRect, drawOval, drawCircle, drawArc | 依次为 点、线、矩形、圆角矩形、椭圆、圆、圆弧
绘制图片 | drawBitmap, drawPicture | 绘制位图和图片
绘制文本 | drawText, 	drawPosText, drawTextOnPath | 依次为 绘制文字、绘制文字时指定每个文字位置、根据路径绘制文字
绘制路径 | drawPath | 绘制贝塞尔曲线需要用到该函数
顶点操作 | drawVertices, drawBitmapMesh | 通过对顶点操作可以使图像形变，drawVertices直接对画布作用、 drawBitmapMesh只对绘制的Bitmap作用
画布剪裁 | clipPath, 	clipRect | 设置画布的显示区域
画布快照 | save, restore | 两者一般成对使用，save是保存当前状态，restore是回滚到上一次保存的状态
画布形变 | translate, scale, rotate, skew | 依次为 位移、缩放、 旋转、倾斜<br/> 相关： [坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md)    [角度与弧度](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6/%E8%A7%92%E5%BA%A6%E4%B8%8E%E5%BC%A7%E5%BA%A6.md)
Matrix(矩阵)操作 | getMatrix, setMatrix, concat | 实际画布的位移，缩放等操作的都是图像矩阵Matrix，只不过Matrix比较难以理解和使用，故封装了一些常用的方法。

PS： Canvas常用方法在上面表格中基本全部列出了，当然还存在一些其他的方法未列出，具体可以官方文档 [Canvas](http://developer.android.com/reference/android/graphics/Canvas.html)

从上表可以看出，Canvas的可用操作还是有很多的，限于篇幅，本文仅讲解 绘制颜色， 绘制基本形状，简要介绍画布形变操作，最后会利用以上知识做一个小实例用来<b>练(zhaung)手(B)</b>，其余内容会在以后的文章中更加详细的介绍。

******
### Canvas基础示例：

#### 绘制颜色：
绘制颜色是填充整个画布，常用于绘制底色。
```
  canvas.drawColor(Color.BLUE); //绘制蓝色
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art/drawColor.jpeg" width = "270" height = "480" alt="title" align=center />  
  
#### 创建画笔：
要想绘制内容，首先需要先创建一个画笔，如下：
```
  // 1.创建一个画笔
  private Paint mPaint = new Paint();
  
  // 2.初始化画笔
    private void initPaint() {
        mPaint.setColor(Color.BLACK);       //设置画笔颜色
        mPaint.setStyle(Paint.Style.FILL);  //设置画笔模式为填充
        mPaint.setStrokeWidth(10f);         //设置画笔宽度为10px
    }
  
  // 3.在构造函数中初始化
    public SloopView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }
```
在创建完画笔之后，就可以在Canvas中绘制各种内容了。

#### 绘制点：
可以绘制一个点，也可以绘制一组点，如下：
```
        canvas.drawPoint(200, 200, mPaint);     //在坐标(200,200)位置绘制一个点
        canvas.drawPoints(new float[]{          //绘制一组点，坐标位置由float数组指定
                500,500,
                500,600,
                500,700
        },mPaint);
```
关于坐标原点默认在左上角，水平向右为x轴增大方向，竖直向下为y轴增大方向。更多参考这里 [安卓自定义View基础 - 坐标系](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/%E5%9D%90%E6%A0%87%E7%B3%BB.md)

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art/drawPoint.jpeg" width = "270" height = "480" alt="title" align=center />  
#### 绘制直线：
绘制直线需要两个点，初始点和结束点，同样绘制直线也可以绘制一条或者绘制一组：
```
        canvas.drawLine(300,300,500,600,mPaint);    // 在坐标(300,300)(500,600)之间绘制一条直线
        canvas.drawLines(new float[]{               // 绘制一组线 每四数字(两个点的坐标)确定一条线
                100,200,200,200,
                100,300,200,300
        },mPaint);
```

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art/drawLine.jpeg" width = "270" height = "480" alt="title" align=center />  
#### 绘制矩形：
确定确定一个矩形最少需要四个数据，就是对角线的两个点的坐标值，通常我们会采用<b>左上角</b>和<b>右下角</b>的两个点的坐标(当然了右上和左下也可以)，像这样：

![](https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/%E5%9D%90%E6%A0%87%E7%B3%BB/Art/%E5%9D%90%E6%A0%87%E7%B3%BB5.png)

关于绘制矩形，Canvas提供了<b>三种重载</b>方法，第一种就是提供<b>四个数值(对角线两个点的坐标)来确定一个矩形</b>进行绘制。
其余两种是先将矩形封装为<b>Rect</b>或<b>RectF</b>(实际上仍然是用两个坐标点来确定的矩形)，然后传递给Canvas绘制，如下：
```
        // 第一种
        canvas.drawRect(100,100,800,400,mPaint);

        // 第二种
        Rect rect = new Rect(100,100,800,400);
        canvas.drawRect(rect,mPaint);

        // 第三种
        RectF rectF = new RectF(100,100,800,400);
        canvas.drawRect(rectF,mPaint);
```
以上三种方法所绘制出来的结果是完全一样的。

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art/drawRect.jpeg" width = "270" height = "480" alt="title" align=center /> 

看到这里,相信很多观众会产生一个疑问，<b>为什么会有Rect和RectF两种？两者有什么区别吗？</b>

答案当然是存在区别的，两种最大的区别就是<b>精度不同</b>，<b>Rect</b>是<b>int(整形)</b>的，而<b>RectF</b>则是<b>float(单精度浮点型)</b>的。当然了除了精度不同，两种提供的方法也稍微存在差别，在这里我们暂时无需关注，想了解更多参见官方文档 [Rect](http://developer.android.com/reference/android/graphics/Rect.html) 和 [RectF](http://developer.android.com/reference/android/graphics/RectF.html)

#### 绘制圆角矩形：
绘制圆角矩形也提供了两种重载方式，如下：
```
        // 第一种
        RectF rectF = new RectF(100,100,800,400);
        canvas.drawRoundRect(rectF,30,30,mPaint);
        
        // 第二种
        canvas.drawRoundRect(100,100,800,400,30,30,mPaint);
```
上面两种方法绘制效果也是一样的，但鉴于第二种方法在API21的时候才添加上，所以我们一般使用的都是第一种。

<img src="https://github.com/GcsSloop/AndroidNote/blob/master/%E9%97%AE%E9%A2%98/Canvas/Art/drawRoundRect1.jpeg" width = "270" height = "480" alt="title" align=center /> 

下面简单解析一下圆角矩形的几个必要的参数的意思。

#### 绘制椭圆：

#### 绘制圆：

#### 绘制圆弧：



