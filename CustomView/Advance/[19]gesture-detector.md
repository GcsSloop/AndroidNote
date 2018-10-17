# Android 手势检测(GestureDetector)

Android 手势检测，主要是 GestureDetector 相关内容的用法和注意事项，本文依旧属于事件处理这一体系，部分内容会涉及到之前文章提及过的知识点，如果你没看过之前的文章，可以到 [自定义 View 系列](http://www.gcssloop.com/customview/CustomViewIndex) 来查看这些内容。

在开发 Android 手机应用过程中，可能需要对一些手势作出响应，如：单击、双击、长按、滑动、缩放等。这些都是很常用的手势。就拿最简单的双击来说吧，假如我们需要判断一个控件是否被双击(即在较短的时间内快速的点击两次)，似乎是一个很容易的任务，但仔细考虑起来，要处理的细节问题也有不少，例如：

1. **记录点击次数**，为了判断是否被点击超过 1 次，所以必须记录点击次数。
2. **记录点击时间**，由于双击事件是较快速的点击两次，像点击一次后，过来几分钟再点击一次肯定不能算是双击事件，所以在记录点击次数的同时也要记录上一次的点击时间，我们可以设置本次点击距离上一次时间超过一定时间(例如：超过100ms)就不识别为双击事件。
3. **点击状态重置**，在响应双击事件，或者判断不是双击事件的时候要重置计数器和上一次点击时间。重置既可以在点击的时候判断并进行重新设置，也可以使用定时器等超过一定时间后重置状态。

这样看起来，判断一个双击事件就有这么多麻烦事情，更别其他的手势了，虽然这些看起来都很简单，但设计起来需要考虑的细节情况实在是太多了。

那么有没有一种更好的方法来方便的检测手势呢？当然有啦，因为这些手势很常用，系统早就封装了一些方法给我们用，接下来我们就看看它们是如何使用的。

## GestureDetector

> GestureDetector 可以使用 MotionEvents 检测各种手势和事件。GestureDetector.OnGestureListener 是一个回调方法，在发生特定的事件时会调用 Listener 中对应的方法回调。这个类只能用于检测触摸事件的 MotionEvent，不能用于轨迹球事件。    
> (话说轨迹球已经消失多长时间了，估计很多人都没见过轨迹球这种东西)。
>
> 如何使用：
>
> - 创建一个 GestureDetector 实例。
> - 在onTouchEvent（MotionEvent）方法中，确保调用 GestureDetector 实例的 onTouchEvent（MotionEvent）。回调中定义的方法将在事件发生时执行。
> - 如果侦听 onContextClick（MotionEvent），则必须在 View 的 onGenericMotionEvent（MotionEvent）中调用 GestureDetector OnGenericMotionEvent（MotionEvent）。

 GestureDetector 本身的方法比较少，使用起来也非常简单，下面让我们先看一下它的简单使用示例，分解开来大概需要三个步骤。

```java
// 1.创建一个监听回调
SimpleOnGestureListener listener = new SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击666", Toast.LENGTH_SHORT).show();
        return super.onDoubleTap(e);
    }
};

// 2.创建一个检测器
final GestureDetector detector = new GestureDetector(this, listener);

// 3.给监听器设置数据源
view.setOnTouchListener(new View.OnTouchListener() {
    @Override public boolean onTouch(View v, MotionEvent event) {
        return detector.onTouchEvent(event);
    }
});
```

接下来我们先了解一下 GestureDetector 里面都有哪些内容。

### 1. 构造函数

GestureDetector 一共有 5 种构造函数，但有 2 种被废弃了，1 种是重复的，所以我们只需要关注其中的 2 种构造函数即可，如下：

| 构造函数                                     |
| ---------------------------------------- |
| GestureDetector(Context context, GestureDetector.OnGestureListener listener) |
| GestureDetector(Context context, GestureDetector.OnGestureListener listener, Handler handler) |

第 1 种构造函数里面需要传递两个参数，上下文(Context) 和 手势监听器(OnGestureListener)，这个很容易理解，就不再过多叙述，上面的例子中使用的就是这一种。

第 2 种构造函数则需要多传递一个 Handler 作为参数，这个有什么作用呢？其实作用也非常简单，这个 Handler 主要是为了给 GestureDetector 提供一个 Looper。

在通常情况下是不需这个 Handler 的，因为它会在内部自动创建一个 Handler 用于处理数据，如果你在主线程中创建 GestureDetector，那么它内部创建的 Handler 会自动获得主线程的 Looper，然而如果你在一个没有创建 Looper 的子线程中创建 GestureDetector 则需要传递一个带有 Looper 的 Handler 给它，否则就会因为无法获取到 Looper 导致创建失败。

第 2 种构造函数使用方式如下(下面是两种在子线程中创建 GestureDetector 的方法)：

```java
// 方式一、在主线程创建 Handler
final Handler handler = new Handler();
new Thread(new Runnable() {
    @Override public void run() {
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();

// 方式二、在子线程创建 Handler，并且指定 Looper
new Thread(new Runnable() {
    @Override public void run() {
        final Handler handler = new Handler(Looper.getMainLooper());
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener() , handler);
        // ... 省略其它代码 ...
    }
}).start();
```

当然了，使用其它创建 Handler 的方式也是可以的，重点传递的 Handler 一定要有 Looper，敲黑板，重点是 Handler 中的 Looper。假如子线程准备了 Looper 那么可以直接使用第 1 种构造函数进行创建，如下：

```java
new Thread(new Runnable() {
    @Override public void run() {
        Looper.prepare(); // <- 重点在这里
        final GestureDetector detector = new GestureDetector(MainActivity.this, new
                GestureDetector.SimpleOnGestureListener());
        // ... 省略其它代码 ...
    }
}).start();
```

### 2.手势监听器

既然是手势检测，自然要在对应的手势出现的时候通知调用者，最合适的自然是事件监听器模式。目前 GestureDetecotr 有四种监听器。

| 监听器                                      | 简介                                       |
| ---------------------------------------- | ---------------------------------------- |
| [OnContextClickListener](https://developer.android.com/reference/android/view/GestureDetector.OnContextClickListener.html) | 这个很容易让人联想到ContextMenu，然而它和ContextMenu并没有什么关系，它是在Android6.0(API 23)才添加的一个选项，是用于检测外部设备上的按钮是否按下的，例如蓝牙触控笔上的按钮，一般情况下，忽略即可。 |
| [OnDoubleTapListener](https://developer.android.com/reference/android/view/GestureDetector.OnDoubleTapListener.html) | 双击事件，有三个回调类型：双击(DoubleTap)、单击确认(SingleTapConfirmed) 和 双击事件回调(DoubleTapEvent) |
| [OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html) | 手势检测，主要有以下类型事件：按下(Down)、 一扔(Fling)、长按(LongPress)、滚动(Scroll)、触摸反馈(ShowPress) 和 单击抬起(SingleTapUp) |
| [SimpleOnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.SimpleOnGestureListener.html) | 这个是上述三个接口的空实现，一般情况下使用这个比较多，也比较方便。        |

#### 2.1 OnContextClickListener

由于 OnContextClickListener 主要是用于检测外部设备按钮的，关于它需要注意一点，如果侦听 onContextClick(MotionEvent)，则必须在 View 的 onGenericMotionEvent(MotionEvent)中调用 GestureDetector 的 OnGenericMotionEvent(MotionEvent)。

由于目前我们用到这个监听器的场景并不多，所以也就不展开介绍了，重点关注后面几个监听器。

#### 2.2 OnDoubleTapListener

这个很明显就是用于检测双击事件的，它有三个回调接口，分别是 onDoubleTap、onDoubleTapEvent 和 onSingleTapConfirmed。

##### **2.2.1 onDoubleTap 与 onSingleTapConfirmed**

**如果你只想监听双击事件，那么只用关注 onDoubleTap 就行了，如果你同时要监听单击事件则需要关注 onSingleTapConfirmed 这个回调函数**。

有人可能会有疑问，监听单击事件为什么要使用 onSingleTapConfirmed，使用 OnClickListener 不行吗？从理论上是可行的，但是我并不推荐这样使用，主要有两个原因：  
1.它们两个是存在一定冲突的，如果你看过 [事件分发机制详解](http://www.gcssloop.com/customview/dispatch-touchevent-source) 就会知道，如果想要两者同时被触发，则 setOnTouchListener 不能消费事件，如果 onTouchListener 消费了事件，就可能导致 OnClick 无法正常触发。  
2.需要同时监听单击和双击，则说明单击和双击后响应逻辑不同，然而使用 OnClickListener 会在双击事件发生时触发两次，这显然不是我们想要的结果。而使用 onSingleTapConfirmed 就不用考虑那么多了，你完全可以把它当成单击事件来看待，而且在双击事件发生时，onSingleTapConfirmed 不会被调用，这样就不会引发冲突。

如果你需要同时监听两种点击事件可以这样写：

```java
GestureDetector detector = new GestureDetector(this, new GestureDetector
        .SimpleOnGestureListener() {
    @Override public boolean onSingleTapConfirmed(MotionEvent e) {
        Toast.makeText(MainActivity.this, "单击", Toast.LENGTH_SHORT).show();
        return false;
    }
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击", Toast.LENGTH_SHORT).show();
        return false;
    }
});
```

关于 onSingleTapConfirmed 原理也非常简单，这一个回调函数在单击事件发生后300ms后触发(注意，不是立即触发的)，只有在确定不会有后续的事件后，既当前事件肯定是单击事件才触发 onSingleTapConfirmed，所以在进行点击操作时，onDoubleTap 和 onSingleTapConfirmed 只会有一个被触发，也就不存在冲突了。

当然，如果你对事件分发机制非常了解的话，随便怎么用都行，条条大路通罗马，我这里只是推荐一种最简单而且不容易出错的实现方案。

##### **2.2.2 onDoubleTapEvent**

**有些细心的小伙伴可能注意到还有一个 onDoubleTapEvent 回调函数，它是干什么的呢？它在双击事件确定发生时会对第二次按下产生的 MotionEvent 信息进行回调。**

至于为什么要存在这样的回调，就要涉及到另一个比较细致的问题了，那就是  onDoubleTap 的触发时间，如果你在这些函数被调用时打印一条日志，那么你会看到这样的信息：

```
GCS-LOG: onDoubleTap
GCS-LOG: onDoubleTapEvent - down
GCS-LOG: onDoubleTapEvent - move
GCS-LOG: onDoubleTapEvent - move
GCS-LOG: onDoubleTapEvent - up
```

通过观察这些信息你会发现它们的调用顺序非常有趣，首先是 onDoubleTap 被触发，之后依次触发 onDoubleTapEvent 的 down、move、up 等信息，为什么说它们有趣呢？是因为这样的调用顺序会引发两种猜想，第一种猜想是 onDoubleTap 是在第二次手指抬起(up)后触发的，而 onDoubleTapEvent 是一种延时回调。第二种猜想则是 onDoubleTap 在第二次手指按下(dowm)时触发，onDoubleTapEvent 是一种实时回调。

通过测试和观察源码发现第二种猜想是正确的，因为第二次按下手指时，即便不抬起也会触发 onDoubleTap 和 onDoubleTapEvent 的 down，而且源码中逻辑也表明 onDoubleTapEvent 是一种实时回调。

这就引发了另一个问题，双击的触发时间，虽然这是一个细微到很难让人注意到的问题，假如说我们想要在第二次按下抬起后才判定这是一个双击操作，触发后续的内容，则不能使用 onDoubleTap 了，需要使用 onDoubleTapEvent 来进行更细微的控制，如下：

```java
final GestureDetector detector = new GestureDetector(MainActivity.this, new GestureDetector.SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Logger.e("第二次按下时触发");
        return super.onDoubleTap(e);
    }

    @Override public boolean onDoubleTapEvent(MotionEvent e) {
        switch (e.getActionMasked()) {
            case MotionEvent.ACTION_UP:
                Logger.e("第二次抬起时触发");
                break;
        }
        return super.onDoubleTapEvent(e);
    }
});
```

如果你不需要控制这么细微的话，忽略即可(Logger 是我自己封装的日志库，忽略即可)。

#### 2.3 OnGestureListener

这个是手势检测中较为核心的一个部分了，主要检测以下类型事件：按下(Down)、 一扔(Fling)、长按(LongPress)、滚动(Scroll)、触摸反馈(ShowPress) 和 单击抬起(SingleTapUp)。

##### 2.3.1 onDown

```java
@Override public boolean onDown(MotionEvent e) {
    return true;
}
```

看过前面的文章应该知道，down 在事件分发体系中是一个较为特殊的事件，为了保证事件被唯一的 View 消费，哪个 View 消费了 down 事件，后续的内容就会传递给该 View。如果我们想让一个 View 能够接收到事件，有两种做法：

1、让该 View 可以点击，因为可点击状态会默认消费 down 事件。

2、手动消费掉 down 事件。

由于图片、文本等一些控件默认是不可点击的，所以我们要么声明它们的 clickable 为 true，要么在发生 down 事件是返回 true。所以 onDown 在这里的作用就很明显了，就是为了保证让该控件能拥有消费事件的能力，以接受后续的事件。

##### 2.3.2 onFling

Failing 中文直接翻译过来就是一扔、抛、甩，最常见的场景就是在 ListView 或者 RecyclerView 上快速滑动时手指抬起后它还会滚动一段时间才会停止。onFling 就是检测这种手势的。

```java
@Override
public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float
        velocityY) {
    return super.onFling(e1, e2, velocityX, velocityY);
}
```

在 onFling 的回调中共有四个参数，分别是：

| 参数        | 简介                 |
| --------- | ------------------ |
| e1        | 手指按下时的 Event。      |
| e2        | 手指抬起时的 Event。      |
| velocityX | 在 X 轴上的运动速度(像素／秒)。 |
| velocityY | 在 Y 轴上的运动速度(像素／秒)。 |

我们可以通过 e1 和 e2 获取到手指按下和抬起时的坐标、时间等相关信息，通过 velocityX 和 velocityY 获取到在这段时间内的运动速度，单位是像素／秒(即 1 秒内滑动的像素距离)。

这个我们自己用到的地方比较少，但是也可以帮助我们简单的做出一些有趣的效果，例如下面的这种弹球效果，会根据滑动的力度和方向产生不同的弹跳效果。

![](https://ww2.sinaimg.cn/large/006tKfTcgy1fhe10uwa4fg308c0e6wn5.gif)

其实这种原理非常简单，简化之后如下：

1. 记录 velocityX 和 velocityY 作为初始速度，之后不断让速度衰减，直至为零。
2. 根据速度和当前小球的位置计算一段时间后的位置，并在该位置重新绘制小球。
3. 判断小球边缘是否碰触控件边界，如果碰触了边界则让速度反向。

根据这三条基本的逻辑就可以做出比较像的弹球效果，[具体的Demo可以看这里](https://raw.githubusercontent.com/GcsSloop/AndroidNote/master/CustomView/Demo/FailingBall.zip)。

##### 2.3.3 onLongPress

这个是检测长按事件的，即手指按下后不抬起，在一段时间后会触发该事件。

```java
@Override 
public void onLongPress(MotionEvent e) {
}
```

##### 2.3.4 onScroll

onScroll 就是监听滚动事件的，它看起来和 onFaling 比较像，不同的是，onSrcoll 后两个参数不是速度，而是滚动的距离。

```java
@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float 
        distanceY) {
    return super.onScroll(e1, e2, distanceX, distanceY);
}
```

| 参数        |             |
| --------- | ----------- |
| e1        | 手指按下时的Event |
| e2        | 手指抬起时的Event |
| distanceX | 在 X 轴上划过的距离 |
| distanceY | 在 Y 轴上划过的距离 |

##### 2.3.5 onShowPress

它是用户按下时的一种回调，主要作用是给用户提供一种视觉反馈，可以在监听到这种事件时可以让控件换一种颜色，或者产生一些变化，告诉用户他的动作已经被识别。

不过这个消息和 onSingleTapConfirmed 类似，也是一种延时回调，延迟时间是 180 ms，假如用户手指按下后立即抬起或者事件立即被拦截，时间没有超过 180 ms的话，这条消息会被 remove 掉，也就不会触发这个回调。

```java
@Override 
public void onShowPress(MotionEvent e) {
}
```

##### 2.3.6 onSingleTapUp

```java
@Override 
public boolean onSingleTapUp(MotionEvent e) {
    return super.onSingleTapUp(e);
}
```

这个也很容易理解，就是用户单击抬起时的回调，但是它和上面的 `onSingleTapConfirmed` 之间有何不同呢？和 `onClick` 又有何不同呢？

单击事件触发：

```java
GCS: onSingleTapUp
GCS: onClick
GCS: onSingleTapConfirmed
```

| 类型                   | 触发次数 | 摘要   |
| -------------------- | ---- | ---- |
| onSingleTapUp        | 1    | 单击抬起 |
| onSingleTapConfirmed | 1    | 单击确认 |
| onClick              | 1    | 单击事件 |

双击事件触发：

```java
GCS: onSingleTapUp
GCS: onClick
GCS: onDoubleTap // <- 双击
GCS: onClick
```

| 类型                   | 触发次数 | 摘要           |
| -------------------- | ---- | ------------ |
| onSingleTapUp        | 1    | 在双击的第一次抬起时触发 |
| onSingleTapConfirmed | 0    | 双击发生时不会触发。   |
| onClick              | 2    | 在双击事件时触发两次。  |

可以看出来这三个事件还是有所不同的，根据自己实际需要进行使用即可

#### 2.4 SimpleOnGestureListener

这个里面并没有什么内容，只是对上面三种 Listener 的空实现，在上面的例子中使用的基本都是这监听器。因为它用起来更方便一点。

这主要是 GestureDetector 构造函数的设计问题，以只监听 OnDoubleTapListener 为例，如果想要使用 OnDoubleTapListener 接口则需要这样进行设置：

```java
GestureDetector detector = new GestureDetector(this, new GestureDetector
        .SimpleOnGestureListener());
detector.setOnDoubleTapListener(new GestureDetector.OnDoubleTapListener() {
    @Override public boolean onSingleTapConfirmed(MotionEvent e) {
        Toast.makeText(MainActivity.this, "单击确认", Toast.LENGTH_SHORT).show();
        return false;
    }

    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击", Toast.LENGTH_SHORT).show();
        return false;
    }

    @Override public boolean onDoubleTapEvent(MotionEvent e) {
        // Toast.makeText(MainActivity.this,"",Toast.LENGTH_SHORT).show();
        return false;
    }
});
```

既然都已经创建 SimpleOnGestureListener 了，再创建一个 OnDoubleTapListener 显然十分浪费，如果构造函数不使用 SimpleOnGestureListener，而是使用 OnGestureListener 的话，会多出几个无用的空实现，显然很浪费，所以在一般情况下，老老实实的使用 SimpleOnGestureListener 就好了。

### 3. 相关方法

除了各类监听器之外，与 GestureDetector 相关的方法其实并不多，只有几个，下面来简单介绍一下。

| 方法                      | 摘要                                       |
| ----------------------- | ---------------------------------------- |
| setIsLongpressEnabled   | 通过布尔值设置是否允许触发长按事件，true 表示允许，false 表示不允许。 |
| isLongpressEnabled      | 判断当前是否允许触发长按事件，true 表示允许，false 表示不允许。    |
| onTouchEvent            | 这个是其中一个重要的方法，在最开始已经演示过使用方式了。             |
| onGenericMotionEvent    | 这个是在 API 23 之后才添加的内容，主要是为 OnContextClickListener 服务的，暂时不用关注。 |
| setContextClickListener | 设置 ContextClickListener 。                |
| setOnDoubleTapListener  | 设置 OnDoubleTapListener 。                 |

### 结语

关于手势检测部分的 GestureDetector 相关内容基本就这么多了，其实手势检测还有一个 ScaleGestureDetector 也是为手势检测服务的，限于篇幅，本次就讲这么多吧。

其实手势检测辅助类 GestureDetector 本身并不是很复杂，带上注释等内容才不到1000行，感兴趣的可以自己研究一下实现方式。

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

## 参考资料

[文档 · GestureDetector ](https://developer.android.com/reference/android/view/GestureDetector.html)   
[源码 · GestureDetector](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/java/android/view/GestureDetector.java)