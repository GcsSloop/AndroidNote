# 特殊控件的事件处理方案

本文带大家了解 Android 特殊形状控件的事件处理方式，主要是利用了 Region 和 Matrix 的一些方法，超级实用的事件处理方案，相信看完本篇之后，任何奇葩控件的事件处理都会变得十分简单。

不得不说，Android 对事件体系封装的非常棒，即便对事件体系不太了解的人，只要简单的调用方法就能使用，而且具有防呆设计，能够保证事件流的完整性和统一性，最大可能性的避免了事件处理的混乱，着实令人佩服。

然而世界上并没有绝对完美的东西，**当【事件处理】遇上【自定义View】，一场好戏就开演了，玩的好叫坐镇军前，指挥千军万马而分毫不乱，玩的不好就是抓耳挠腮，眼见敌人前后包抄而无可奈何。**

## 特殊形状控件

在通常的情况下，自定义 View 直接使用系统的事件体系处理就行，我们也不需要特殊处理，然而当一些特殊的控件出现的时候，麻烦就来了，举个栗子：

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f9t9u0tignj308c08cq3f.jpg)

这是一个在遥控器上非常常见的按键布局，注意中间上下左右选择的部分，看起来十分简单，然而当你真正准备在手机上实现的时候麻烦就出现了。因为所有的View默认都是矩形的，所以事件接收区域也是矩形的，如果直接使用系统提供的 View 来组合出一摸一样的布局也很简单，但点击区域该如何处理？显然这样有部分点击区域是在控件外面的，并且控件之间会产生重叠区域，点击的时候容易判断错误，例如：

> 红色方框表示用 View 组合的情况下，单个 View 的可点击区域。

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f9ta3eymeej308c08cwf1.jpg)

当我们面对这样比较奇特的控件的时候，有很多处理办法，比较投机的一种就是背景贴一个静态图，按钮做成透明的，设置小一点，放在对应的位置，这样可以保证不会误触，当然了如果想要点击效果可以在按钮按下的时候更新一下背景图，这样虽然也可以，但可点击区域会变小，体验效果会变差。设计方案变得非常复杂，而且逻辑也不容易处理，可以说是一种非常糟糕的设计。

当然了，看了我这么多文章的小伙伴应该也猜到我接下来要说什么了，没错，就是自定义 View。当我们面对一些奇葩控件的时候，自定义 View 就变成了一种非常好用的处理方案。

相信小伙伴们看过 [前面的文章][CustomViewIndex] 之后，对各种图形的绘制已经不成问题了，所以我们直接处理重点问题。



## 特殊形状控的点击区域判断

要进行特殊形状的点击判断，要用到一个之前没有使用过的类：Region。

Region 直接翻译的意思是 地域，区域。**在此处应该是区域的意思**。它和 Path 有些类似，但 Path 可以是不封闭图形，而 Region 总是封闭的。可以通过 `setPath` 方法将 Path 转换为 Region。

 **本文中我们重点要使用到的是 Region 中的 `contains` 方法，这个方法可以判断一个点是否包含在该区域内。**

下面是一个简单的示例程序:

```java
public class RegionClickView extends View {
    Paint mPaint;
    Region globalRegion;
    Region circleRegion1;
    Region circleRegion2;
    Path circlePath1;
    Path circlePath2;

    public RegionClickView(Context context) {
        super(context);

        mPaint = new Paint();
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(Color.GRAY);

        // 用Path创建两个圆
        circlePath1 = new Path();
        circlePath2 = new Path();
        circlePath1.addCircle(100, 100, 50, Path.Direction.CW);
        circlePath2.addCircle(400, 400, 50, Path.Direction.CW);

        // 创建 Region
        circleRegion1 = new Region();
        circleRegion2 = new Region();
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        // 将剪裁边界设置为视图大小
        globalRegion = new Region(0, 0, w, h);

        // ▼将 Path 添加到 Region 中
        circleRegion1.setPath(circlePath1, globalRegion);
        circleRegion2.setPath(circlePath2, globalRegion);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                int x = (int) event.getX();
                int y = (int) event.getY();

                // ▼点击区域判断
                if (circleRegion1.contains(x,y)){
                    Toast.makeText(this.getContext(),"圆1被点击",Toast.LENGTH_SHORT).show();
                }
                if (circleRegion2.contains(x,y)){
                    Toast.makeText(this.getContext(),"圆2被点击",Toast.LENGTH_SHORT).show();
                }
                break;
        }
        return super.onTouchEvent(event);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // ▼注意此处将全局变量转化为局部变量，方便 GC 回收 Canvas
        Path circle1 = circlePath1;
        Path circle2 = circlePath2;

        // 绘制两个圆
        canvas.drawPath(circle1,mPaint);
        canvas.drawPath(circle2,mPaint);
    }
}
```

> 代码中比较重要的内容都用 ▼ 符号标记出来了。

上述代码非常简单，就是创建了两个 Path，在 Path 中添加圆形，之后将 Path 设置到 Region 中，当手指在屏幕上按下的时候判断手指按下位置是否在 Region 区域内。

代码整体逻辑非常简单，大家测试一下就行了。



## 画布变换后坐标转换问题

还是上述的例子，绘制一个上下左右选择按键，这个控件是上下左右对称的，熟悉我代码风格的小伙伴都知道，如果遇上这种问题，我肯定是要将坐标系平移到这个控件中心的，这样数据比较好计算，然而进行画布变换操作会产生一个新问题：**手指触摸的坐标系和画布坐标系不统一，就可能引起手指触摸位置和绘制位置不统一。**

举个栗子：

> 在手指按下位置绘制一个圆。

```java
public class CanvasVonvertTouchTest extends CustomView{
    float down_x = -1;
    float down_y = -1;

    public CanvasVonvertTouchTest(Context context) {
        this(context, null);
    }

    public CanvasVonvertTouchTest(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()){
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                down_x = event.getX();
                down_y = event.getY();
                invalidate();
                break;

            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                down_x = down_y = -1;
                invalidate();
                break;
        }

        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        float x = down_x;
        float y = down_y;

        // 绘制触摸坐标系，颜色为灰色，为了能够显示出坐标系，将坐标系位置稍微偏移了一点
        drawTouchCoordinateSpace(canvas);

        canvas.translate(mViewWidth/2, mViewHeight/2);  // 画布平移

        // 绘制平移后的坐标系，颜色为红色
        drawTranslateCoordinateSpace(canvas);

        // 如果没有就返回
        if (x == -1 && y == -1)
            return;

        // 在触摸位置绘制一个小圆
        canvas.drawCircle(x,y,20,mDeafultPaint);
    }

    /**
     *  绘制触摸坐标系，颜色为灰色，为了能够显示出坐标系，将坐标系位置稍微偏移了一点
     */
    private void drawTouchCoordinateSpace(Canvas canvas) {
        canvas.save();
        canvas.translate(10,10);
        CanvasAidUtils.set2DAxisLength(1000, 0, 1400, 0);
        CanvasAidUtils.setLineColor(Color.GRAY);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
        canvas.restore();
    }

    /**
     * 绘制平移后的坐标系，颜色为红色
     */
    private void drawTranslateCoordinateSpace(Canvas canvas) {
        CanvasAidUtils.set2DAxisLength(500, 500, 700, 700);
        CanvasAidUtils.setLineColor(Color.RED);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
    }
}
```

可以看到，直接拿手指触摸位置的坐标来绘制会导致绘制位置不正确，大概会像这样子：

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f9tdc3p84uj308c0d4mx9.jpg)

两者坐标是相同的，但是由于坐标系不同，导致实际显示位置不同。

**那么问题来了，我们在之前的文章中讲过，映射不同坐标系的坐标用 什么来着？**  
**是 Matrix。**

如果看过我之前的文章但没有想起来的说明你们根本没有认真看，全部拖出去糟蹋 5 分钟！  
没看过的点 [Matrix原理][Matrix_Basic] 和  [Matrix详解][Matrix_Method] 。

> **Matrix 是一个矩阵，主要功能是坐标映射，数值转换。**

那么接下来我们就对上面的示例进行简单的改造一下，让触摸位置和实际绘制绘制重合。

**注意：比较重要的修改位置用▼标记出来了。**

```java
public class CanvasVonvertTouchTest extends CustomView{
    float down_x = -1;
    float down_y = -1;

    public CanvasVonvertTouchTest(Context context) {
        this(context, null);
    }

    public CanvasVonvertTouchTest(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getActionMasked()){
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                // ▼ 注意此处使用 getRawX，而不是 getX
                down_x = event.getRawX();
                down_y = event.getRawY();
                invalidate();
                break;

            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                down_x = down_y = -1;
                invalidate();
                break;
        }

        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        float[] pts = {down_x, down_y};

        // 绘制触摸坐标系，颜色为灰色，为了能够显示出坐标系，将坐标系位置稍微偏移了一点
        drawTouchCoordinateSpace(canvas);

        canvas.translate(mViewWidth/2, mViewHeight/2);  // 画布平移

        // 绘制平移后的坐标系，颜色为红色
        drawTranslateCoordinateSpace(canvas);

        // 如果没有就返回
        if (pts[0] == -1 && pts[1] == -1)
            return;

        // ▼ 获得当前矩阵的逆矩阵
        Matrix invertMatrix = new Matrix();
        canvas.getMatrix().invert(invertMatrix);

        // ▼ 使用 mapPoints 将触摸位置转换为画布坐标
        invertMatrix.mapPoints(pts);

        // 在触摸位置绘制一个小圆
        canvas.drawCircle(pts[0],pts[1],20,mDeafultPaint);
    }

    /**
     *  绘制触摸坐标系，颜色为灰色，为了能够显示出坐标系，将坐标系位置稍微偏移了一点
     */
    private void drawTouchCoordinateSpace(Canvas canvas) {
        canvas.save();
        canvas.translate(10,10);
        CanvasAidUtils.set2DAxisLength(1000, 0, 1400, 0);
        CanvasAidUtils.setLineColor(Color.GRAY);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
        canvas.restore();
    }

    /**
     * 绘制平移后的坐标系，颜色为红色
     */
    private void drawTranslateCoordinateSpace(Canvas canvas) {
        CanvasAidUtils.set2DAxisLength(500, 500, 700, 700);
        CanvasAidUtils.setLineColor(Color.RED);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
        CanvasAidUtils.draw2DCoordinateSpace(canvas);
    }
}

```

**绘制结果：**

小白点和黑色的圆没有完全重合是因为系统显示触摸位置的绘制逻辑和我使用的绘制逻辑不太相同，两者的位置是一样的。

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f9te2mzxcvj308c0d40st.jpg)

其实核心部分就这两点：

```java
// ▼ 注意此处使用 getRawX，而不是 getX
down_x = event.getRawX();
down_y = event.getRawY();

// -------------------------------------

// ▼ 获得当前矩阵的逆矩阵
Matrix invertMatrix = new Matrix();
canvas.getMatrix().invert(invertMatrix);

// ▼ 使用 mapPoints 将触摸位置转换为画布坐标
invertMatrix.mapPoints(pts);
```

1. 使用全局坐标系
2. 使用逆矩阵的 mapPoints

**原理嘛，其实非常简单，我们在画布上正常的绘制，需要将画布坐标系转换为全局坐标系后才能真正的绘制内容。所以我们反着来，将获得到的全局坐标系坐标使用当前画布的逆矩阵转化一下，就转化为当前画布的坐标系坐标了，如果对 [Matrix原理][Matrix_Basic] 和  [Matrix详解][Matrix_Method]  理解了，即便我不说你们也肯定会想到这个方案的。**

## 仿遥控器按钮代码示例

在解决了上述两大难题之后，相信不论形状如何奇葩的自定义控件，基本上都难不倒大家了，最后用一个简单的示例作为结尾，还是文章开头所举的例子，核心内容就是上面讲的两个东西。

```java
public class RemoteControlMenu extends CustomView {
    Path up_p, down_p, left_p, right_p, center_p;
    Region up, down, left, right, center;

    Matrix mMapMatrix = null;

    int CENTER = 0;
    int UP = 1;
    int RIGHT = 2;
    int DOWN = 3;
    int LEFT = 4;
    int touchFlag = -1;
    int currentFlag = -1;

    MenuListener mListener = null;

    int mDefauColor = 0xFF4E5268;
    int mTouchedColor = 0xFFDF9C81;


    public RemoteControlMenu(Context context) {
        this(context, null);
    }

    public RemoteControlMenu(Context context, AttributeSet attrs) {
        super(context, attrs);

        up_p = new Path();
        down_p = new Path();
        left_p = new Path();
        right_p = new Path();
        center_p = new Path();

        up = new Region();
        down = new Region();
        left = new Region();
        right = new Region();
        center = new Region();

        mDeafultPaint.setColor(mDefauColor);
      	mDeafultPaint.setAntiAlias(true);

        mMapMatrix = new Matrix();

    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mMapMatrix.reset();
		
      	// 注意这个区域的大小
        Region globalRegion = new Region(-w, -h, w, h);
        int minWidth = w > h ? h : w;
        minWidth *= 0.8;

        int br = minWidth / 2;
        RectF bigCircle = new RectF(-br, -br, br, br);

        int sr = minWidth / 4;
        RectF smallCircle = new RectF(-sr, -sr, sr, sr);

        float bigSweepAngle = 84;
        float smallSweepAngle = -80;

        // 根据视图大小，初始化 Path 和 Region
        center_p.addCircle(0, 0, 0.2f * minWidth, Path.Direction.CW);
        center.setPath(center_p, globalRegion);

        right_p.addArc(bigCircle, -40, bigSweepAngle);
        right_p.arcTo(smallCircle, 40, smallSweepAngle);
        right_p.close();
        right.setPath(right_p, globalRegion);

        down_p.addArc(bigCircle, 50, bigSweepAngle);
        down_p.arcTo(smallCircle, 130, smallSweepAngle);
        down_p.close();
        down.setPath(down_p, globalRegion);

        left_p.addArc(bigCircle, 140, bigSweepAngle);
        left_p.arcTo(smallCircle, 220, smallSweepAngle);
        left_p.close();
        left.setPath(left_p, globalRegion);

        up_p.addArc(bigCircle, 230, bigSweepAngle);
        up_p.arcTo(smallCircle, 310, smallSweepAngle);
        up_p.close();
        up.setPath(up_p, globalRegion);

    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        float[] pts = new float[2];
        pts[0] = event.getRawX();
        pts[1] = event.getRawY();
        mMapMatrix.mapPoints(pts);

        int x = (int) pts[0];
        int y = (int) pts[1];

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                touchFlag = getTouchedPath(x, y);
                currentFlag = touchFlag;
                break;
            case MotionEvent.ACTION_MOVE:
                currentFlag = getTouchedPath(x, y);
                break;
            case MotionEvent.ACTION_UP:
                currentFlag = getTouchedPath(x, y);
            	// 如果手指按下区域和抬起区域相同且不为空，则判断点击事件
                if (currentFlag == touchFlag && currentFlag != -1 && mListener != null) {
                    if (currentFlag == CENTER) {
                        mListener.onCenterCliched();
                    } else if (currentFlag == UP) {
                        mListener.onUpCliched();
                    } else if (currentFlag == RIGHT) {
                        mListener.onRightCliched();
                    } else if (currentFlag == DOWN) {
                        mListener.onDownCliched();
                    } else if (currentFlag == LEFT) {
                        mListener.onLeftCliched();
                    }
                }
                touchFlag = currentFlag = -1;
                break;
            case MotionEvent.ACTION_CANCEL:
                touchFlag = currentFlag = -1;
                break;
        }

        invalidate();
        return true;
    }

  	// 获取当前触摸点在哪个区域
    int getTouchedPath(int x, int y) {
        if (center.contains(x, y)) {
            return 0;
        } else if (up.contains(x, y)) {
            return 1;
        } else if (right.contains(x, y)) {
            return 2;
        } else if (down.contains(x, y)) {
            return 3;
        } else if (left.contains(x, y)) {
            return 4;
        }
        return -1;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.translate(mViewWidth / 2, mViewHeight / 2);

        // 获取测量矩阵(逆矩阵)
        if (mMapMatrix.isIdentity()) {
            canvas.getMatrix().invert(mMapMatrix);
        }

      	// 绘制默认颜色
        canvas.drawPath(center_p, mDeafultPaint);
        canvas.drawPath(up_p, mDeafultPaint);
        canvas.drawPath(right_p, mDeafultPaint);
        canvas.drawPath(down_p, mDeafultPaint);
        canvas.drawPath(left_p, mDeafultPaint);

      	// 绘制触摸区域颜色
        mDeafultPaint.setColor(mTouchedColor);
        if (currentFlag == CENTER) {
            canvas.drawPath(center_p, mDeafultPaint);
        } else if (currentFlag == UP) {
            canvas.drawPath(up_p, mDeafultPaint);
        } else if (currentFlag == RIGHT) {
            canvas.drawPath(right_p, mDeafultPaint);
        } else if (currentFlag == DOWN) {
            canvas.drawPath(down_p, mDeafultPaint);
        } else if (currentFlag == LEFT) {
            canvas.drawPath(left_p, mDeafultPaint);
        }
        mDeafultPaint.setColor(mDefauColor);
    }

    public void setListener(MenuListener listener) {
        mListener = listener;
    }

   	// 点击事件监听器
    public interface MenuListener {
        void onCenterCliched();

        void onUpCliched();

        void onRightCliched();

        void onDownCliched();

        void onLeftCliched();
    }
}
```

**运行效果:**

当手指在某一区域活动时，该区域会高亮显示，如果注册了监听器，点击某一区域会触发监听器回调。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9tinrk6ilj308c0eat92.jpg)



## 总结

本文虽然代码比较多，但核心概念非常简单，主要涉及以下两点：

1. Region 的区域检测。
2. Matrix 的坐标映射。

**这两个知识点都不是很难，然而灵活运用起来却是非常强大的，如果有对 Matrix 不了解的小伙伴，推荐去看我 [之前的文章][CustomViewIndex]，里面有关于Matrix的详细介绍，**



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>



[CustomViewIndex]: http://www.gcssloop.com/customview/CustomViewIndex
[Matrix_Basic]: http://www.gcssloop.com/customview/Matrix_Basic
[Matrix_Method]: http://www.gcssloop.com/customview/Matrix_Method



