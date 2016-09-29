# 事件分发机制原理


之前讲解了很多与View绘图相关的知识，你可以在 [安卓自定义View教程目录](http://www.gcssloop.com/customview/CustomViewIndex) 中查看到这些文章，如果你理解了这些文章，那么至少2D绘图部分不是难题了，大部分的需求都能满足，但是关于View还有很多知识点，例如: `让绘图更加炫酷的Paint`，`让View动起来的动画`，`与用户交互的触控事件` 等一系列内容。**本次就带大家简单的了解一下与交互息息相关的东西－事件分发原理**。

> 本次魔法小火车的终点站是事件分发，请各位魔法师带好装备，准备登车启程。

**注意：本文中所有源码分析部分均基于 API23(Android 6.0) 版本，由于安卓系统源码改变很多，可能与之前版本有所不同，但基本流程都是一致的。**



## 为什么要有事件分发机制？

**安卓上面的View是树形结构的，View可能会重叠在一起，当我们点击的地方有多个View都可以响应的时候，这个点击事件应该给谁呢？为了解决这一个问题，就有了事件分发机制。**

如下图，View是一层一层嵌套的，当手指点击 `View1` 的时候，下面的`ViewGroupA`、 `RootView`  等也是能够响应的，为了确定到底应该是哪个View处理这次点击事件，就需要事件分发机制来帮忙。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f87nsnluf5j308c0eamxg.jpg)



## View的结构:

我们的View是树形结构的，在上一个问题中实例View的结构大致如下:

**layout文件:**

```XML
<com.gcssloop.touchevent.test.RootView
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="300dp"
	android:background="#4E5268"
	android:layout_margin="20dp"
	tools:context="com.gcssloop.touchevent.MainActivity">

	<com.gcssloop.touchevent.test.ViewGroupA
		android:background="#95C3FA"
		android:layout_width="200dp"
		android:layout_height="200dp">

		<com.gcssloop.touchevent.test.View1
			android:background="#BDDA66"
			android:layout_width="130dp"
			android:layout_height="130dp"/>

	</com.gcssloop.touchevent.test.ViewGroupA>

	<com.gcssloop.touchevent.test.View2
		android:layout_alignParentRight="true"
		android:background="#BDDA66"
		android:layout_width="80dp"
		android:layout_height="80dp"/>

</com.gcssloop.touchevent.test.RootView>
```



**View结构:**

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f87juodlepj308q09ut8v.jpg)

**可以看到在上面的View结构中莫名多出来的两个东西，`PhoneWindow` 和 `DecorView` ，这两个我们并没有在Layout文件中定义过，但是为什么会存在呢？**

> 仔细观察上面的 layout 文件，你会发现一个问题，我在 layout 文件中的最顶层 View(Group) 的大小并不是填满父窗体的，留下了大量的空白区域，由于我们的手机屏幕不能透明，所以这些空白区域肯定要显示一些东西，那么应该显示什么呢？
>
> 有过安卓开发经验的都知道，屏幕上没有View遮挡的部分会显示主题的颜色。不仅如此，最上面的一个标题栏也没有在 layout 文件中，这个标题栏又是显示在哪里的呢？
>
> **你没有猜错，这个主题颜色和标题栏等内容就是显示在`DecorView`中的。**



**现在知道 `DecorView` 是干什么的了，那么`PhoneWindow` 又有什么作用？**

> 要了解 PhoneWindow 是干啥的，首先要了解啥是 Window ，看官方说明：
>
> Abstract base class for a top-level window look and behavior policy. An instance of this class should be used as the top-level view added to the window manager. It provides standard UI policies such as a background, title area, default key processing, etc.
>
>
> 简单来说，Window是一个抽象类，是所有视图的最顶层容器，视图的外观和行为都归他管，不论是背景显示，标题栏还是事件处理都是他管理的范畴，它其实就像是View界的太上皇(虽然能管的事情看似很多，但是没实权，因为抽象类不能直接使用)。
>
> 而 PhoneWindow 作为 Window 的唯一亲儿子(唯一实现类)，自然就是 View 界的皇帝了，PhoneWindow 的权利可是非常大大，不过对于我们来说用处并不大，因为皇帝平时都是躲在深宫里面的，虽然偶尔用特殊方法能见上一面，但想要完全指挥 PhoneWindow 为你工作是很困难的。
>
> 而上面说的 DecorView 是 PhoneWindow 的一个内部类，其职位相当于小太监，就是跟在 PhoneWindow 身边专业为 PhoneWindow 服务的，除了自己要干活之外，也负责消息的传递，PhoneWindow 的指示通过 DecorView 传递给下面的 View，而下面 View 的信息也通过 DecorView 回传给 PhoneWindow。


## 事件分发、拦截与消费

下表省略了 PhoneWidow 和 DecorView。

> `√` 表示有该方法。
>
> `X` 表示没有该方法。

|  类型  |         相关方法          | Activity | ViewGroup | View |
| :--: | :-------------------: | :------: | :-------: | :--: |
| 事件分发 |  dispatchTouchEvent   |    √     |     √     |  √   |
| 事件拦截 | onInterceptTouchEvent |    X     |     √     |  X   |
| 事件消费 |     onTouchEvent      |    √     |     √     |  √   |


这个三个方法均有一个 boolean(布尔) 类型的返回值，通过返回 true 和 false 来控制事件传递的流程。

PS: 从上表可以看到 Activity 和 View 都是没有事件拦截的，这是因为：

> Activity 作为原始的事件分发者，如果 Activity 拦截了事件会导致整个屏幕都无法响应事件，这肯定不是我们想要的效果。
>
> View最为事件传递的最末端，要么消费掉事件，要么不处理进行回传，根本没必要进行事件拦截。



## 事件分发流程

前面我们了解到了我们的View是树形结构的，基于这样的结构，我们的事件可以进行有序的分发。

事件收集之后最先传递给 Activity， 然后依次向下传递，大致如下：

```
Activity －> PhoneWindow －> DecorView －> ViewGroup －> ... －> View
```

这样的事件分发机制逻辑非常清晰，可是，你是否注意到一个问题？如果最后分发到View，如果这个View也没有处理事件怎么办，就这样让事件浪费掉？

当然不会啦，如果没有任何View消费掉事件，那么这个事件会按照反方向回传，最终传回给Activity，如果最后 Activity 也没有处理，本次事件才会被抛弃:

```
Activity <－ PhoneWindow <－ DecorView <－ ViewGroup <－ ... <－ View
```

**看到这里，我不禁微微一皱眉，这个东西咋看起来那么熟悉呢？再仔细一看，这不就是一个非常经典的[责任链模式](https://zh.wikipedia.org/wiki/%E8%B4%A3%E4%BB%BB%E9%93%BE%E6%A8%A1%E5%BC%8F)吗，** 如果我能处理就拦截下来自己干，如果自己不能处理或者不确定就交给责任链中下一个对象。

**这种设计是非常精巧的，上层View既可以直接拦截该事件，自己处理，也可以先询问(分发给)子View，如果子View需要就交给子View处理，如果子View不需要还能继续交给上层View处理。既保证了事件的有序性，又非常的灵活。在我第一次将这个逻辑弄清楚的时候，看着这样精妙的设计，简直想欢呼庆贺一下。**

其实关于事件传递机制，吴小龙的 [Android事件传递机制分析](http://wuxiaolong.me/2015/12/19/MotionEvent/) 一文中的比喻非常有趣，本文也会借鉴一些其中的内容。

先确定几个角色：

> Activity － 公司大老板
>
> RootView － 项目经理
>
> ViewGroupA － 技术小组长
>
> View1 － 码农小王(公司里唯一的码农)
>
> View2 － 跑龙套的路人甲，无视即可

**PS：由于 PhoneWindow 和 DecorView 我们无法直接操作，以下所有示例均省略了 PhoneWindow 和 DecorView。**



### 1.点击 View1 区域但没有任何 View 消费事件

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f87nsnluf5j308c0eamxg.jpg)

当手指在 `View1` 区域点击了一下之后，如果所有View都不消耗事件，你就能看到一个完整的事件分发流程，大致如下：

> 红色箭头方向表示事件分发方向。
>
> 绿色箭头方向表示事件回传方向。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f88i0q8uozj30nm0kqwhm.jpg)


> **注意: 上图显示分发流程仅仅是一个示意流程，并不代表实际情况，如果按照实际情况绘制，会导致流程图非常复杂和混乱，在纠结了好久之后做了一个艰难的决定，采用这样一个简化后的流程。**
>
> 上面的流程中存在部分不合理内容，请大家选择性接受。
>
> 1. 事件返回时 `dispatchTouchEvent` 直接指向了父View的 `onTouchEvent` 这一部分是不合理的，实际上它仅仅是给了父View的 `dispatchTouchEvent` 一个 false 返回值，父View根据返回值来调用自身的 `onTouchEvent`。
> 2. ViewGroup 是根据 `onInterceptTouchEvent` 的返回值来确定是调用子View的 `dispatchTouchEvent` 还是自身的 `onTouchEvent`， 并没有将调用交给 `onInterceptTouchEvent`。
> 3. ViewGroup 的事件分发机制伪代码如下，可以看出调用的顺序。
>
> ```java
> public boolean dispatchTouchEvent(MotionEvent ev) {
>     boolean result = false;             // 默认状态为没有消费过
>
>     if (!onInterceptTouchEvent(ev)) {   // 如果没有拦截交给子View
>         result = child.dispatchTouchEvent(ev);
>     }
>
>     if (!result) {                      // 如果事件没有被消费,询问自身onTouchEvent
>         result = onTouchEvent(ev);
>     }
>
>     return result;
> }
> ```

**测试:**

> **情景：老板: 我看公司最近业务不咋地，准备发展一下电商业务，下周之前做个淘宝出来试试怎么样。**
>
> **事件顺序，老板(MainActivity)要做淘宝，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到最底层的时候，码农小王(View1)发现做不了，于是消息又一层一层的回传到老板那里。**
>
> 可以看到整个事件传递路线非常有序。从Activity开始，最后回传给Activity结束(由于我们无法操作Phone Window和DecorView，所以没有它们的信息)。

```
MainActivity [老板]: dispatchTouchEvent     经理,我准备发展一下电商业务,下周之前做一个淘宝出来.
RootView     [经理]: dispatchTouchEvent     呼叫技术部,老板要做淘宝,下周上线.
RootView     [经理]: onInterceptTouchEvent  (老板可能疯了,但又不是我做.)
ViewGroupA   [组长]: dispatchTouchEvent     老板要做淘宝,下周上线?
ViewGroupA   [组长]: onInterceptTouchEvent  (看着不太靠谱,先问问小王怎么看)
View1        [码农]: dispatchTouchEvent     做淘宝???
View1        [码农]: onTouchEvent           这个真心做不了啊.
ViewGroupA   [组长]: onTouchEvent           小王说做不了.
RootView     [经理]: onTouchEvent           报告老板, 技术部说做不了.
MainActivity [老板]: onTouchEvent           这么简单都做不了,你们都是干啥的(愤怒).
```



### 2.点击 View1 区域且事件被 View1 消费

如果事件被View1消费掉了则事件会回传告诉上层View这个事件已经被我解决了，上层View就无需再响应了。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f88ll27wv9j30nm0kqtbo.jpg)

> 注意：这张图中的事件回传路径才是正确的路径。

**测试:**

> **情景：老板: 我觉得咱们这个app按钮不好看，做的有光泽一点，要让人有一种想点的欲望。**
>
> **事件顺序，老板(MainActivity)要做改界面，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到最底层的时候，码农小王(View1)就在按钮上添加了一道光(为啥是小王呢？因为公司没有设计师)。**
>
> 可以看出，事件一旦被消费就意味着消息传递的结束，上层View知道了事件已经被消费掉，就不再处理了。

```
MainActivity [老板]: dispatchTouchEvent     把按钮做的好看一点,要有光泽,给人一种点击的欲望.
RootView     [经理]: dispatchTouchEvent     技术部,老板说按钮不好看,要加一道光.
RootView     [经理]: onInterceptTouchEvent  
ViewGroupA   [组长]: dispatchTouchEvent     给按钮加上一道光.
ViewGroupA   [组长]: onInterceptTouchEvent  
View1        [码农]: dispatchTouchEvent     加一道光.
View1        [码农]: onTouchEvent           做好了.
```

> 加一道光：
>
> ![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f88oqj0o4jj304j03paa4.jpg)



### 3.点击 View1 区域但事件被 ViewGroupA 拦截

> 上层的View有权拦截事件，不传递给下层View，例如 ListView 滑动的时候，就不会将事件传递给下层的子 View。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f88p3r45vfj30nm0kqn00.jpg)

> 注意：可以看到，如果上层拦截了事件，下层View将接收不到事件信息。

**测试：**

> **情景：老板: 报告一下项目进度。**
>
> **事件顺序，老板(MainActivity)要知道项目进度，这个事件通过各个部门(ViewGroup)一层一层的往下传，传到技术组组长(ViewGroupA)的时候，组长(ViewGroupA)上报任务即可。无需告知码农小王(View1)。**

```
MainActivity [老板]: dispatchTouchEvent     现在项目做到什么程度了?
RootView     [经理]: dispatchTouchEvent     技术部,你们的app快做完了么?
RootView     [经理]: onInterceptTouchEvent  
ViewGroupA   [组长]: dispatchTouchEvent     项目进度?
ViewGroupA   [组长]: onInterceptTouchEvent  
ViewGroupA   [组长]: onTouchEvent           正在测试,明天就测试完了
```

### 其他情况

事件分发机制设计到到情形非常多，这里就不一一列举了，记住以下几条原则就行了。

* 1.如果事件被消费，就意味着事件信息传递终止。
* 2.如果事件一直没有被消费，最后会传给Activity，如果Activity也不需要就被抛弃。
* 3.判断事件是否被消费是根据返回值，而不是根据你是否使用了事件。


#### [文中测试用的源码下载](https://raw.githubusercontent.com/GcsSloop/AndroidNote/master/CustomView/Demo/dispatchTouchEventDemo.zip)

## 总结

View的事件分发机制实际上就是一个非常经典的责任链模式，如果你了解责任链模式，那么事件分发对你来说并不是什么难题，如果你不了解责任链模式，刚好借此机会学习一下啦。

>  **责任链模式：**
>
>  当有多个对象均可以处理同一请求的时候，将这些对象串联成一条链，并沿着这条链传递改请求，直到有对象处理它为止。

Android 中事件分发机制原理虽然非常简单，但由于实际场景非常复杂，一旦具体到某个场景中变得很麻烦，而本文仅仅是带你简单的了解一下事件分发机制，更详细的内容和具体的一些特殊情形处理会在后续文章中进行讲解。

由于个人水平有限，文章中可能会出现错误，如果你觉得哪一部分有错误，或者发现了错别字等内容，欢迎在评论区告诉我，另外，据说关注[作者微博](http://weibo.com/GcsSloop)不仅能第一时间收到新文章消息，还能变帅哦。



## 参考资料

[Activity](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/app/Activity.java)<br/>
[PhoneWindow](https://android.googlesource.com/platform/frameworks/base/+/696cba573e651b0e4f18a4718627c8ccecb3bda0/policy/src/com/android/internal/policy/impl/PhoneWindow.java)<br/>
[ViewGroup](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/ViewGroup.java)<br/>
[View](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/View.java)<br/>
[Android事件传递机制分析](http://wuxiaolong.me/2015/12/19/MotionEvent/)<br/>
[Android ViewGroup/View 事件分发机制详解](http://anany.me/2015/11/08/touchevent/)<br/>
[ Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463)<br/>
[ Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/sinyu890807/article/details/9153747)<br/>
[更简单的学习Android事件分发](http://www.idtkm.com/customview/customview11/)<br/>
《安卓开发艺术探索》



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

