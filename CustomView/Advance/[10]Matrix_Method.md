# Matrix方法

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### 相关文章: [自定义View目录](http://www.gcssloop.com/1970/01/CustomViewIndex/)

在上一篇文章中，我们对Matrix做了一个简单的了解，偏向理论，在本文中则会详细的讲解Matrix的具体用法，以及Matrix的一些实用技巧。

## Matrix方法表

按照惯例，先放方法表做概览。

|   方法类别   | 相关API                                    | 摘要                         |
| :------: | ---------------------------------------- | :------------------------- |
| 设置(set)  | setConcat setRotate setScale setSkew setTranslate | 设置变换                       |
| 前乘(pre)  | preConcat preRotate preScale preSkew preTranslate | 前乘变换                       |
| 后乘(post) | postConcat postRotate postScale postSkew postTranslate | 后乘变换                       |
|   特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos | 一些特殊操作                     |
|   矩阵相关   | invert isAffine isIdentity               | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ... |


## Matrix方法详解

### 构造方法

构造方法没有在上面表格中列出。

**无参构造**

``` java
Matrix ()
```
创建一个全新的Matrix，使用格式如下：

``` java
Matrix matrix = new Matrix();
```

通过这种方式创建出来的并不是一个数值全部为空的矩阵，而是一个单位矩阵,如下:

![](http://latex.codecogs.com/png.latex?$$
\\left [ 
\\begin{matrix} 
1 & 0 & 0 \\\\
0 & 1 & 0 \\\\
0 & 0 & 1 
\\end{1} 
\\right ]
$$)


**有参构造**

``` java
Matrix (Matrix src)
```

这种方法则需要一个已经存在的矩阵作为参数，使用格式如下:

``` java
Matrix matrix = new Matrix(src);
```

创建一个Matrix，并对src深拷贝(理解为新的matrix和src是两个对象，但内部数值相同即可)。


### 基本方法

基本方法内容比较简单，在此处简要介绍一下。

**1.equals**

比较两个Matrix的数值是否相同。

**2.hashCode**

获取Matrix的哈希值。

**3.toString**

将Matrix转换为字符串: `Matrix{[1.0, 0.0, 0.0][0.0, 1.0, 0.0][0.0, 0.0, 1.0]}`

**4.toShortString**

将Matrix转换为短字符串: `[1.0, 0.0, 0.0][0.0, 1.0, 0.0][0.0, 0.0, 1.0]`


### 数值操作

数值操作这一组方法可以帮助我们直接控制Matrix里面的数值。

**1.set**

``` java
void set (Matrix src)
```

没有返回值，有一个参数，作用是将参数Matrix的数值复制到当前Matrix中。如果参数为空，则重置当前Matrix，相当于`reset()`。

**2.reset**

``` java
void reset ()
```

重置当前Matrix(将当前Matrix重置为单位矩阵)。

**3.setValues**

``` java
void setValues (float[] values)
```

setValues的参数是浮点型的一维数组，长度需要大于9，拷贝数组中的前9位数值赋值给当前Matrix。

**4.getValues**

``` java
void getValues (float[] values)
```

很显然，getValues和setValues是一对方法，参数也是浮点型的一维数组，长度需要大于9，将Matrix中的数值拷贝进参数的前9位中。

### 数值计算

**1.mapPoints**

``` java
void mapPoints (float[] pts)

void mapPoints (float[] dst, float[] src)

void mapPoints (float[] dst, int dstIndex,float[] src, int srcIndex, int pointCount)
```

计算一组点基于当前Matrix变换后的位置，(由于是计算点，所以参数中的float数组长度一般都是偶数的,若为奇数，则最后一个数值不参与计算)。

它有三个重载方法:

(1) `void mapPoints (float[] pts)` 方法仅有一个参数，pts数组作为参数传递原始数值，计算结果仍存放在pts中。

示例:

``` java
// 初始数据为三个点 (0, 0) (80, 100) (400, 300) 
float[] pts = new float[]{0, 0, 80, 100, 400, 300};

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

// 输出pts计算之前数据
Log.i(TAG, "before: "+ Arrays.toString(pts));

// 调用map方法计算
matrix.mapPoints(pts);

// 输出pts计算之后数据
Log.i(TAG, "after : "+ Arrays.toString(pts));
```

结果:

```
before: [0.0, 0.0, 80.0, 100.0, 400.0, 300.0]
after : [0.0, 0.0, 40.0, 100.0, 200.0, 300.0]
```

(2) `void mapPoints (float[] dst, float[] src)` ，src作为参数传递原始数值，计算结果存放在dst中，src不变。

如果原始数据需要保留则一般使用这种方法。

示例:

``` java
// 初始数据为三个点 (0, 0) (80, 100) (400, 300)
float[] src = new float[]{0, 0, 80, 100, 400, 300};
float[] dst = new float[6];

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

// 输出计算之前数据
Log.i(TAG, "before: src="+ Arrays.toString(src));
Log.i(TAG, "before: dst="+ Arrays.toString(dst));

// 调用map方法计算
matrix.mapPoints(dst,src);

// 输出计算之后数据
Log.i(TAG, "after : src="+ Arrays.toString(src));
Log.i(TAG, "after : dst="+ Arrays.toString(dst));
```

结果:

```
before: src=[0.0, 0.0, 80.0, 100.0, 400.0, 300.0]
before: dst=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
after : src=[0.0, 0.0, 80.0, 100.0, 400.0, 300.0]
after : dst=[0.0, 0.0, 40.0, 100.0, 200.0, 300.0]
```

(3) `void mapPoints (float[] dst, int dstIndex,float[] src, int srcIndex, int pointCount)` 可以指定只计算一部分数值。

| 参数         | 摘要           |
| ---------- | ------------ |
| dst        | 目标数据         |
| dstIndex   | 目标数据存储位置起始下标 |
| src        | 源数据          |
| srcIndex   | 源数据存储位置起始下标  |
| pointCount | 计算的点个数       |


示例:

>
>将第二、三个点计算后存储进dst最开始位置。

``` java
// 初始数据为三个点 (0, 0) (80, 100) (400, 300)
float[] src = new float[]{0, 0, 80, 100, 400, 300};
float[] dst = new float[6];

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

// 输出计算之前数据
Log.i(TAG, "before: src="+ Arrays.toString(src));
Log.i(TAG, "before: dst="+ Arrays.toString(dst));

// 调用map方法计算(最后一个2表示两个点，即四个数值,并非两个数值)
matrix.mapPoints(dst, 0, src, 2, 2);

// 输出计算之后数据
Log.i(TAG, "after : src="+ Arrays.toString(src));
Log.i(TAG, "after : dst="+ Arrays.toString(dst));
```

结果:

```
before: src=[0.0, 0.0, 80.0, 100.0, 400.0, 300.0]
before: dst=[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]
after : src=[0.0, 0.0, 80.0, 100.0, 400.0, 300.0]
after : dst=[40.0, 100.0, 200.0, 300.0, 0.0, 0.0]
```

**2.mapRadius**

``` java
float mapRadius (float radius)
```

测量半径，由于圆可能会因为画布变换变成椭圆，所以此处测量的是平均半径。

示例:

``` java
float radius = 100;
float result = 0;

// 构造一个matrix，x坐标缩放0.5
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);

Log.i(TAG, "mapRadius: "+radius);

result = matrix.mapRadius(radius);

Log.i(TAG, "mapRadius: "+result);
```

结果:

```
mapRadius: 100.0
mapRadius: 70.71068
```


**3.mapRect**

```
boolean mapRect (RectF rect)

boolean mapRect (RectF dst, RectF src)
```

测量矩形变换后位置。

(1) `boolean mapRect (RectF rect)` 测量rect并将测量结果放入rect中，返回值是判断矩形经过变换后是否仍为矩形。

示例：

``` java
RectF rect = new RectF(400, 400, 1000, 800);

// 构造一个matrix
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);
matrix.postSkew(1,0);

Log.i(TAG, "mapRadius: "+rect.toString());

boolean result = matrix.mapRect(rect);

Log.i(TAG, "mapRadius: "+rect.toString());
Log.e(TAG, "isRect: "+ result);
```

结果：

```
mapRadius: RectF(400.0, 400.0, 1000.0, 800.0)
mapRadius: RectF(600.0, 400.0, 1300.0, 800.0)
isRect: false
```

>
>由于使用了错切，所以返回结果为false。

(2) `boolean mapRect (RectF dst, RectF src)` 测量src并将测量结果放入dst中，返回值是判断矩形经过变换后是否仍为矩形,和之前没有什么太大区别，此处就不啰嗦了。

**4.mapVectors**

测量向量。

``` java
void mapVectors (float[] vecs)

void mapVectors (float[] dst, float[] src)

void mapVectors (float[] dst, int dstIndex, float[] src, int srcIndex, int vectorCount)
```

`mapVectors` 与 `mapPoints` 基本上是相同的，可以直接参照上面的`mapPoints`使用方法。

而两者唯一的区别就是`mapVectors`不会受到位移的影响，这符合向量的定律，如果你不了解的话，请找到以前教过你的老师然后把学费要回来。

区别:

``` java
float[] src = new float[]{1000, 800};
float[] dst = new float[2];

// 构造一个matrix
Matrix matrix = new Matrix();
matrix.setScale(0.5f, 1f);
matrix.postTranslate(100,100);

// 计算向量, 不受位移影响
matrix.mapVectors(dst, src);
Log.i(TAG, "mapVectors: "+Arrays.toString(dst));

// 计算点
matrix.mapPoints(dst, src);
Log.i(TAG, "mapPoints: "+Arrays.toString(dst));
```

结果:

```
mapVectors: [500.0, 800.0]
mapPoints: [600.0, 900.0]
```

### set、pre 与 post

对于四种基本变换 平移(translate)、缩放(scale)、旋转(rotate)、 错切(skew) 它们每一种都三种操作方法，分别为 设置(set)、 前乘(pre) 和 后乘 (post)。

**关于四种基本变换的知识和三种对应操作的区别，详细可以参考 [Canvas之画布操作](http://www.gcssloop.com/2015/02/Canvas_Convert/) 和 [Matrix原理](http://www.gcssloop.com/2015/02/Matrix_Basic/) 这两篇文章的内容。**

由于之前的文章已经详细的讲解过了它们的原理与用法，所以此处就简要的介绍一下:

| 方法   | 简介                                   |
| ---- | ------------------------------------ |
| set  | 设置，会覆盖掉之前的数值，导致之前的操作失效。              |
| pre  | 前乘，相当于矩阵的右乘， `M' = M * S`  (S指为特殊矩阵) |
| post | 后乘，相当于矩阵的左乘，`M' = S * M` （S指为特殊矩阵）   |

**Matrix 相关的重要知识：**

- 1.一开始从Canvas中获取到到Matrix并不是初始矩阵，而是经过偏移后到矩阵，且偏移距离就是距离屏幕左上角的位置。

- > 这个可以用于判定View在屏幕上的绝对位置，View可以根据所处位置做出调整。

- 2.构造Matrix时使用的是矩阵乘法，前乘(pre)与后乘(post)结果差别很大。

- > 这个直接参见上一篇文章 [Matrix原理](http://www.gcssloop.com/2015/02/Matrix_Basic/) 即可。

- 3.受矩阵乘法影响，后面的执行的操作可能会影响到之前的操作。

- > 使用时需要注意构造顺序。


### 特殊方法

这一类方法看似不起眼，但拿来稍微加工一下就可能制作意想不到的效果。

**1.setPolyToPoly**

```java
boolean setPolyToPoly (
        float[] src, 
        int srcIndex, 
        float[] dst, 
        int dstIndex, 
        int pointCount)
```







### 矩阵相关

## Matrix实用技巧

## About

## 参考资料
