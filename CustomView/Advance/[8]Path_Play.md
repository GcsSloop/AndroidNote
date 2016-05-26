# Path之玩出花样

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView)



可以看到，在经过 
[Path之基本操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_Basic.md)
[Path之贝塞尔曲线](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B6%5DPath_Bezier.md) 和 
[Path之完结篇(伪)](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B7%5DPath_Over.md) 后， Path中各类方法基本上都讲完了，表格中还没有讲解到到方法就是矩阵变换了，难道本篇终于要讲矩阵了？
非也，矩阵这一部分仍在后面单独讲解，本篇主要讲解PathMeasure这个类与Path的一些使用技巧。

> PS：不要问我为什么不讲PathEffect，因为这个方法在后面的Paint系列中。

******

## PathMeasure

顾名思义，PathMeasure是一个用来测量Path的类，主要有以下方法:

### 构造方法

方法名 | 释义
---|---
PathMeasure() | 创建一个空的PathMeasure
PathMeasure(Path path, boolean forceClosed) | 创建PathMeasure并关联一个指定的Path(Path需要已经创建完成)。

### 公共方法

返回值  | 方法名                                                                   | 释义
--------|--------------------------------------------------------------------------|-------------------
void    | setPath(Path path, boolean forceClosed)                                  | 关联一个Path
boolean | isClosed()                                                               | 是否闭合
float   | getLength()                                                              | 获取Path的长度
boolean |	nextContour()                                                            | 跳转到下一个轮廓
boolean | getSegment(float startD, float stopD, Path dst, boolean startWithMoveTo) | 截取片段
boolean | getPosTan(float distance, float[] pos, float[] tan)                      | 获取指定长度的位置坐标及该点切线值
boolean | getMatrix(float distance, Matrix matrix, int flags)                      | 获取指定长度的位置坐标及该点Matrix

PathMeasure的方法也不多，接下来我们就逐一的讲解一下。

#### 构造函数

构造函数有两个。

**无参构造函数：**

``` java
  PathMeasure ()
```

用这个构造函数可创建一个空的PathMeasure，但是使用之前需要先调用 setPath 方法来与 Path 进行关联。被关联的 Path 必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

**有参构造函数：**

``` java
  PathMeasure (Path path, boolean forceClosed)
```

用这个构造函数是创建一个 PathMeasure 并关联一个 Path， 其实和创建一个空的 PathMeasure 后调用 setPath 进行关联效果是一样的，同样，被关联的 Path 也必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

该方法有两个参数，第一个参数自然就是被关联的Path了，第二个参数是用来确保 Path 闭合，如果设置为 true， 则不论之前Path是否闭合，都会自动闭合该 Path。

**在这里有两点需要明确:**

> 
* 1. 不论 forceClosed 设置为何种状态(true 或者 false)， 都不会影响原有Path的状态，**即 Path 与 PathMeasure 关联之后，Path不会有任何改变。**
* 2. forceClosed 的设置状态可能会影响测量结果，**如果 Path 未闭合但在与 PathMeasure 关联的时候设置 forceClosed 为 true 时，测量结果可能会比 Path 实际长度稍长一点。**

下面我们用一个例子来验证一下：

```
    canvas.translate(mViewWidth/2,mViewHeight/2);

    Path path = new Path();

    path.lineTo(0,200);
    path.lineTo(200,200);
    path.lineTo(200,0);

    PathMeasure measure1 = new PathMeasure(path,false);
    PathMeasure measure2 = new PathMeasure(path,true);

    Log.e("TAG", "forceClosed=false---->"+measure1.getLength());
    Log.e("TAG", "forceClosed=true----->"+measure2.getLength());

    canvas.drawPath(path,mDeafultPaint);
```

log如下:
```
 25521-25521/com.gcssloop.canvas E/TAG: forceClosed=false---->600.0
 25521-25521/com.gcssloop.canvas E/TAG: forceClosed=true----->800.0
```

绘制在界面上的效果如下:




## 总结


## About Me

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)

<a href="https://github.com/GcsSloop/README/blob/master/README.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300 height=100 /> </a>

## 参考资料
[]()<br/>
[]()<br/>
