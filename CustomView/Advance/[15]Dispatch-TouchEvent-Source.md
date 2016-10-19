# 事件分发机制详解

在上一篇文章 [事件分发机制原理][dispatch-touchevent-theory] 中简要分析了一下事件分发机制的原理，原理是十分简单的，一句话就能总结：**责任链模式，事件层层传递，直到被消费。** 虽然原理简单，但是随着 Android 不断的发展，实际运用场景也越来越复杂，所以想要彻底玩转事件分发机制还需要一定技巧，本篇事件分发机制详解将带大家了解 ...

> **你以为我接下来要讲源码？**  
> 我就不按套路，所有的源码都是为了适应具体的应用场景而写的，只要能够理解运用场景，理解源码也就十分简单了。所以本篇的核心问题是：**正确理解在实际场景中事件分发机制的作用。** 会涉及到源码，但不是主角。

**注意：本文中所有源码分析部分均基于 API23(Android 6.0) 版本，由于安卓系统源码改变很多，可能与之前版本有所不同，但基本流程都是一致的。**



## 常见事件

既然是事件分发，总要有事件才能分发吧，所以我们先了解一下常见的几种事件。

根据面向对象思想，事件被封装成 MotionEvent 对象，由于本篇重点不在于此，所以只会涉及到几个与手指触摸相关的常见事件:

| 事件            | 简介                        |
| ------------- | ------------------------- |
| ACTION_DOWN   | 手指 **初次接触到屏幕** 时触发。       |
| ACTION_MOVE   | 手指 **在屏幕上滑动** 时触发，会会多次触发。 |
| ACTION_UP     | 手指 **离开屏幕** 时触发。          |
| ACTION_CANCEL | 事件 **被上层拦截** 时触发。         |

对于单指触控来说，一次简单的交互流程是这样的:

**手指落下(ACTION_DOWN) －> 移动(ACTION_MOVE) －> 离开(ACTION_UP)**  

> * 本次事例中 ACTION_MOVE 有多次触发。
> * 如果仅仅是单击(手指按下再抬起)，不会触发 ACTION_MOVE。

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f8oz1704ylg30bo0jqgmx.gif)

## 事件分发、拦截与消费

关于这一部分内容，上一篇文章 [事件分发机制原理][dispatch-touchevent-theory] 已经将流程整理的比较清楚了，本文会深入细节来研究这些内容。之所以分开讲，是为了防止大家被细节所迷惑而忽略了整体逻辑。

> `√` 表示有该方法。
>
> `X` 表示没有该方法。

|  类型  |         相关方法          | ViewGroup | View |
| :--: | :-------------------: | :-------: | :--: |
| 事件分发 |  dispatchTouchEvent   |     √     |  √   |
| 事件拦截 | onInterceptTouchEvent |     √     |  X   |
| 事件消费 |     onTouchEvent      |     √     |  √   |



### View 相关

`dispatchTouchEvent` 是事件分发机制中的核心，所有的事件调度都归它管。不过我细看表格， ViewGroup 有 dispatchTouchEvent 也就算了，毕竟人家有一堆 ChildView 需要管理，但为啥 View 也有？这就引出了我们的第一个疑问。

#### Q: 为什么 View 会有 dispatchTouchEvent ?

A: 我们知道 View 可以注册很多事件监听器，例如：单击事件(onClick)、长按事件(onLongClick)、触摸事件(onTouch)，并且View自身也有 onTouchEvent 方法，那么问题来了，这么多与事件相关的方法应该由谁管理？毋庸置疑就是 `dispatchTouchEvent`，所以 View 也会有事件分发。

相信看到这里很多小伙伴会产生第二个疑问，View 有这么多事件监听器，到底哪个先执行？

#### Q: 与 View 事件相关的各个方法调用顺序是怎样的？

A: **如果不去看源码，想一下让自己设计会怎样？**

* 单击事件(onClickListener) 需要两个两个事件(ACTION_DOWN 和 ACTION_UP )才能触发，如果先分配给onClick判断，等它判断完，用户手指已经离开屏幕，黄花菜都凉了，定然造成 View 无法响应其他事件，应该最后调用。(最后)
* 长按事件(onLongClickListener) 同理，也是需要长时间等待才能出结果，肯定不能排到前面，但因为不需要ACTION_UP，应该排在 onClick 前面。(onLongClickListener > onClickListener)
* 触摸事件(onTouchListener) 如果用户注册了触摸事件，说明用户要自己处理触摸事件了，这个应该排在最前面。(最前)
* View自身处理(onTouchEvent) 提供了一种默认的处理方式，如果用户已经处理好了，也就不需要了，所以应该排在 onTouchListener 后面。(onTouchListener > onTouchEvent)

**所以事件的调度顺序应该是 `onTouchListener > onTouchEvent > onLongClickListener > onClickListener`**。

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f8r8jg9mw5j308y07mglw.jpg)

下面我们来看一下实际测试结果:

> 手指按下，不移动，稍等片刻再抬起。

```shell
[Listener ]: onTouchListener      ACTION_DOWN
[GcsView  ]: onTouchEvent         ACTION_DOWN
[Listener ]: onLongClickListener  
[Listener ]: onTouchListener      ACTION_UP
[GcsView  ]: onTouchEvent         ACTION_UP
[Listener ]: onClickListener      
```

可以看到，测试结果也支持我们猜测的结论，因为长按 onLongClickListener 不需要 ACTION_UP 所以会在 ACTION_DOWN 之后就触发。

接下来就看一下源码是怎么设计的(省略了大量无关代码)：

```java
public boolean dispatchTouchEvent(MotionEvent event) {
    ...
    boolean result = false;	// result 为返回值，主要作用是告诉调用者事件是否已经被消费。
    if (onFilterTouchEventForSecurity(event)) {
        ListenerInfo li = mListenerInfo;
        /** 
         * 如果设置了OnTouchListener，并且当前 View 可点击，就调用监听器的 onTouch 方法，
         * 如果 onTouch 方法返回值为 true，就设置 result 为 true。
         */
        if (li != null && li.mOnTouchListener != null
                && (mViewFlags & ENABLED_MASK) == ENABLED
                && li.mOnTouchListener.onTouch(this, event)) {
            result = true;
        }
      
        /** 
         * 如果 result 为 false，则调用自身的 onTouchEvent。
         * 如果 onTouchEvent 返回值为 true，则设置 result 为 true。
         */
        if (!result && onTouchEvent(event)) {
            result = true;
        }
    }
    ...
    return result;
}
```

> **如果觉得源码还是太长，那么用伪代码实现应当是这样的(省略若干安全判断)，简单粗暴:**
>
> ```java
> public boolean dispatchTouchEvent(MotionEvent event) {
>   if (mOnTouchListener.onTouch(this, event)) {
>       return true;
>   } else if (onTouchEvent(event)) {
>       return true;
>   }
>   return false;
> }
> ```

正当你沉迷在源码的"精妙"逻辑的时候，你可能没发现有两个东西失踪了，等回过神来，定睛一看，哎呦妈呀，**OnClick 和 OnLongClick 去哪里了？**

不要担心，OnClick 和 OnLongClick 的具体调用位置在 **onTouchEvent** 中，看源码(同样省略大量无关代码):

```java
public boolean onTouchEvent(MotionEvent event) {
    ...
    final int action = event.getAction();
  	// 检查各种 clickable
    if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
            (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
        switch (action) {
            case MotionEvent.ACTION_UP:
                ...
                removeLongPressCallback();  // 移除长按
                ...
                performClick();             // 检查单击
                ...
                break;
            case MotionEvent.ACTION_DOWN:
                ...
                checkForLongClick(0);       // 检测长按
                ...
                break;
            ...
        }
        return true;                        // ◀︎表示事件被消费
    }
    return false;
}
```

> **注意了，第一个重点要出现了(敲黑板)!**
>
> ![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f8p431ehi3j304w04wmx1.jpg)
>
> **注意上面代码中存在一个 `return true;` 并且是只要 View 可点击就返回 true，就表示事件被消费了。**
>
> 举个栗子: I have a **RelativeLayout**，I have a **View**，Ugh，**RelativeLayout - View** 
>
> ```xml
> <RelativeLayout
>     android:background="#CCC"
>     android:id="@+id/layout"
>     android:onClick="myClick"
>     android:layout_width="200dp"
>     android:layout_height="200dp">
>     <View
>         android:clickable="true"
>         android:layout_width="200dp"
>         android:layout_height="200dp" />
> </RelativeLayout>
> ```
>
> 现在你有了一个 **RelativeLayout - View** 你开开心心的为 RelativeLayout 设置了一个点击事件`myClick`，然而你会发现不论怎么点都不会接收到信息，仔细一看，发现内部的 View 有一个属性 `android:clickable="true"` 正是这个看似不起眼的属性把事件给消费掉了，由此我们可以得出如下结论:  
> **1. 不论 View 自身是否注册点击事件，只要 View 是可点击的就会消费事件。**  
> **2. 事件是否被消费由返回值决定，true 表示消费，false 表示不消费，与是否使用了事件无关。**

关于 View 的事件分发先说这么多，下面我们来看一下 ViewGroup 的事件分发。



### ViewGroup 相关

**ViewGroup(通常是各种Layout) 的事件分发相对来说就要麻烦一些，因为 ViewGroup 不仅要考虑自身，还要考虑各种 ChildView，一旦处理不好就容易引起各种事件冲突，正所谓养儿方知父母难啊。**

#### VIewGroup 的事件分发流程又是如何的呢？

 上一篇文章 [事件分发机制原理][dispatch-touchevent-theory] 中我们了解到事件是通过ViewGroup一层一层传递的，最终传递给 View，ViewGroup 要比它的 ChildView 先拿到事件，并且有权决定是否告诉要告诉 ChildView。在默认的情况下 ViewGroup 事件分发流程是这样的。

* 1.判断自身是否需要(询问 onInterceptTouchEvent 是否拦截)，如果需要，调用自己的 onTouchEvent。
* 2.自身不需要或者不确定，则询问 ChildView ，一般来说是调用手指触摸位置的 ChildView。
* 3.如果子 ChildView 不需要则调用自身的 onTouchEvent。


用伪代码应该是这样的:

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    boolean result = false;             // 默认状态为没有消费过

    if (!onInterceptTouchEvent(ev)) {   // 如果没有拦截交给子View
        result = child.dispatchTouchEvent(ev);
    }

    if (!result) {                      // 如果事件没有被消费,询问自身onTouchEvent
        result = onTouchEvent(ev);
    }

    return result;
}
```

**有人看到这里可能会有疑问，我看过源码，ViewGroup 的 `dispatchTouchEvent` 可有二百多行呢，你弄这几行就想忽悠我，别以为我读书少。**

当然了，上述源码是不完善的，还有很多问题是没有解决的，例如:

##### 1. ViewGroup 中可能有多个 ChildView，如何判断应该分配给哪一个？

这个很容易，就是把所有的 ChildView 遍历一遍，如果手指触摸的点在 ChildView 区域内就分发给这个View。

##### 2. 当该点的 ChildView 有重叠时应该如何分配？

当 ChildView 重叠时，**一般会分配给显示在最上面的 ChildView**。  
如何判断哪个是显示在最上面的呢？后面加载的一般会覆盖掉之前的，所以**显示在最上面的是最后加载的**。

如下:

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools" 
    android:id="@+id/activity_main"
    android:layout_width="match_parent" 
    android:layout_height="match_parent"
    tools:context="com.gcssloop.viewtest.MainActivity">
    <View
        android:id="@+id/view1"
        android:background="#E4A07B"
        android:layout_width="200dp"
        android:layout_height="200dp"/>
    <View
        android:id="@+id/view2"
        android:layout_margin="100dp"
        android:background="#BDDA66"
        android:layout_width="200dp"
        android:layout_height="200dp"/>
</RelativeLayout>
```

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f8r0i301sgj308w0fmwez.jpg)

当手指点击有重叠区域时，分如下几种情况:

1. 只有 View1 可点击时，事件将会分配给 View1，即使被 View2 遮挡，这一部分仍是 View1 的可点击区域。
2. 只有 View2 可点击时，事件将会分配给 View2。
3. View1 和 View2 均可点击时，事件会分配给后加载的 View2，View2 将事件消费掉，View1接收不到事件。

**注意:** 

* 上面说的是可点击，可点击包括很多种情况，只要你给View注册了 `onClickListener、onLongClickListener、OnContextClickListener` 其中的任何一个监听器或者设置了 `android:clickable="true"` 就代表这个 View 是可点击的。  
  另外，某些 View 默认就是可点击的，例如，Button，CheckBox 等。
* 给 View 注册 `OnTouchListener` 不会影响 View 的可点击状态。即使给 View 注册 `OnTouchListener` ，**只要不返回 `true` 就不会消费事件**。


##### 3. ViewGroup 和 ChildView 同时注册了事件监听器(onClick等)，哪个会执行?

事件优先给 ChildView，会被 ChildView消费掉，ViewGroup 不会响应。

##### 4. 所有事件都应该被同一 View 消费

在上面的例子中我们分析后可以了解到，同一次点击事件只能被一个 View 消费，这是为什呢？主要是为了防止事件响应混乱，如果再一次完整的事件中分别将不同的事件分配给了不同的 View 容易造成事件响应混乱。

> ( View 中 onClick 事件需要同时接收到 ACTION_DOWN 和 ACTION_UP 才能触发，如果分配给了不同的 View，那么 onClick 将无法被正确触发)。

**安卓为了保证所有的事件都是被一个 View 消费的，对第一次的事件( ACTION_DOWN )进行了特殊判断，View 只有消费了 ACTION_DOWN 事件，才能接收到后续的事件(可点击控件会默认消费所有事件)，并且会将后续所有事件传递过来，不会再传递给其他 View，除非上层 View 进行了拦截。**  
**如果上层 View 拦截了当前正在处理的事件，会收到一个 ACTION_CANCEL，表示当前事件已经结束，后续事件不会再传递过来。**



**源码:**

> 其实如果能够理解上面的内容，不看源码也能非常顺利的使用事件分发，但源码中能挖掘出更多的内容。

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
  	// 调试用
    if (mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
    }

  	// 判断事件是否是针对可访问的焦点视图(很晚才添加的内容，个人猜测和屏幕辅助相关，方便盲人等使用设备)
    if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
        ev.setTargetAccessibilityFocus(false);
    }

    boolean handled = false;
    if (onFilterTouchEventForSecurity(ev)) {
        final int action = ev.getAction();
        final int actionMasked = action & MotionEvent.ACTION_MASK;

        // 处理第一次ACTION_DOWN.
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // 清除之前所有的状态
            cancelAndClearTouchTargets(ev);
            resetTouchState();
        }

        // 检查是否需要拦截.
        final boolean intercepted;
        if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {
            final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
            if (!disallowIntercept) {
                intercepted = onInterceptTouchEvent(ev);	// 询问是否拦截
                ev.setAction(action); 						// 恢复操作，防止被更改
            } else {
                intercepted = false;
            }
        } else {
          	// 没有目标来处理该事件，而且也不是一个新的事件事件(ACTION_DOWN), 进行拦截。
            intercepted = true;
        }

      	// 判断事件是否是针对可访问的焦点视图
        if (intercepted || mFirstTouchTarget != null) {
            ev.setTargetAccessibilityFocus(false);
        }

        // 检查事件是否被取消(ACTION_CANCEL).
        final boolean canceled = resetCancelNextUpFlag(this)
                || actionMasked == MotionEvent.ACTION_CANCEL;

        final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
        TouchTarget newTouchTarget = null;
        boolean alreadyDispatchedToNewTouchTarget = false;
      	
      	// 如果没有取消也没有被拦截	(进入事件分发)
        if (!canceled && !intercepted) {

            // 如果事件是针对可访问性焦点视图，我们将其提供给具有可访问性焦点的视图。
          	// 如果它不处理它，我们清除该标志并像往常一样将事件分派给所有的 ChildView。 
            // 我们检测并避免保持这种状态，因为这些事非常罕见。
            View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                    ? findChildWithAccessibilityFocus() : null;

            if (actionMasked == MotionEvent.ACTION_DOWN
                    || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                        : TouchTarget.ALL_POINTER_IDS;

                // 清除此指针ID的早期触摸目标，防止不同步。
                removePointersFromTouchTargets(idBitsToAssign);

                final int childrenCount = mChildrenCount;
                if (newTouchTarget == null && childrenCount != 0) {
                    final float x = ev.getX(actionIndex);	// 获取触摸位置坐标
                    final float y = ev.getY(actionIndex);
                    // 查找可以接受事件的 ChildView
                    final ArrayList<View> preorderedList = buildOrderedChildList();
                    final boolean customOrder = preorderedList == null
                            && isChildrenDrawingOrderEnabled();
                    final View[] children = mChildren;
                  	// ▼注意，从最后向前扫描
                    for (int i = childrenCount - 1; i >= 0; i--) {
                        final int childIndex = customOrder
                                ? getChildDrawingOrder(childrenCount, i) : i;
                        final View child = (preorderedList == null)
                                ? children[childIndex] : preorderedList.get(childIndex);

                        // 如果有一个视图具有可访问性焦点，我们希望它首先获取事件，
                      	// 如果不处理，我们将执行正常的分派。 
                      	// 尽管这可能会分发两次，但它能保证在给定的时间内更安全的执行。
                        if (childWithAccessibilityFocus != null) {
                            if (childWithAccessibilityFocus != child) {
                                continue;
                            }
                            childWithAccessibilityFocus = null;
                            i = childrenCount - 1;
                        }

                      	// 检查View是否允许接受事件(即处于显示状态(VISIBLE)或者正在播放动画)
                      	// 检查触摸位置是否在View区域内
                        if (!canViewReceivePointerEvents(child)
                                || !isTransformedTouchPointInView(x, y, child, null)) {
                            ev.setTargetAccessibilityFocus(false);
                            continue;
                        }

                      	// getTouchTarget 中判断了 child 是否包含在 mFirstTouchTarget 中
                      	// 如果有返回 target，如果没有返回 null 
                        newTouchTarget = getTouchTarget(child);
                        if (newTouchTarget != null) {
                            // ChildView 已经准备好接受在其区域内的事件。
                            newTouchTarget.pointerIdBits |= idBitsToAssign;
                            break;	// ◀︎已经找到目标View，跳出循环
                        }

                        resetCancelNextUpFlag(child);
                        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                            mLastTouchDownTime = ev.getDownTime();
                            if (preorderedList != null) {
                                for (int j = 0; j < childrenCount; j++) {
                                    if (children[childIndex] == mChildren[j]) {
                                        mLastTouchDownIndex = j;
                                        break;
                                    }
                                }
                            } else {
                                mLastTouchDownIndex = childIndex;
                            }
                            mLastTouchDownX = ev.getX();
                            mLastTouchDownY = ev.getY();
                            newTouchTarget = addTouchTarget(child, idBitsToAssign);
                            alreadyDispatchedToNewTouchTarget = true;
                            break;
                        }
                      
                        ev.setTargetAccessibilityFocus(false);
                    }
                    if (preorderedList != null) preorderedList.clear();
                }

                if (newTouchTarget == null && mFirstTouchTarget != null) {
                    // 没有找到 ChildView 接收事件
                    newTouchTarget = mFirstTouchTarget;
                    while (newTouchTarget.next != null) {
                        newTouchTarget = newTouchTarget.next;
                    }
                    newTouchTarget.pointerIdBits |= idBitsToAssign;
                }
            }
        }

        // 分发 TouchTarget
        if (mFirstTouchTarget == null) {
            // 没有 TouchTarget，将当前 ViewGroup 当作普通的 View 处理。
            handled = dispatchTransformedTouchEvent(ev, canceled, null,
                    TouchTarget.ALL_POINTER_IDS);
        } else {
            // 分发TouchTarget，如果我们已经分发过，则避免分配给新的目标。 
          	// 如有必要，取消分发。
            TouchTarget predecessor = null;
            TouchTarget target = mFirstTouchTarget;
            while (target != null) {
                final TouchTarget next = target.next;
                if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                    handled = true;
                } else {
                    final boolean cancelChild = resetCancelNextUpFlag(target.child)
                            || intercepted;
                    if (dispatchTransformedTouchEvent(ev, cancelChild,
                            target.child, target.pointerIdBits)) {
                        handled = true;
                    }
                    if (cancelChild) {
                        if (predecessor == null) {
                            mFirstTouchTarget = next;
                        } else {
                            predecessor.next = next;
                        }
                        target.recycle();
                        target = next;
                        continue;
                    }
                }
                predecessor = target;
                target = next;
            }
        }

        // 如果需要，更新指针的触摸目标列表或取消。
        if (canceled
                || actionMasked == MotionEvent.ACTION_UP
                || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
            resetTouchState();
        } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
            final int actionIndex = ev.getActionIndex();
            final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
            removePointersFromTouchTargets(idBitsToRemove);
        }
    }

    if (!handled && mInputEventConsistencyVerifier != null) {
        mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
    }
    return handled;
}
```



## 核心要点

1. **事件分发原理: 责任链模式，事件层层传递，直到被消费。**
2. **View 的 `dispatchTouchEvent` 主要用于调度自身的监听器和 onTouchEvent。**
3. **View的事件的调度顺序是 onTouchListener > onTouchEvent > onLongClickListener > onClickListener 。**
4. **不论 View 自身是否注册点击事件，只要 View 是可点击的就会消费事件。**
5. **事件是否被消费由返回值决定，true 表示消费，false 表示不消费，与是否使用了事件无关。**
6. **ViewGroup 中可能有多个 ChildView 时，将事件分配给包含点击位置的 ChildView。**
7. **ViewGroup 和 ChildView 同时注册了事件监听器(onClick等)，由 ChildView 消费。**
8. **一次触摸流程中产生事件应被同一 View 消费，全部接收或者全部拒绝。**
9. **只要接受 ACTION_DOWN 就意味着接受所有的事件，拒绝 ACTION_DOWN 则不会收到后续内容。**
10. **如果当前正在处理的事件被上层 View 拦截，会收到一个 ACTION_CANCEL，后续事件不会再传递过来**。



## 总结

本文啰嗦了这么多内容，但真正需要注意的就是核心要点中的几个概念，只要能正确理解这些概念，相信理解事件分发机制将再也不是难题。

> 最后，个人推荐阅读源码的方法，先尝试用自己的角度去分析，建立概念，然后看源码进行验证、对比，如果发现自己建立的概念有问题，就尝试修正自己的概念，这样比较容易理解原作者的意图，也不容易被众多的代码所迷惑。
>
> 就像 ViewGroup 中的 dispatchTouchEvent 内容非常多，主要是为了应对实际的场景，里面有很多 安全判断，处理多指触控 等内容，这些如果不先建立概念就去看源码很容易被这些细节问题所迷惑。



## 参考资料

[View ](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/View.java)  
[ViewGroup.java](https://android.googlesource.com/platform/frameworks/base/+/master/core/java/android/view/ViewGroup.java)  
[Android Touch事件分发详解](https://github.com/CharonChui/AndroidNote/blob/master/Android%E5%8A%A0%E5%BC%BA/Android%20Touch%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E8%AF%A6%E8%A7%A3.md)  
[基于源码来了解Android的事件分发机制](http://minjie.tech/2016/09/03/%E5%9F%BA%E4%BA%8E%E6%BA%90%E7%A0%81%E6%9D%A5%E4%BA%86%E8%A7%A3Android%E7%9A%84%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6/)



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>



[dispatch-touchevent-theory]: http://www.gcssloop.com/customview/dispatch-touchevent-theory	"事件分发机制原理－GcsSloop"

