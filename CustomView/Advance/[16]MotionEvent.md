# MotionEvent 详解

Android MotionEvent 详解，之前用了两篇文章 [事件分发机制原理][customview/dispatch-touchevent-theory] 和 [事件分发机制详解][customview/dispatch-touchevent-source] 来讲解事件分发，而作为事件分发主角之一的 MotionEvent 并没有过多的说明，本文就带大家了解 MotionEvent 的相关内容，简要介绍触摸事件，主要包括 单点触控、多点触控、鼠标事件 以及 getAction() 和 getActionMasked() 的区别。

Android 将所有的输入事件都放在了 MotionEvent 中，随着安卓的不断发展壮大，MotionEvent 也开始变得越来越复杂，下面是我自己整理的 MotionEvent 大事记:   

|          版本号          | 更新内容                        |
| :-------------------: | --------------------------- |
| Android 1.0 (API  1 ) | 支持单点触控和轨迹球的事件。              |
| Android 1.6 (API  4 ) | 支持手势。                       |
| Android 2.2 (API  8 ) | 支持多点触控。                     |
| Android 3.1 (API 12)  | 支持触控笔，鼠标，键盘，操纵杆，游戏控制器等输入工具。 |

以上仅仅是简要的说明几次比较大的变动，细小的修复和更新不计其数，此处就不一一列出了，反正也没人关心这些东西。  
MotionEvent 负责集中处理所有类型设备的输入事件，但是由于某些设备使用的几率较小本文会忽略讲解，或者简要讲解，例如：  
1、轨迹球只出现在最早的设备上，现代的设备上已经见不到了，本文不再叙述。  
2、触控笔和手指处理流程基本相同，不再多说。  
3、鼠标在手机上使用概率也比较小，会在文末简要介绍。  



## 单点触控

单点触控就非常简单啦，入门的工程师都会用，上一篇文章也简要介绍过，主要涉及以下几个事件:

| 事件             | 简介                       |
| -------------- | ------------------------ |
| ACTION_DOWN    | 手指 **初次接触到屏幕** 时触发。      |
| ACTION_MOVE    | 手指 **在屏幕上滑动** 时触发，会多次触发。 |
| ACTION_UP      | 手指 **离开屏幕** 时触发。         |
| ACTION_CANCEL  | 事件 **被上层拦截** 时触发。        |
| ACTION_OUTSIDE | 手指 **不在控件区域** 时触发。       |

和以下的几个方法:

| 方法          | 简介                     |
| :---------- | ---------------------- |
| getAction() | 获取事件类型。                |
| getX()      | 获得触摸点在当前 View 的 X 轴坐标。 |
| getY()      | 获得触摸点在当前 View 的 Y 轴坐标。 |
| getRawX()   | 获得触摸点在整个屏幕的 X 轴坐标。     |
| getRawY()   | 获得触摸点在整个屏幕的 Y 轴坐标。     |

> 关于 `get` 和 `getRaw`  的区别可以参考这一篇文章 [安卓自定义View基础-坐标系][customview/CoordinateSystem]

单点触控一次简单的交互流程是这样的:

**手指落下(ACTION_DOWN) －> 多次移动(ACTION_MOVE) －> 离开(ACTION_UP)**  

> * 本次事例中 ACTION_MOVE 有多次触发。
> * 如果仅仅是单击(手指按下再抬起)，不会触发 ACTION_MOVE。

![单点触摸事件流程](http://ww4.sinaimg.cn/large/005Xtdi2jw1f8oz1704ylg30bo0jqgmx.gif)

针对单点触控的事件处理一般是这样写的:

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    // ▼ 注意这里使用的是 getAction()，先埋一个小尾巴。
    switch (event.getAction()){
        case MotionEvent.ACTION_DOWN:
        	// 手指按下
            break;
        case MotionEvent.ACTION_MOVE:
            // 手指移动
            break;
        case MotionEvent.ACTION_UP:
            // 手指抬起
            break;
        case MotionEvent.ACTION_CANCEL:
            // 事件被拦截 
            break;
        case MotionEvent.ACTION_OUTSIDE:
            // 超出区域 
            break;
    }
    return super.onTouchEvent(event);
}
```

相信小伙伴对此已经非常熟悉了，经常使用的东西，我也不啰嗦了。

但其中有两个比较特殊的事件:  `ACTION_CANCEL` 和  `ACTION_OUTSIDE` 。  
为什么说特殊呢，因为它们是由程序触发而产生的，而且触发条件也非常特殊，通常情况下即便不处理这两个事件也没有什么问题。接下来我们就扒一扒它们的真面目:



### ACTION_CANCEL

**`ACTION_CANCEL` 的触发条件是事件被上层拦截**，然而我们在 [事件分发机制原理][customview/dispatch-touchevent-theory] 一文中了解到当事件被上层 View 拦截的时候，ChildView 是收不到任何事件的，ChildView 收不到任何事件，自然也不会收到 `ACTION_CANCEL` 了，所以说这个 `ACTION_CANCEL` 的正确触发条件并不是这样，那么是什么呢？

**事实上，只有上层 View 回收事件处理权的时候，ChildView 才会收到一个 `ACTION_CANCEL` 事件。**

这样说可能不太容易理解，咱举个例子? 

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9auctayt4j302s03o744.jpg)

> 例如：上层 View 是一个 RecyclerView，它收到了一个 `ACTION_DOWN` 事件，由于这个可能是点击事件，所以它先传递给对应 ItemView，询问 ItemView 是否需要这个事件，然而接下来又传递过来了一个 `ACTION_MOVE` 事件，且移动的方向和 RecyclerView 的可滑动方向一致，所以 RecyclerView 判断这个事件是滚动事件，于是要收回事件处理权，这时候对应的 ItemView 会收到一个 `ACTION_CANCEL` ，并且不会再收到后续事件。
>
> **通俗一点？**
>
> RecyclerView：儿砸，这里有一个 `ACTION_DOWN` 你看你要不要。  
> ItemView       ：好嘞，我看看。  
> RecyclerView：噫？居然是移动事件`ACTION_MOVE`，我要滚起来了，儿砸，我可能要把你送去你姑父家(缓存区)了，在这之前给你一个 `ACTION_CANCEL`，你要收好啊。  
> ItemView       ：…...
>
> 这是实际开发中最有可能见到 `ACTION_CANCEL` 的场景了。



### ACTION_OUTSIDE

`ACTION_OUTSIDE`的触发条件更加奇葩，从字面上看，outside 意思不就是超出区域么？然而不论你如何滑动超出控件区域都不会触发 `ACTION_OUTSIDE` 这个事件。相信很多魔法师都对此很是疑惑，说好的超出区域呢？

实际上这个事件根本就不是在这里用的，看官方解释(装一下逼)：

>  A movement has happened outside of the normal bounds of the UI element. This does not provide a full gesture, but only the initial location of the movement/touch.
>
> 一个触摸事件已经发生了UI元素的正常范围之外。因此不再提供完整的手势，只提供 运动/触摸 的初始位置。

我们知道，正常情况下，如果初始点击位置在该视图区域之外，该视图根本不可能会收到事件，然而，万事万物都不是绝对的，肯定还有一些特殊情况，你可曾还记得点击 Dialog 区域外关闭吗？Dialog 就是一个特殊的视图(没有占满屏幕大小的窗口)，能够接收到视图区域外的事件(虽然在通常情况下你根本用不到这个事件)，除了 Dialog 之外，你最可能看到这个事件的场景是悬浮窗，当然啦，想要接收到视图之外的事件需要一些特殊的设置。

> 设置视图的 WindowManager 布局参数的 flags为[`FLAG_WATCH_OUTSIDE_TOUCH`](http://developer.android.com/reference/android/view/WindowManager.LayoutParams.html#FLAG_WATCH_OUTSIDE_TOUCH)，这样点击事件发生在这个视图之外时，该视图就可以接收到一个 `ACTION_OUTSIDE` 事件。
>
> 参见StackOverflow：[How to dismiss the dialog with click on outside of the dialog?](http://stackoverflow.com/questions/8384067/how-to-dismiss-the-dialog-with-click-on-outside-of-the-dialog)

由于这个事件用到的几率比较小，此处就不展开叙述了，以后用到的时候再详细讲解。



## 多点触控

Android 在 2.2 版本的时候开始支持多点触控，一旦出现了多点触控，很多东西就突然之间变得麻烦起来了，首先要解决的问题就是 **多个手指同时按在屏幕上，会产生很多的事件，这些事件该如何区分呢？**

为了区分这些事件，工程师们用了一个很简单的办法－－**编号，当手指第一次按下时产生一个唯一的号码，手指抬起或者事件被拦截就回收编号，就这么简单。** 

**第一次按下的手指特殊处理作为主指针，之后按下的手指作为辅助指针**，然后随之衍生出来了以下事件(注意增加的事件和事件简介的变化)：

| 事件                          | 简介                             |
| --------------------------- | ------------------------------ |
| ACTION_DOWN                 | **第一个** 手指 **初次接触到屏幕** 时触发。    |
| ACTION_MOVE                 | 手指 **在屏幕上滑动** 时触发，会多次触发。       |
| ACTION_UP                   | **最后一个** 手指 **离开屏幕** 时触发。      |
| **ACTION_POINTER_DOWN**     | 有非主要的手指按下(**即按下之前已经有手指在屏幕上**)。 |
| **ACTION_POINTER_UP**       | 有非主要的手指抬起(**即抬起之后仍然有手指在屏幕上**)。 |
| 以下事件类型不推荐使用                 | －－－－－－－－－－－－－－－－－－             |
| ~~ACTION_POINTER\_1\_DOWN~~ | 第 2 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_2\_DOWN~~ | 第 3 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_3\_DOWN~~ | 第 4 个手指按下，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_1\_UP~~   | 第 2 个手指抬起，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_2\_UP~~   | 第 3 个手指抬起，已废弃，不推荐使用。           |
| ~~ACTION_POINTER\_3\_UP~~   | 第 4 个手指抬起，已废弃，不推荐使用。           |

和以下方法：

| 方法                              | 简介                                       |
| ------------------------------- | ---------------------------------------- |
| getActionMasked()               | 与 `getAction()` 类似，**多点触控必须使用这个方法获取事件类型**。 |
| getActionIndex()                | 获取该事件是哪个指针(手指)产生的。                       |
| getPointerCount()               | 获取在屏幕上手指的个数。                             |
| getPointerId(int pointerIndex)  | 获取一个指针(手指)的唯一标识符ID，在手指按下和抬起之间ID始终不变。     |
| findPointerIndex(int pointerId) | 通过PointerId获取到当前状态下PointIndex，之后通过PointIndex获取其他内容。 |
| getX(int pointerIndex)          | 获取某一个指针(手指)的X坐标                          |
| getY(int pointerIndex)          | 获取某一个指针(手指)的Y坐标                          |

由于多点触控部分涉及内容比较多，也很复杂，我准备单独用一篇文章进行详细叙述，所以这里只叙述一些基础的内容作为铺垫：

### getAction() 与 getActionMasked()

当多个手指在屏幕上按下的时候，会产生大量的事件，如何在获取事件类型的同时区分这些事件就是一个大问题了。

一般来说我们可以通过为事件添加一个int类型的index属性来区分，但是我们知道谷歌工程师是有洁癖的(在 [自定义View分类与流程][customview/CustomViewProcess] 的onMeasure中已经见识过了)，为了添加一个通常数值不会超过10的index属性就浪费一个int大小的空间简直是不能忍受的，于是工程师们将这个index属性和事件类型直接合并了。

int类型共32位(0x00000000)，他们用最低8位(0x000000**ff**)表示事件类型，再往前的8位(0x0000**ff**00)表示事件编号，以手指按下为例讲解数值是如何合成的:

> ACTION_DOWN                   的默认数值为 (0x00000000)  
> ACTION_POINTER_DOWN 的默认数值为 (0x00000005)

|  手指按下   | 触发事件(数值)                                 |
| :-----: | :--------------------------------------- |
| 第1个手指按下 | ACTION_DOWN                  (0x0000**00**00) |
| 第2个手指按下 | ACTION_POINTER_DOWN (0x0000**01**05)     |
| 第3个手指按下 | ACTION_POINTER_DOWN (0x0000**02**05)     |
| 第4个手指按下 | ACTION_POINTER_DOWN (0x0000**03**05)     |

**注意：**  
上面表格中用粗体标示出的数值，可以看到随着按下手指数量的增加，这个数值也是一直变化的，进而导致我们使用 `getAction()` 获取到的数值无法与标准的事件类型进行对比，为了解决这个问题，他们创建了一个 `getActionMasked()` 方法，这个方法可以清除index数值，让其变成一个标准的事件类型。  
**1、多点触控时必须使用 `getActionMasked()` 来获取事件类型。**  
**2、单点触控时由于事件数值不变，使用 `getAction()` 和 `getActionMasked()` 两个方法都可以。**  
**3、使用 getActionIndex() 可以获取到这个index数值。不过请注意，getActionIndex() 只在 down 和 up 时有效，move 时是无效的。**

目前来说获取事件类型使用 `getActionMasked()` 就行了，但是如果一定要编译时兼容古董版本的话，可以考虑使用这样的写法:

```java
final int action = (Build.VERSION.SDK_INT >= Build.VERSION_CODES.FROYO)
                ? event.getActionMasked()
                : event.getAction();
switch (action){
    case MotionEvent.ACTION_DOWN:
        // TODO
        break;
}
```



### PointId

虽然前面刚刚说了一个 actionIndex，可以使用 getActionIndex() 获得，但通过 actionIndex 字面意思知道，这个只表示事件的序号，而且根据其说明文档解释，这个 ActionIndex 只有在手指按下(down)和抬起(up)时是有用的，在移动(move)时是没有用的，事件追踪非常重要的一环就是移动(move)，然而它却没卵用，这也太不实在了 (￣Д￣)ﾉ

**郑重声明：追踪事件流，请认准 PointId，这是唯一官方指定标准，不要相信 ActionIndex 那个小婊砸。**

PointId 在手指按下时产生，手指抬起或者事件被取消后消失，是一个事件流程中唯一不变的标识，可以在手指按下时 通过 `getPointerId(int pointerIndex)` 获得。 (参数中的 pointerIndex 就是 actionIndex)

关于事件流的追踪等问题在讲解多点触控时再详细讲解。



## 历史数据(批处理)

由于我们的设备非常灵敏，手指稍微移动一下就会产生一个移动事件，所以移动事件会产生的特别频繁，为了提高效率，系统会将近期的多个移动事件(move)按照事件发生的顺序进行排序打包放在同一个 MotionEvent 中，与之对应的产生了以下方法：

| 事件                                | 简介                                       |
| --------------------------------- | ---------------------------------------- |
| getHistorySize()                  | 获取历史事件集合大小                               |
| getHistoricalX(int pos)           | 获取第pos个历史事件x坐标<br/> (pos < getHistorySize()) |
| getHistoricalY(int pos)           | 获取第pos个历史事件y坐标<br/> (pos < getHistorySize()) |
| getHistoricalX (int pin, int pos) | 获取第pin个手指的第pos个历史事件x坐标<br/> (pin < getPointerCount(), pos < getHistorySize() ) |
| getHistoricalY (int pin, int pos) | 获取第pin个手指的第pos个历史事件y坐标<br/> (pin < getPointerCount(), pos < getHistorySize() ) |

**注意：**  

1. pin 全称是 pointerIndex，表示第几个手指，此处为了节省空间使用了缩写。  
2. 历史数据只有 ACTION_MOVE 事件。
3. 历史数据单点触控和多点触控均可以用。

下面是官方文档给出的一个简单使用示例：

```java
void printSamples(MotionEvent ev) {
     final int historySize = ev.getHistorySize();
     final int pointerCount = ev.getPointerCount();
     for (int h = 0; h < historySize; h++) {
         System.out.printf("At time %d:", ev.getHistoricalEventTime(h));
         for (int p = 0; p < pointerCount; p++) {
             System.out.printf("  pointer %d: (%f,%f)",
                 ev.getPointerId(p), ev.getHistoricalX(p, h), ev.getHistoricalY(p, h));
         }
     }
     System.out.printf("At time %d:", ev.getEventTime());
     for (int p = 0; p < pointerCount; p++) {
         System.out.printf("  pointer %d: (%f,%f)",
             ev.getPointerId(p), ev.getX(p), ev.getY(p));
     }
}
```



## 获取事件发生的时间

获取事件发生的时间。

| 方法                              | 简介           |
| ------------------------------- | ------------ |
| getDownTime()                   | 获取手指按下时的时间。  |
| getEventTime()                  | 获取当前事件发生的时间。 |
| getHistoricalEventTime(int pos) | 获取历史事件发生的时间。 |

> 1. pos 表示历史数据中的第几个数据。( pos < getHistorySize() )
> 2. 返回值类型为 long，单位是毫秒。

## 获取压力(接触面积大小)

MotionEvent支持获取某些输入设备(手指或触控笔)的与屏幕的接触面积和压力大小，主要有以下方法：

> 描述中使用了手指，触控笔也是一样的。

| 方法                                       | 简介                           |
| ---------------------------------------- | ---------------------------- |
| getSize ()                               | 获取第1个手指与屏幕接触面积的大小            |
| getSize (int pin)                        | 获取第pin个手指与屏幕接触面积的大小          |
| getHistoricalSize (int pos)              | 获取历史数据中第1个手指在第pos次事件中的接触面积   |
| getHistoricalSize (int pin, int pos)     | 获取历史数据中第pin个手指在第pos次事件中的接触面积 |
| getPressure ()                           | 获取第一个手指的压力大小                 |
| getPressure (int pin)                    | 获取第pin个手指的压力大小               |
| getHistoricalPressure (int pos)          | 获取历史数据中第1个手指在第pos次事件中的压力大小   |
| getHistoricalPressure (int pin, int pos) | 获取历史数据中第pin个手指在第pos次事件中的压力大小 |

> 1. pin 全称是 pointerIndex，表示第几个手指。(pin < getPointerCount() )
> 2. pos 表示历史数据中的第几个数据。( pos < getHistorySize() )

**注意：**

**1、获取接触面积大小和获取压力大小是需要硬件支持的。**  
**2、非常不幸的是大部分设备所使用的电容屏不支持压力检测，但能够大致检测出接触面积。**  
**3、大部分设备的 `getPressure()` 是使用接触面积来模拟的。**  
**4、由于某些未知的原因(可能系统版本和硬件问题)，某些设备不支持该方法。**

我用不同的设备对这两个方法进行了测试，然而不同设备测试出来的结果不相同，之后经过我多方查证，发现是系统问题，有的设备上只有 `getSize()` 能用，有的设备上只有 `getPressure()` 能用，而有的则两个都不能用。

**由于获取接触面积和获取压力大小受系统和硬件影响，使用的时候一定要进行数据检测，以防因为设备问题而导致程序出错。**



## 鼠标事件

由于触控笔事件和手指事件处理流程大致相同，所以就不讲解了，这里讲解一下与鼠标相关的几个事件：

| 事件                 | 简介                                       |
| ------------------ | ---------------------------------------- |
| ACTION_HOVER_ENTER | 指针移入到窗口或者View区域，但没有按下。                   |
| ACTION_HOVER_MOVE  | 指针在窗口或者View区域移动，但没有按下。                   |
| ACTION_HOVER_EXIT  | 指针移出到窗口或者View区域，但没有按下。                   |
| ACTION_SCROLL      | 滚轮滚动，可以触发水平滚动(AXIS_HSCROLL)或者垂直滚动(AXIS_VSCROLL) |

注意：

1、这些事件类型是 安卓4.0 (API 14) 才添加的。  
2、使用 ` getActionMasked()` 获得这些事件类型。  
3、这些事件不会传递到 [onTouchEvent(MotionEvent)](https://developer.android.com/reference/android/view/View.html#onTouchEvent(android.view.MotionEvent)) 而是传递到 [onGenericMotionEvent(MotionEvent)](https://developer.android.com/reference/android/view/View.html#onGenericMotionEvent(android.view.MotionEvent)) 。



## 输入设备类型判断

输入设备类型判断也是安卓4.0 (API 14) 才添加的，主要包括以下几种设备：

| 设备类型              | 简介   |
| ----------------- | ---- |
| TOOL_TYPE_ERASER  | 橡皮擦  |
| TOOL_TYPE_FINGER  | 手指   |
| TOOL_TYPE_MOUSE   | 鼠标   |
| TOOL_TYPE_STYLUS  | 手写笔  |
| TOOL_TYPE_UNKNOWN | 未知类型 |

**使用 `getToolType(int pointerIndex)` 来获取对应的输入设备类型，pointIndex可以为0，但必须小于 `getPointerCount()`。**



## 总结

虽然本文标题是 MotionEvent 详解，但由于 MotionEvent 实在太庞大了，本文只能涉及一些比较常用的内容，某些不太常用的内容就在以后用到的时候再详细介绍吧，像游戏手柄等输入设备由于我暂时不做游戏开发，也没有过多了解，所以就不介绍给大家啦。

由于个人水平有限，文章中可能会出现错误，如果你觉得哪一部分有错误，或者发现了错别字等内容，欢迎在评论区告诉我，另外，据说关注 [作者微博](http://weibo.com/GcsSloop) 不仅能第一时间收到新文章消息，还能变帅哦。



## 参考资料

[MotionEvent ](https://developer.android.com/reference/android/view/MotionEvent.html)  
[Android MotionEvent详解](http://www.jianshu.com/p/0c863bbde8eb)  



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>



[customview/CoordinateSystem]: http://www.gcssloop.com/customview/CoordinateSystem	"安卓自定义View基础-坐标系"
[customview/CustomViewProcess]: http://www.gcssloop.com/customview/CustomViewProcess	"安卓自定义View进阶-分类与流程"
[customview/dispatch-touchevent-theory]: http://www.gcssloop.com/customview/dispatch-touchevent-theory	"安卓自定义View进阶-事件分发机制原理"
[customview/dispatch-touchevent-source]: http://www.gcssloop.com/customview/dispatch-touchevent-source	"安卓自定义View进阶-事件分发机制详解"
