# Path之玩出花样(PathMeasure)

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### [【本系列相关文章】](https://github.com/GcsSloop/AndroidNote/tree/master/CustomView/README.md)


可以看到，在经过 
[Path之基本操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_Basic.md)
[Path之贝塞尔曲线](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B6%5DPath_Bezier.md) 和 
[Path之完结篇(伪)](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B7%5DPath_Over.md) 后， Path中各类方法基本上都讲完了，表格中还没有讲解到到方法就是矩阵变换了，难道本篇终于要讲矩阵了？
非也，矩阵这一部分仍在后面单独讲解，本篇主要讲解 PathMeasure 这个类与 Path 的一些使用技巧。

> PS：不要问我为什么不讲 PathEffect，因为这个方法在后面的Paint系列中。

先放一个图镇楼，省的下面无聊的内容把你们都吓跑了Σ(￣。￣ﾉ)ﾉ

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f4fp2myqo4g308c05k75k.gif)

******

##  Path & PathMeasure

顾名思义，PathMeasure是一个用来测量Path的类，主要有以下方法:

### 构造方法

方法名 | 释义
---|---
PathMeasure() | 创建一个空的PathMeasure
PathMeasure(Path path, boolean forceClosed) | 创建 PathMeasure 并关联一个指定的Path(Path需要已经创建完成)。

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

******


### 1.构造函数

构造函数有两个。

**无参构造函数：**

``` java
  PathMeasure ()
```

用这个构造函数可创建一个空的 PathMeasure，但是使用之前需要先调用 setPath 方法来与 Path 进行关联。被关联的 Path 必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

**有参构造函数：**

``` java
  PathMeasure (Path path, boolean forceClosed)
```

用这个构造函数是创建一个 PathMeasure 并关联一个 Path， 其实和创建一个空的 PathMeasure 后调用 setPath 进行关联效果是一样的，同样，被关联的 Path 也必须是已经创建好的，如果关联之后 Path 内容进行了更改，则需要使用 setPath 方法重新关联。

该方法有两个参数，第一个参数自然就是被关联的 Path 了，第二个参数是用来确保 Path 闭合，如果设置为 true， 则不论之前Path是否闭合，都会自动闭合该 Path(如果Path可以闭合的话)。

**在这里有两点需要明确:**

> 
* 1. 不论 forceClosed 设置为何种状态(true 或者 false)， 都不会影响原有Path的状态，**即 Path 与 PathMeasure  关联之后，之前的的 Path 不会有任何改变。**
* 2. forceClosed 的设置状态可能会影响测量结果，**如果 Path 未闭合但在与 PathMeasure 关联的时候设置 forceClosed 为 true 时，测量结果可能会比 Path 实际长度稍长一点，获取到到是该 Path 闭合时的状态。**

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

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f49allf7gij308c0et3yk.jpg)

我们所创建的 Path 实际上是一个边长为 200 的正方形的三条边，通过上面的示例就能验证以上两个问题。

> 
* 1.我们将 Path 与两个的 PathMeasure 进行关联，并给 forceClosed 设置了不同的状态，之后绘制再绘制出来的 Path 没有任何变化，所以与 Path 与 PathMeasure进行关联并不会影响 Path 状态。
* 2.我们可以看到，设置 forceClosed 为 true 的方法比设置为 false 的方法测量出来的长度要长一点，这是由于 Path 没有闭合的缘故，多出来的距离正是 Path 最后一个点与最开始一个点之间点距离。**forceClosed 为 false 测量的是当前 Path  状态的长度， forceClosed 为 true，则不论Path是否闭合测量的都是 Path 的闭合长度。**





### 2.setPath、 isClosed 和 getLength

这三个方法都如字面意思一样，非常简单，这里就简单是叙述一下，不再过多讲解。

setPath 是 PathMeasure 与 Path 关联的重要方法，效果和 构造函数 中两个参数的作用是一样的。

isClosed 用于判断 Path 是否闭合，但是如果你在关联 Path 的时候设置 forceClosed 为 true 的话，这个方法的返回值则一定为true。

getLength 用于获取 Path 的总长度，在之前的测试中已经用过了。





### 3.getSegment

getSegment 用于获取Path的一个片段，方法如下：

``` java
  boolean getSegment (float startD, float stopD, Path dst, boolean startWithMoveTo)
```

方法各个参数释义：

参数            | 作用                             | 备注
----------------|----------------------------------|--------------------------------------------
返回值(boolean) | 判断截取是否成功                 | true 表示截取成功，结果存入dst中，false 截取失败，不会改变dst中内容
startD          | 开始截取位置距离 Path 起点的长度 | 取值范围: 0 <= startD < stopD <= Path总长度
stopD           | 结束截取位置距离 Path 起点的长度 | 取值范围: 0 <= startD < stopD <= Path总长度
dst             | 截取的 Path 将会添加到 dst 中    | 注意: 是添加，而不是替换
startWithMoveTo | 起始点是否使用 moveTo            | 用于保证截取的 Path 第一个点位置不变

> 
* 如果 startD、stopD 的数值不在取值范围 [0, getLength] 内，或者 startD == stopD 则返回值为 false，不会改变 dst 内容。
* 如果在安卓4.4或者之前的版本，在默认开启硬件加速的情况下，更改 dst 的内容后可能绘制会出现问题，请关闭硬件加速或者给 dst  添加一个单个操作，例如: dst.rLineTo(0, 0)

我们先看看这个方法如何使用：

我们创建了一个 Path， 并在其中添加了一个矩形，现在我们想截取矩形中的一部分，就是下图中红色的部分。

> 矩形边长400dp，起始点在左上角，顺时针

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f4bohcxzhqj308c0etwej.jpg)

代码:

``` java
    canvas.translate(mViewWidth / 2, mViewHeight / 2);          // 平移坐标系

    Path path = new Path();                                     // 创建Path并添加了一个矩形
    path.addRect(-200, -200, 200, 200, Path.Direction.CW);

    Path dst = new Path();                                      // 创建用于存储截取后内容的 Path

    PathMeasure measure = new PathMeasure(path, false);         // 将 Path 与 PathMeasure 关联

    // 截取一部分存入dst中，并使用 moveTo 保持截取得到的 Path 第一个点的位置不变
    measure.getSegment(200, 600, dst, true);                    

    canvas.drawPath(dst, mDeafultPaint);                        // 绘制 dst
```

结果如下：

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f4bpmetj7wj308c0etdfu.jpg)

从上图可以看到我们成功到将需要到片段截取了出来，然而当 dst 中有内容时会怎样呢？

``` java
    canvas.translate(mViewWidth / 2, mViewHeight / 2);          // 平移坐标系

    Path path = new Path();                                     // 创建Path并添加了一个矩形
    path.addRect(-200, -200, 200, 200, Path.Direction.CW);

    Path dst = new Path();                                      // 创建用于存储截取后内容的 Path
    dst.lineTo(-300, -300);                                     // <--- 在 dst 中添加一条线段

    PathMeasure measure = new PathMeasure(path, false);         // 将 Path 与 PathMeasure 关联

    measure.getSegment(200, 600, dst, true);                   // 截取一部分 并使用 moveTo 保持截取得到的 Path 第一个点的位置不变

    canvas.drawPath(dst, mDeafultPaint);                        // 绘制 Path
```

结果如下:

![](http://ww3.sinaimg.cn/large/005Xtdi2gw1f4cg8rl0wmj308c0et74b.jpg)

从上面的示例可以看到 dst 中的线段保留了下来，可以得到结论：**被截取的 Path 片段会添加到 dst 中，而不是替换 dst 中到内容。**

前面两个例子中 startWithMoveTo 均为 true， 如果设置为false会怎样呢?

``` java
    canvas.translate(mViewWidth / 2, mViewHeight / 2);          // 平移坐标系

    Path path = new Path();                                     // 创建Path并添加了一个矩形
    path.addRect(-200, -200, 200, 200, Path.Direction.CW);

    Path dst = new Path();                                      // 创建用于存储截取后内容的 Path
    dst.lineTo(-300, -300);                                     // 在 dst 中添加一条线段

    PathMeasure measure = new PathMeasure(path, false);         // 将 Path 与 PathMeasure 关联

    measure.getSegment(200, 600, dst, false);                   // <--- 截取一部分 不使用 startMoveTo, 保持 dst 的连续性

    canvas.drawPath(dst, mDeafultPaint);                        // 绘制 Path
```

结果如下：

![](http://ww2.sinaimg.cn/large/005Xtdi2gw1f4cgdgc7etj308c0et3yk.jpg)

从该示例我们又可以得到一条结论：**如果 startWithMoveTo 为 true, 则被截取出来到Path片段保持原状，如果 startWithMoveTo 为 false，则会将截取出来的 Path 片段的起始点移动到 dst 的最后一个点，以保证 dst 的连续性。**

从而我们可以用以下规则来判断 startWithMoveTo 的取值：

取值  | 主要功用
------|------------------
true  | 保证截取得到的 Path 片段不会发生形变
false | 保证存储截取片段的 Path(dst) 的连续性





### 4.nextContour

我们知道 Path 可以由多条曲线构成，但不论是 getLength , getgetSegment 或者是其它方法，都只会在其中第一条线段上运行，而这个 `nextContour` 就是用于跳转到下一条曲线到方法，_如果跳转成功，则返回 true， 如果跳转失败，则返回 false。_

如下，我们创建了一个 Path 并使其中包含了两个闭合的曲线，内部的边长是200，外面的边长是400，现在我们使用 PathMeasure 分别测量两条曲线的总长度。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f4ctzjr08dj308c0et74c.jpg)

代码：

``` java
    canvas.translate(mViewWidth / 2, mViewHeight / 2);      // 平移坐标系

    Path path = new Path();

    path.addRect(-100, -100, 100, 100, Path.Direction.CW);  // 添加小矩形
    path.addRect(-200, -200, 200, 200, Path.Direction.CW);  // 添加大矩形

    canvas.drawPath(path,mDeafultPaint);                    // 绘制 Path
    
    PathMeasure measure = new PathMeasure(path, false);     // 将Path与PathMeasure关联

    float len1 = measure.getLength();                       // 获得第一条路径的长度

    measure.nextContour();                                  // 跳转到下一条路径

    float len2 = measure.getLength();                       // 获得第二条路径的长度

    Log.i("LEN","len1="+len1);                              // 输出两条路径的长度
    Log.i("LEN","len2="+len2);
```

log输出结果:
```
05-30 02:00:33.899 19879-19879/com.gcssloop.canvas I/LEN: len1=800.0
05-30 02:00:33.899 19879-19879/com.gcssloop.canvas I/LEN: len2=1600.0
```

通过测试，我们可以得到以下内容：

* 1.曲线的顺序与 Path 中添加的顺序有关。
* 2.getLength 获取到到是当前一条曲线分长度，而不是整个 Path 的长度。
* 3.getLength 等方法是针对当前的曲线(其它方法请自行验证)。





#### 5.getPosTan

这个方法是用于得到路径上某一长度的位置以及该位置的正切值：
``` java
  boolean getPosTan (float distance, float[] pos, float[] tan)
```

方法各个参数释义：

参数            | 作用                             | 备注
----------------|----------------------------------|--------------------------------------------
返回值(boolean) | 判断获取是否成功                 | true表示成功，数据会存入 pos 和 tan 中，<br/>false 表示失败，pos 和 tan 不会改变
distance        | 距离 Path 起点的长度             | 取值范围: 0 <= distance <= getLength
pos             | 该点的坐标值                     | 坐标值: (x==[0], y==[1])
tan             | 该点的正切值                     | 正切值: (x==[0], y==[1])

这个方法也不难理解，除了其中 `tan` 这个东东，这个东西是干什么的呢？

`tan` 是用来判断 Path 的趋势的，即在这个位置上曲线的走向，请看下图示例，注意箭头的方向:

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f4dtufydm4g308c0etmyl.gif)

**[点击这里下载箭头图片](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4gam21ktoj3069069jre.jpg)**

可以看到 上图中箭头在沿着 Path 运动时，方向始终与 Path 走向保持一致，下面我们来看看代码是如何实现的:

首先我们需要定义几个必要的变量:

``` java
    private float currentValue = 0;     // 用于纪录当前的位置,取值范围[0,1]映射Path的整个长度

    private float[] pos;                // 当前点的实际位置
    private float[] tan;                // 当前点的tangent值,用于计算图片所需旋转的角度
    private Bitmap mBitmap;             // 箭头图片
    private Matrix mMatrix;             // 矩阵,用于对图片进行一些操作
```

初始化这些变量(在构造函数中调用这个方法):

``` java
    private void init(Context context) {
        pos = new float[2];
        tan = new float[2];
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inSampleSize = 2;       // 缩放图片
        mBitmap = BitmapFactory.decodeResource(context.getResources(), R.drawable.arrow, options);
        mMatrix = new Matrix();
    }
```

具体绘制:

``` java

    canvas.translate(mViewWidth / 2, mViewHeight / 2);      // 平移坐标系

    Path path = new Path();                                 // 创建 Path

    path.addCircle(0, 0, 200, Path.Direction.CW);           // 添加一个圆形

    PathMeasure measure = new PathMeasure(path, false);     // 创建 PathMeasure

    currentValue += 0.005;                                  // 计算当前的位置在总长度上的比例[0,1]
    if (currentValue >= 1) {
        currentValue = 0;
    }

    measure.getPosTan(measure.getLength() * currentValue, pos, tan);        // 获取当前位置的坐标以及趋势

    mMatrix.reset();                                                        // 重置Matrix
    float degrees = (float) (Math.atan2(tan[1], tan[0]) * 180.0 / Math.PI); // 计算图片旋转角度

    mMatrix.postRotate(degrees, mBitmap.getWidth() / 2, mBitmap.getHeight() / 2);   // 旋转图片
    mMatrix.postTranslate(pos[0] - mBitmap.getWidth() / 2, pos[1] - mBitmap.getHeight() / 2);   // 将图片绘制中心调整到与当前点重合

    canvas.drawPath(path, mDeafultPaint);                                   // 绘制 Path
    canvas.drawBitmap(mBitmap, mMatrix, mDeafultPaint);                     // 绘制箭头

    invalidate();                                                           // 重绘页面
```

**核心要点:**

> 
* 1.**通过 `tan` 得值计算出图片旋转的角度**，tan 是 tangent 的缩写，即中学中常见的正切， 其中tan[0](x)是邻边边长，tan[1](y)是对边边长，而Math中 `atan2` 方法是根据正切是数值计算出该角度的大小,得到的单位是弧度，所以上面又将弧度转为了角度。
* 2.**通过 `Matrix` 来设置图片对旋转角度和位移**，这里使用的方法与前面讲解过对 canvas操作 有些类似，对于 `Matrix` 会在后面专一进行讲解，敬请期待。
* 3.**页面刷新**，页面刷新此处是在 onDraw 里面调用了 invalidate 方法来保持界面不断刷新，但并不提倡这么做，正确对做法应该是使用 线程 或者 ValueAnimator 来控制界面的刷新，关于控制页面刷新这一部分会在后续的 动画部分 详细讲解，同样敬请期待。




### 6.getMatrix

这个方法是用于得到路径上某一长度的位置以及该位置的正切值的矩阵：
``` java
boolean getMatrix (float distance, Matrix matrix, int flags)
```

方法各个参数释义：

参数            | 作用                             | 备注
----------------|----------------------------------|--------------------------------------------
返回值(boolean) | 判断获取是否成功                 | true表示成功，数据会存入matrix中，false 失败，matrix内容不会改变
distance        | 距离 Path 起点的长度             | 取值范围: 0 <= distance <= getLength
matrix          | 根据 falgs 封装好的matrix        | 会根据 flags 的设置而存入不同的内容
flags           | 规定哪些内容会存入到matrix中     | 可选择<br/>POSITION_MATRIX_FLAG(位置) <br/>ANGENT_MATRIX_FLAG(正切)

其实这个方法就相当于我们在前一个例子中封装 `matrix` 的过程由 `getMatrix` 替我们做了，我们可以直接得到一个封装好到 `matrix`，岂不快哉。 

但是我们看到最后到 `flags` 选项可以选择 `位置` 或者 `正切` ,如果我们两个选项都想选择怎么办？

如果两个选项都想选择，可以将两个选项之间用 `|` 连接起来，如下：
```
measure.getMatrix(distance, matrix, PathMeasure.TANGENT_MATRIX_FLAG | PathMeasure.POSITION_MATRIX_FLAG);
```

我们可以将上面都例子中 `getPosTan` 替换为 `getMatrix`， 看看是不是会显得简单很多:

具体绘制: 

``` java
    Path path = new Path();                                 // 创建 Path

    path.addCircle(0, 0, 200, Path.Direction.CW);           // 添加一个圆形

    PathMeasure measure = new PathMeasure(path, false);     // 创建 PathMeasure

    currentValue += 0.005;                                  // 计算当前的位置在总长度上的比例[0,1]
    if (currentValue >= 1) {
        currentValue = 0;
    }

    // 获取当前位置的坐标以及趋势的矩阵
    measure.getMatrix(measure.getLength() * currentValue, mMatrix, PathMeasure.TANGENT_MATRIX_FLAG | PathMeasure.POSITION_MATRIX_FLAG);
    
    mMatrix.preTranslate(-mBitmap.getWidth() / 2, -mBitmap.getHeight() / 2);   // <-- 将图片绘制中心调整到与当前点重合(注意:此处是前乘pre)

    canvas.drawPath(path, mDeafultPaint);                                   // 绘制 Path
    canvas.drawBitmap(mBitmap, mMatrix, mDeafultPaint);                     // 绘制箭头

    invalidate();                                                           // 重绘页面
```

> 由于此处代码运行结果与上面一样，便不再贴图片了，请参照上面一个示例的效果图。

可以看到使用 getMatrix 方法的确可以节省一些代码，不过这里依旧需要注意一些内容:

>
* 1.对 `matrix` 的操作必须要在 `getMatrix` 之后进行，否则会被 `getMatrix` 重置而导致无效。
* 2.矩阵对旋转角度默认为图片的左上角，我们此处需要使用 `preTranslate` 调整为图片中心。
* 3.pre(矩阵前乘) 与 post(矩阵后乘) 的区别，此处请等待后续的文章或者自行搜索。

*****


## Path & SVG

我们知道，用Path可以创建出各种个样的图形，但如果图形过于复杂时，用代码写就不现实了，不仅麻烦，而且容易出错，所以在绘制复杂的图形时我们一般是将 SVG 图像转换为 Path。

**你说什么是 SVG?**
 
SVG 是一种矢量图，内部用的是 xml 格式化存储方式存储这操作和数据，你完全可以将 SVG 看作是 Path 的各项操作简化书写后的存储格式。 
 
Path 和 SVG 结合通常能诞生出一些奇妙的东西，如下:

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f4g87vfjbeg30690b4go8.gif)
![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f4g89vqhqwg30690b4mzu.gif)

>
**该图片来自这个开源库 ->[PathView](https://github.com/geftimov/android-pathview)** <br/>
**SVG 转 Path 的解析可以用这个库 -> [AndroidSVG](https://bigbadaboom.github.io/androidsvg/)**

限于篇幅以及本人精力，这一部分就暂不详解了，感兴趣的可以直接看源码，或者搜索一些相关的解析文章。

*****

## Path使用技巧

**话说本篇文章的名字不是叫 玩出花样么？怎么只见前面啰啰嗦嗦的扯了一大堆不明所以的东西，花样在哪里？**

>
**前面的内容虽然啰嗦繁杂，但却是重中之重的基础，如果在修仙界，这叫根基，而下面讲述的内容的是招式，有了根基才能演化出千变万化的招式，而没有根基只学招式则是徒有其表，只能学一样会一样，很难适应千变万化的需求。**

先放一个效果图，然后分析一下实现过程:

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f4fp2myqo4g308c05k75k.gif)

这是一个搜索的动效图，通过分析可以得到它应该有四种状态，分别如下:

状态     |概述
---------|--------------------------------------------------
初始状态 | 初始状态，没有任何动效，只显示一个搜索标志 :mag:
准备搜索 | 放大镜图标逐渐变化为一个点
正在搜索 | 围绕这一个圆环运动，并且线段长度会周期性变化
准备结束 | 从一个点逐渐变化成为放大镜图标

这些状态是有序转换的，转换流程以及转换条件如下：

> 其中 `正在搜索` 这个状态持续时间长度是不确定的，在没有搜索完成前，应该一直处于搜索状态。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f4g4i3c06cj30a60gb3yt.jpg)

简单的分析了其大致的流程之后，就到了制作的重点:对细节对把握。

### Path 划分

为了制作对方便，此处整个动效用了两个 Path， 一个是中间对放大镜， 另一个则是外侧的圆环,将两者全部画出来是这样子的。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f4gbhxuuktj308c0etwej.jpg) 

其中 Path 的走向要把握好，如下(只是一个放大镜，并不是♂):

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4gbjj3fd1j308c05kt8i.jpg)

其中圆形上面的点可以用 PathMeasure 测量，无需计算。

### 动画状态与时间关联

此处使用的是 ValueAnimator，它可以将一段时间映射到一段数值上，随着时间变化不断的更新数值，并且可以使用插值器开控制数值变化规律(此处使用的是默认插值器)。

> PS: 本来不想提前暴露这个的，准备偷偷留到动画部分(｡-_-｡) 但实在是没有优雅的替代方案了。

### 具体绘制

绘制部分是根据 当前状态以及从 ValueAnimator 获得的数值来截取 Path 中合适的部分绘制出来。

### 最终效果

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f4gbmtj0mvg308c0etq58.gif)

### 源码

上面的内容是为了帮助大家从把控全局流程以及理解某些细节的设计思路，而更多的内容都藏在代码中，代码总体也不算长，感兴趣的可以自己敲一遍。

#### [戳这里查看源码](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/Code/SearchView.md)

> PS: 本代码仅作为示例使用，还有诸多不足，如 自定义属性，视图大小， 点击事件， 监听回调 等，并不适合直接使用，有需要的可以自行补足相关内容。


## 总结

**本文中虽然后面的内容看起来比较高大上一点，但前面"啰嗦"的废话才是真正的干货，把前面的东西学会了，后面的各种效果都能信手拈来，如果只研究后面的东西，则是取其形，而难以会其意。**

#### PS: 由于本人水平有限，某些地方可能存在误解或不准确，如果你对此有疑问可以提交Issues进行反馈。

## About Me

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300 height=100 /> </a>

## 参考资料
[PathMeasure](https://developer.android.com/reference/android/graphics/PathMeasure.html)<br/>
[AndroidSVG](https://bigbadaboom.github.io/androidsvg/)<br/>
[android-pathview](https://github.com/geftimov/android-pathview)<br/>
[android Path 和 PathMeasure 进阶](http://blog.csdn.net/cquwentao/article/details/51436852)<br/>
[]()<br/>






