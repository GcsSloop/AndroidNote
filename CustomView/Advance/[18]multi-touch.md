# Android 多点触控详解

Android 多点触控详解，在前面的几篇文章中我们大致了解了 Android 中的事件处理流程和一些简单的处理方案，本次带大家了解 Android 多点触控相关的一些知识。  

**多点触控** ( **Multitouch**，也称 **Multi-touch** )，即同时接受屏幕上多个点的人机交互操作，多点触控是从 Android 2.0 开始引入的功能，在 Android 2.2 时对这一部分进行了重新设计。

在本文开始之前，先回顾一下 [MotionEvent详解][motionevent] 中提到过的内容：

- Android 将所有的事件都封装进了 `Motionvent` 中。
- 我们可以通过复写 `onTouchEvent` 或者设置 `OnTouchListener` 来获取 View 的事件。
- 多点触控获取事件类型请使用 `getActionMasked()` 。
- 追踪事件流请使用 `PointId`。

**多点触控相关的事件：**

| 事件                          | 简介                             |
| --------------------------- | ------------------------------ |
| ACTION_DOWN                 | **第一个** 手指 **初次接触到屏幕** 时触发。    |
| ACTION_MOVE                 | 手指 **在屏幕上滑动** 时触发，会多次触发。       |
| ACTION_UP                   | **最后一个** 手指 **离开屏幕** 时触发。      |
| **ACTION_POINTER_DOWN**     | 有非主要的手指按下(**即按下之前已经有手指在屏幕上**)。 |
| **ACTION_POINTER_UP**       | 有非主要的手指抬起(**即抬起之后仍然有手指在屏幕上**)。 |
| 以下事件类型不推荐使用                 | －－－以下事件在 2.2 版本以上被标记为废弃－－－     |
| ~~ACTION_POINTER\_1\_DOWN~~ | 第 2 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_2\_DOWN~~ | 第 3 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_3\_DOWN~~ | 第 4 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_1\_UP~~   | 第 2 个手指抬起，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_2\_UP~~   | 第 3 个手指抬起，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_3\_UP~~   | 第 4 个手指抬起，已废弃，不推荐使用。           |

**多点触控相关的方法：**

| 方法                              | 简介                                       |
| ------------------------------- | ---------------------------------------- |
| getActionMasked()               | 与 `getAction()` 类似，**多点触控需要使用这个方法获取事件类型**。 |
| getActionIndex()                | 获取该事件是哪个指针(手指)产生的。                       |
| getPointerCount()               | 获取在屏幕上手指的个数。                             |
| getPointerId(int pointerIndex)  | 获取一个指针(手指)的唯一标识符ID，在手指按下和抬起之间ID始终不变。     |
| findPointerIndex(int pointerId) | 通过PointerId获取到当前状态下PointIndex，之后通过PointIndex获取其他内容。 |
| getX(int pointerIndex)          | 获取某一个指针(手指)的X坐标                          |
| getY(int pointerIndex)          | 获取某一个指针(手指)的Y坐标                          |

回顾完毕，开始正文。



## 一、多点触控相关问题

在引入多点触控之前，事件的类型很少，基本事件类型只有按下(down)、移动(move) 和 抬起(up)，即便加上那些特殊的事件类型也只有几种而已，所以我们可以用几个常量来标记这些事件，在使用的时候使用 `getAction()` 方法来获取具体的事件，之后和这些常量进行对比就行了。

在 Android 2.0 版本的时候，开始引入多点触控技术，由于技术上并不成熟，硬件和驱动也跟不上，多数设备只能支持追踪两三个点而已，因此在设计 API 上采取了一种简单粗暴的方案，添加了几个常量用于多点触控的事件类型的判断。

| 事件                      | 简介                   |
| ----------------------- | -------------------- |
| ACTION_POINTER\_1\_DOWN | 第 2 个手指按下，已废弃，不推荐使用。 |
| ACTION_POINTER\_2\_DOWN | 第 3 个手指按下，已废弃，不推荐使用。 |
| ACTION_POINTER\_3\_DOWN | 第 4 个手指按下，已废弃，不推荐使用。 |
| ACTION_POINTER\_1\_UP   | 第 2 个手指抬起，已废弃，不推荐使用。 |
| ACTION_POINTER\_2\_UP   | 第 3 个手指抬起，已废弃，不推荐使用。 |
| ACTION_POINTER\_3\_UP   | 第 4 个手指抬起，已废弃，不推荐使用。 |

这些事件类型是用来判断非主要手指(第一个按下的称为主要手指)的按下和抬起，使用起来大概是这样子：

```java
switch (event.getAction()) {
    case MotionEvent.ACTION_DOWN:           break;
    case MotionEvent.ACTION_UP:             break;
    case MotionEvent.ACTION_MOVE:           break;
    case MotionEvent.ACTION_POINTER_1_DOWN: break;
    case MotionEvent.ACTION_POINTER_2_DOWN: break;
    case MotionEvent.ACTION_POINTER_3_DOWN: break;
    case MotionEvent.ACTION_POINTER_1_UP:   break;
    case MotionEvent.ACTION_POINTER_2_UP:   break;
    case MotionEvent.ACTION_POINTER_3_UP:   break;
}
```

看到这里可能会产生以下的一些疑问？

### 1.为什么没有 ACTION_POINTER_X_MOVE ?

在多指触控中所有的移动事件都是使用 `ACTION_MOVE`， 并没有追踪某一个手指的 move 事件类型，个人猜测主要是因为：**很难无歧义的实现单独追踪每一个手指。**

要理解这个，首先要明白设备是如何识别多点触控的，设备没有眼睛，不能像我们人一样看到有几个手指(或者触控笔)在屏幕上。  
目前大多数 Android 设备都是电容屏，它们感知触摸是利用手指(触控笔)与屏幕接触产生的微小电流变化，之后通过计算这些电流变化来得出具体的触摸位置，在多点触控中，当两个触摸点足够靠近时，设备实际上是无法分清这两个点的。因此当两个触摸点靠近(重合)后再分开，设备很可能就无法正确的追踪两个点了，所以也很难实现无歧义的追踪每一个点。

并且从软件上来说，事件的编号产生和复用也是一个大问题，例如下面的场景：

| 事件           | 手指数量 | 编号变化                      |
| ------------ | :--: | ------------------------- |
| 一个手指按下(命名为A) |  1   | A手指的编号为0，id为0             |
| 一个手指按下(命名为B) |  2   | B手指的编号为1，id为1             |
| A手指抬起        |  1   | B手指编号变更为0，id不变为1          |
| 一个手指按下(命名为C) |  2   | C手指编号为0，id为0，B手指编号为1，id为1 |

注意观察上面编号和id的变化，有两个问题，**1、B手指的编号变化了。2、A手指和C手指id是相同的(A手指抬起后，C手指按下替代了A手指)。**所以这就引出了一个问题：如果存在 ACTION_POINTER_X_MOVE，那么X应该用什么标志呢？编号会变化，id虽然不会变化，但id会被复用，例如A手指抬起后C手指按下，C手指复用了A手指的id。所以不论使用哪一个都不能保证唯一性。

当然了，解决问题最好的方式就是把问题抛出去，既然从硬件和软件上都不能保证唯一性和不变性，就不做区分了，因此所有的 move 事件都是 `ACTION_MOVE`, 具体是哪个手指产生的 move 用户可以结合其他事件(按下和抬起)来综合判断。

### 2.超过4个手指怎么办？

**2.0 兼容版**，在2.2 之前的设计中，其提供的常量最多能判断四个手指的抬起和落下，当超过四个手指时怎么办呢？

由于在 2.2 版本之前，由于没有 `getActionMasked` 方法，我们可以自己自己手动进行计算，例如下面这样 ：

```java
String TAG = "Gcs";

int action = event.getAction() & MotionEvent.ACTION_MASK;
int index = (event.getAction() & MotionEvent.ACTION_POINTER_INDEX_MASK)
        >> MotionEvent.ACTION_POINTER_INDEX_SHIFT;

switch (action) {
    case MotionEvent.ACTION_DOWN:
        Log.e(TAG,"第1个手指按下");
        break;
    case MotionEvent.ACTION_UP:
        Log.e(TAG,"最后1个手指抬起");
        break;
    case MotionEvent.ACTION_POINTER_1_DOWN: // 此时相当于 ACTION_POINTER_DOWN
        Log.e(TAG,"第"+(index+1)+"个手指按下");
        break;
    case MotionEvent.ACTION_POINTER_1_UP:   // 此时相当于 ACTION_POINTER_UP
        Log.e(TAG,"第"+(index+1)+"个手指抬起");
        break;
}
```

在上面的例子中有几点比较关键：

#### 2.1、action 与 Index 的获得

我们在 [MotionEvent详解][motionevent] 中了解过，Android中的事件一般用最后8位来表示事件类型，再往前8位来表示Index。

例如多指触控的按下事件，其事件类型是 0x000000**05**， 其Index标志位是 0x0000**00**05，随着更多的手指按下，其中变化的部分是 Index 标志位，最后两位是始终不变的，所以我们只要能将这两个分离开就行了。

**取得事件类型(action)**

```java
// 获取事件类型
int action = event.getAction() & MotionEvent.ACTION_MASK;
```

这个非常简单，ACTION_MASK=0x000000ff， 与 getAction() 进行按位与操作后保留最后8位内容(十六进制每一个字符转化为二进制是4位)。 

例如：  
0x000001**05** & 0x000000ff = 0x000000**05**

**取得事件索引(index)**

```java
// 获取index编号
int index = (event.getAction() & MotionEvent.ACTION_POINTER_INDEX_MASK)
        >> MotionEvent.ACTION_POINTER_INDEX_SHIFT;
```

ACTION_POINTER_INDEX_MASK = 0x0000ff00  
ACTION_POINTER_INDEX_SHIFT = 8  
首先让 getAction() 与 ACTION_POINTER_INDEX_MASK 按位与之后，只保留 Index 那8位，之后再右移8位，最终就拿到了 Index 的真实数值。

例如：   
0x0000**01**05 & 0x0000ff00 = 0x0000**01**00    
0x0000**01**00 >> 8 = 0x000000**01**

#### 2.2、用 ACTION\_POINTER\_1\_DOWN 代替 ACTION\_POINTER\_DOWN

这是因为在 2.0 版本的时候还没有 ACTION\_POINTER\_DOWN 的这个常量，但是它们两个点数值是相同的，都是 0x00000005，这个你可以查看官方文档或者源码，甚至你直接写 `case 0x00000005` 也行，抬起也是同理。

#### 2.3、只考虑兼容 2.2 以上的版本

当然了，如果你不需要兼容 2.0 版本，只需要兼容到 2.2 以上的话就很简单了，像下面这样：

```java
String TAG = "Gcs";

int index = event.getActionIndex();

switch (event.getActionMasked()) {
    case MotionEvent.ACTION_DOWN:
        Log.e(TAG,"第1个手指按下");
        break;
    case MotionEvent.ACTION_UP:
        Log.e(TAG,"最后1个手指抬起");
        break;
    case MotionEvent.ACTION_POINTER_DOWN:
        Log.e(TAG,"第"+(index+1)+"个手指按下");
        break;
    case MotionEvent.ACTION_POINTER_UP:
        Log.e(TAG,"第"+(index+1)+"个手指抬起");
        break;
}
```

### 3. index 和 pointId 的变化规则

在 2.2 版本以上，我们可以通过 getActionIndex() 轻松获取到事件的索引(Index)，但是这个事件索引的变化还是有点意思的，Index 变化有以下几个特点：

1、从 0 开始，自动增长。  
2、如果之前落下的手指抬起，后面手指的 Index 会随之减小。  
3、Index 变化趋向于第一次落下的数值(落下手指时，前面有空缺会优先填补空缺)。  
4、对 move 事件无效。

下面我们逐条解释一下具体含义。

#### 3.1、从 0 开始，自动增长。

这一条非常简单，也很容易理解，而且在 [MotionEvent详解][motionevent] 中讲解 getAction() 与 getActionMasked() 也简单说过。

|  手指按下   | 触发事件(数值)                                 |
| :-----: | :--------------------------------------- |
| 第1个手指按下 | ACTION_DOWN                  (0x0000**00**00) |
| 第2个手指按下 | ACTION_POINTER_DOWN (0x0000**01**05)     |
| 第3个手指按下 | ACTION_POINTER_DOWN (0x0000**02**05)     |
| 第4个手指按下 | ACTION_POINTER_DOWN (0x0000**03**05)     |

注意加粗的位置，数值随着手指按下而不断变大。

#### 3.2、如果之前落下的手指抬起，后面手指的 Index 会随之减小。

这个也比较容易理解，像下面这样：

|  手指按下   | 触发事件(数值)                                 |
| :-----: | :--------------------------------------- |
| 第1个手指按下 | ACTION_DOWN                  (0x0000**00**00) |
| 第2个手指按下 | ACTION_POINTER_DOWN (0x0000**01**05)     |
| 第3个手指按下 | ACTION_POINTER_DOWN (0x0000**02**05)     |
| 第2个手指抬起 | ACTION_POINTER_UP        (0x0000**01**06) |
| 第3个手指抬起 | ACTION_POINTER_UP        (0x0000**01**06) |

注意最后两次触发的事件，它的 Index 都是 1，这样也比较容易解释，当原本的第 2 个手指抬起后，屏幕上就只剩下两个手指了，之前的第 3 个手指就变成了第 2 个，于是抬起时触发事件的 Index 为 1，即之前落下的手指抬起，后面手指的 Index 会随之减小。

#### 3.3、Index 变化趋向于第一次落下的数值(落下手指时，前面有空缺会优先填补空缺)。

这个就有点神奇了，通过上一条规则，我们知道，某一个手指的 Index 可能会随着其他手指的抬起而变小，这次我们用 4 个手指测试一下 Index 的变化趋势。

|    手指按下     | 触发事件(数值)                                 |
| :---------: | :--------------------------------------- |
|   第1个手指按下   | ACTION_DOWN                  (0x0000**00**00) |
|   第2个手指按下   | ACTION_POINTER_DOWN (0x0000**01**05)     |
| **第3个手指按下** | ACTION_POINTER_DOWN (0x0000**02**05)     |
|   第2个手指抬起   | ACTION_POINTER_UP        (0x0000**01**06) |
| ~~第3个手指抬起~~ | ~~ACTION_POINTER_UP~~        ~~(0x0000**01**06)~~ |
|   第4个手指按下   | ACTION_POINTER_DOWN (0x0000**01**05)     |
| **第3个手指抬起** | ACTION_POINTER_UP        (0x0000**02**06) |

这个要和上一个对比这看，**重点观察第 3 个手指所触发事件区别**，在上一个示例中，随着第 2 个手指的抬起，第 3 个手指变化为第 2(01) 个，所以抬起时触发的是第 2 根手指的抬起事件(删除线部分)。

但是，如果第 2 个手指抬起后，落在屏幕上另外一个手指会怎样？经过测试，发现另外**落下的手指会替代之前第 2 个手指的位置，系统判定为 2(01)，而不是顺延下去变成 3(02)，并且原本第3个手指的index变为原来数值(02)**，但是如果继续落下其他的手指，数值则会顺延。

**即手指抬起时的 Index 会趋向于和按下时相同，虽然在手指数量不足时，Index 会变小，但是当手指变多时，Index 会趋向于保持和按下时一样。**

> PS：由于程序是从0开始计数的，所以 0 就是 1， 1 就是 2 ...

#### 3.4、对 move 事件无效。

这个也比较容易理解，我们所取得的 Index 属性实际上是从事件上分离下来的，但是 move 事件始终为  0x0000**00**02，也就是说，在 move 时不论你移动哪个手指，使用 `getActionIndex()`  获取到的始终是数值 0。

既然 move 事件无法用事件索引(Index)区别，那么该如何区分 move 是那个手指发出的呢？这就要用到 pointId 了，**pointId 和 index 最大的区别就是 pointId 是不变的，始终为第一次落下时生成的数值，不会受到其他手指抬起和落下的影响。**

#### 3.5、pointId 与 index 的异同。

相同点：

- 从 0 开始，自动增长。
- 落下手指时优先填补空缺(填补之前抬起手指的编号)。

不同点：

- Index 会变化，pointId 始终不变。

### 4. Move 相关事件

#### 4.1 actionIndex 与 pointerIndex

在 move 中无法取得 actionIndex 的，我们需要使用 pointerIndex 来获取更多的信息，例如某个手指的坐标：

```java
getX(int pointerIndex)
getY(int pointerIndex)
```

**但是这个 pointerIndex 又是什么呢？和 actionIndex 有区别么？**

实际上这个 pointerIndex 和 actionIndex 区别并不大，两者的数值是相同的，你可以认为 pointerIndex 是特地为 move 事件准备的 actionIndex。

#### 4.2 pointerIndex 与 pointerId

| 类型           | 简介                            |
| ------------ | ----------------------------- |
| pointerIndex | 用于获取具体事件，可能会随着其他手指的抬起和落下而变化   |
| pointerId    | 用于识别手指，手指按下时产生，手指抬起时回收，期间始终不变 |

这两个数值使用以下两个方法相互转换。

| 方法                              | 简介                                       |
| ------------------------------- | ---------------------------------------- |
| getPointerId(int pointerIndex)  | 获取一个指针(手指)的唯一标识符ID，在手指按下和抬起之间ID始终不变。     |
| findPointerIndex(int pointerId) | 通过 pointerId 获取到当前状态下 pointIndex，之后通过 pointIndex 获取其他内容。 |

> 通常情况下，pointerIndex 和 pointerId 是相同的，但也可能会因为某些手指的抬起而变得不同。

#### 4.3 遍历多点触控

先来一个简单的，遍历出多个手指的 move 事件：

```java
String TAG = "Gcs";
switch (event.getActionMasked()) {
    case MotionEvent.ACTION_MOVE:
        for (int i = 0; i < event.getPointerCount(); i++) {
            Log.i("TAG", "pointerIndex="+i+", pointerId="+event.getPointerId(i));
          	// TODO
        }
}
```

通过遍历 pointerCount 获取到所有的 pointerIndex，同时通过 pointerIndex 来获取 pointerId，可以通过不同手指抬起和按下后移动来观察 pointerIndex 和 pointerId 的变化。

#### 4.4 在多点触控中追踪单个手指

要实现追踪单个手指还是有些麻烦的，需要同时使用上 actionIndex， pointerId 和 pointerIndex，例如，我们只追踪第2个手指，并画出其位置：

```java
/**
 * 绘制出第二个手指第位置
 */
public class MultiTouchTest extends CustomView {
    String TAG = "Gcs";

    // 用于判断第2个手指是否存在
    boolean haveSecondPoint = false;

    // 记录第2个手指第位置
    PointF point = new PointF(0, 0);

    public MultiTouchTest(Context context) {
        this(context, null);
    }

    public MultiTouchTest(Context context, AttributeSet attrs) {
        super(context, attrs);

        mDeafultPaint.setAntiAlias(true);
        mDeafultPaint.setTextAlign(Paint.Align.CENTER);
        mDeafultPaint.setTextSize(30);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int index = event.getActionIndex();

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_POINTER_DOWN:
                // 判断是否是第2个手指按下
                if (event.getPointerId(index) == 1){
                    haveSecondPoint = true;
                    point.set(event.getY(), event.getX());
                }
                break;
            case MotionEvent.ACTION_POINTER_UP:
                // 判断抬起的手指是否是第2个
                if (event.getPointerId(index) == 1){
                    haveSecondPoint = false;
                    point.set(0, 0);
                }
                break;
            case MotionEvent.ACTION_MOVE:
                if (haveSecondPoint) {
                    // 通过 pointerId 来获取 pointerIndex
                    int pointerIndex = event.findPointerIndex(1);
                    // 通过 pointerIndex 来取出对应的坐标
                    point.set(event.getX(pointerIndex), event.getY(pointerIndex));
                }
                break;
        }

        invalidate();   // 刷新

        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.save();
        canvas.translate(mViewWidth/2, mViewHeight/2);
        canvas.drawText("追踪第2个按下手指的位置", 0, 0, mDeafultPaint);
        canvas.restore();

        // 如果屏幕上有第2个手指则绘制出来其位置
        if (haveSecondPoint) {
            canvas.drawCircle(point.x, point.y, 50, mDeafultPaint);
        }
    }
}
```

这段代码也非常短，其核心就是通过判断数值为 1 的 pointerId 是否存在，如果存在就在 move 的时候取出其坐标，并绘制出来。

![SecondPoint](http://ww3.sinaimg.cn/large/006y8lVagw1fbs1vkt1mwj30bo0jq3z3.jpg)

> 虽然逻辑简单，但个人感觉写起来还是有些麻烦，如果有更简单的方案欢迎告诉我。



## 二、如何使用多点触控

多点触控应用还是比较广泛的，至少目前大部分的图片查看都需要用到多点触控技术(用于拖动和缩放图片)。

但是在某些看似不需要多触控的地方也需要对多点触控进行判断，只要是多点触控可能引起错误的地方都应该加上多点触控的判断。例如使用到 move 事件的时候，由于 move 事件可能由多个手指同时触发，所以可能会出现同时被多个手指控制的情况，如果不适当的处理，这个 move 就可能由任何一个手指触发。

举一个简单的例子：

如果我们需要一个**可以用单指拖动的图片**。假如我们不进行多指触控的判断，像下面这样：

**没有针对多指触控处理版本：**

```java
/**
 * 一个可以拖图片动的 View
 */
public class DragView1 extends CustomView {
    String TAG = "Gcs";

    Bitmap mBitmap;         // 图片
    RectF mBitmapRectF;     // 图片所在区域
    Matrix mBitmapMatrix;   // 控制图片的 matrix

    boolean canDrag = false;
    PointF lastPoint = new PointF(0, 0);

    public DragView1(Context context) {
        this(context, null);
    }

    public DragView1(Context context, AttributeSet attrs) {
        super(context, attrs);

      	// 调整图片大小
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.outWidth = 960/2;
        options.outHeight = 800/2;

        mBitmap = BitmapFactory.decodeResource(this.getResources(), R.drawable.drag_test, options);
        mBitmapRectF = new RectF(0,0,mBitmap.getWidth(), mBitmap.getHeight());
        mBitmapMatrix = new Matrix();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                // 判断按下位置是否包含在图片区域内
                if (mBitmapRectF.contains((int)event.getX(), (int)event.getY())){
                    canDrag = true;
                    lastPoint.set(event.getX(), event.getY());
             }
                break;
            case MotionEvent.ACTION_UP:
                canDrag = false;
            case MotionEvent.ACTION_MOVE:
                if (canDrag) {
                    // 移动图片
                    mBitmapMatrix.postTranslate(event.getX() - lastPoint.x, event.getY() - lastPoint.y);
                    // 更新上一次点位置
                    lastPoint.set(event.getX(), event.getY());

                    // 更新图片区域
                    mBitmapRectF = new RectF(0, 0, mBitmap.getWidth(), mBitmap.getHeight());
                    mBitmapMatrix.mapRect(mBitmapRectF);

                    invalidate();
                }
                break;
        }

        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawBitmap(mBitmap, mBitmapMatrix, mDeafultPaint);
    }
}
```

这个版本非常简单，当然了，如果正常使用(只使用一个手指)的话也不会出问题，但是当使用多个手指，且有抬起和按下的时候就可能出问题，下面用一个典型的场景演示一下：

![dragview1](http://ww2.sinaimg.cn/large/006y8lVagw1fbs1vm2rppg308c0d4kfm.gif)

注意在第二个手指按下，第一个手指抬起时，此时原本的第二个手指会被识别为第一个，所以图片会直接跳动到第二个手指位置。

为了不出现这种情况，我们可以判断一下 pointId 并且只获取第一个手指的数据，这样就能避免这种情况发生了，如下。

**针对多指触控处理后版本：**

```java
/**
 * 一个可以拖图片动的 View
 */
public class DragView extends CustomView {
    String TAG = "Gcs";

    Bitmap mBitmap;         // 图片
    RectF mBitmapRectF;     // 图片所在区域
    Matrix mBitmapMatrix;   // 控制图片的 matrix

    boolean canDrag = false;
    PointF lastPoint = new PointF(0, 0);

    public DragView(Context context) {
        this(context, null);
    }

    public DragView(Context context, AttributeSet attrs) {
        super(context, attrs);

        BitmapFactory.Options options = new BitmapFactory.Options();
        options.outWidth = 960/2;
        options.outHeight = 800/2;

        mBitmap = BitmapFactory.decodeResource(this.getResources(), R.drawable.drag_test, options);
        mBitmapRectF = new RectF(0,0,mBitmap.getWidth(), mBitmap.getHeight());
        mBitmapMatrix = new Matrix();
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_POINTER_DOWN:
                // ▼ 判断是否是第一个手指 && 是否包含在图片区域内
                if (event.getPointerId(event.getActionIndex()) == 0 && mBitmapRectF.contains((int)event.getX(), (int)event.getY())){
                    canDrag = true;
                    lastPoint.set(event.getX(), event.getY());
                }
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_POINTER_UP:
                // ▼ 判断是否是第一个手指
                if (event.getPointerId(event.getActionIndex()) == 0){
                    canDrag = false;
                }
                break;
            case MotionEvent.ACTION_MOVE:
                // 如果存在第一个手指，且这个手指的落点在图片区域内
                if (canDrag) {
                    // ▼ 注意 getX 和 getY
                    int index = event.findPointerIndex(0);
                    // Log.i(TAG, "index="+index);
                    mBitmapMatrix.postTranslate(event.getX(index)-lastPoint.x, event.getY(index)-lastPoint.y);
                    lastPoint.set(event.getX(index), event.getY(index));

                    mBitmapRectF = new RectF(0,0,mBitmap.getWidth(), mBitmap.getHeight());
                    mBitmapMatrix.mapRect(mBitmapRectF);

                    invalidate();
                }
                break;
        }

        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawBitmap(mBitmap, mBitmapMatrix, mDeafultPaint);
    }
}
```

可以看到，比起上一个版本，只添加了少量代码，就变得更加“智能”了，可以准确识别某一个手指，不会因为手指抬起而认错手指。

![dragview2](http://ww2.sinaimg.cn/large/006y8lVagw1fbs1vmwpu4g308c0d4nf9.gif)

**重点注意最后，第一个手指抬起之后，图片并没有跳跃到第二个手指的位置。**

上面的两个对比示例都精简到了极致，其核心依旧是正确的追踪某一个手指，建议大家自己写一遍体会一下。

------

我感觉很多人看到这里依旧是不明所以的，一些简单的东西还好弄，但是复杂一些，如同时处理多个手指的数值就有些困难了，**假如说你之前没有接触过多点触控的处理，此时让你实现用两个手指来缩放图片还是有些困难的。**

因为这不仅要追踪两个手指的位置，还要根据位置变化来计算缩放比例和缩放中心，单单这两个非常简单的数学问题就能难倒一大批人。

![-510a02a1a3f8d16d](http://ww2.sinaimg.cn/large/006y8lVagw1fbs1vnfni1g307b07ct8m.gif)

当然了，很多麻烦问题都有简单的解决方案，假如说我们真的要实现一个可以用两个或者多个手指缩放的控件，何必要自己算呢，可以尝试一下 Android 自带的解决方案：**手势检测(GestureDetector)**，不仅能自动帮你计算好缩放比例和缩放中心，而且还可以检测出 单击、长按、滑屏 等不同的手势，不过这就不是本篇的事情了，以后有时间会写一下有关手势检测的用法(继续挖坑)。

## 三、总结

前段时间因为各种事情比较忙，这篇文章也没时间去写，所以就一直拖到了现在，期间收到不少读者催更，实在是抱歉了。今后在会尽量保证稳定更新的，争取尽快把自定义View系列这一个大坑填完。 ˊ_>ˋ

关于多点触控，个人认为还算一个比较重要的知识点。尤其是随着 Android 的发展，很多炫酷的交互操作可能会需要用户进行拖拽操作。在进行这类操作的时候进行一下手指的判断还是相当重要的。

本文中需要注意的几个知识点：

- 如何兼容 2.0 版本的多点触控(目前大部分都不需要兼容 2.0 了吧)。
- actionIndex、pointIndex 与 pointId 的区别和用法。
- 如何在多点触控中正确的追踪一个手指。

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

## 参考资料

[MotionEvent ](https://developer.android.com/reference/android/view/MotionEvent.html)  



[motionevent]: http://www.gcssloop.com/customview/motionevent