# Android 手势检测(GestureDetector)

因为工作等原因，已经很长时间没有写文章了，趁节假日来水一篇简单的文章，Android 手势检测，如题，本文依旧是和事件相关的，如果你没看过之前的文章，可以到 [自定义 View 系列](http://www.gcssloop.com/customview/CustomViewIndex) 来查看前面的内容。

在开发 Android 手机应用过程中，可能需要对一些手势作出响应，如：单击、双击、长按、滑动、缩放等。这些都是很常用的手势。就拿最简单的双击来说吧，假如我们需要判断一个控件是否被双击(即在较短的时间内快速的点击两次)，似乎是一个很容易的任务，但仔细考虑起来，要处理的细节问题也有不少，例如：

1. **记录点击次数**，为了判断是否被点击超过 1 次，所以必须记录点击次数。
2. **记录点击时间**，由于双击事件是较快速的点击两次，像点击一次后，过来几分钟再点击一次肯定不能算是双击事件，所以在记录点击次数的同时也要记录上一次的点击时间，我们可以设置本次点击距离上一次时间超过一定时间(例如：超过1s)就不识别为双击事件。
3. **点击状态重置**，在响应双击事件，或者判断不是双击事件的时候要重置计数器和上一次点击时间。重置既可以在点击的时候判断并进行重新设置，也可以使用定时器等超过一定时间后重置状态。

这样看起来，判断一个双击事件就有这么多麻烦事情，更别其他的手势了，虽然这些看起来都很简单，但设计起来需要考虑的细节情况实在是太多了。

那么有没有一种更好的方法来方便的检测手势呢？当然有啦，因为这些手势很常用，系统早就封装了一些方法给我们用，它就是 **GestureDetector** 。我们先看一下关于它的简单介绍：

> GestureDetector 可以使用 MotionEvents 检测各种手势和事件。GestureDetector.OnGestureListener 是一个回调方法，在发生特定的事件时会调用 Listener 中对应的方法回调。这个类只能用于检测触摸事件的 MotionEvent，不能用于轨迹球事件。    
> (话说轨迹球已经消失多长时间了，估计很多人都没见过轨迹球这种东西)。
>
> 如何使用：
>
> - 创建一个 GestureDetector 实例。
> - 在onTouchEvent（MotionEvent）方法中，确保调用 GestureDetector 实例的 onTouchEvent（MotionEvent）。回调中定义的方法将在事件发生时执行。
> - 如果侦听 onContextClick（MotionEvent），则必须在 View 的 onGenericMotionEvent（MotionEvent）中调用 GestureDetector OnGenericMotionEvent（MotionEvent）。

 GestureDetector 本身的方法很少，使用起来也非常简单，下面让我们看一下它的简单使用方法。

```java
// 创建一个监听回调
SimpleOnGestureListener listener = new SimpleOnGestureListener() {
    @Override public boolean onDoubleTap(MotionEvent e) {
        Toast.makeText(MainActivity.this, "双击666", Toast.LENGTH_SHORT).show();
        return super.onDoubleTap(e);
    }
};

// 创建一个检测器
final GestureDetector detector = new GestureDetector(this, listener);

// 给监听器设置数据源
view.setOnTouchListener(new View.OnTouchListener() {
    @Override public boolean onTouch(View v, MotionEvent event) {
        return detector.onTouchEvent(event);
    }
});
```

上面是监听一个双击事件，可以看到它使用起来非常简单，接下来我们就看一下有关于它的详细信息。

