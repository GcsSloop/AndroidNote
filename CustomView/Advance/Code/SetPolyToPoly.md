# Matrix setPolyTOPoly 测试代码

## SetPolyToPoly.java

[下载代码 (右键 -> 另存为)](https://raw.githubusercontent.com/GcsSloop/AndroidNote/master/CustomView/Advance/Code/SetPolyToPoly.java)

```java
public class SetPolyToPoly extends View{
    private static final String TAG = "SetPolyToPoly";

    private int testPoint = 0;
    private int triggerRadius = 180;    // 触发半径为180px

    private Bitmap mBitmap;             // 要绘制的图片
    private Matrix mPolyMatrix;         // 测试setPolyToPoly用的Matrix

    private float[] src = new float[8];
    private float[] dst = new float[8];

    private Paint pointPaint;

    public SetPolyToPoly(Context context) {
        this(context, null);
    }

    public SetPolyToPoly(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SetPolyToPoly(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initBitmapAndMatrix();
    }

    private void initBitmapAndMatrix() {
        mBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.poly_test2);

        float[] temp = {0, 0,                                    // 左上
                mBitmap.getWidth(), 0,                          // 右上
                mBitmap.getWidth(), mBitmap.getHeight(),        // 右下
                0, mBitmap.getHeight()};                        // 左下
        src = temp.clone();
        dst = temp.clone();

        pointPaint = new Paint();
        pointPaint.setAntiAlias(true);
        pointPaint.setStrokeWidth(50);
        pointPaint.setColor(0xffd19165);
        pointPaint.setStrokeCap(Paint.Cap.ROUND);

        mPolyMatrix = new Matrix();
        mPolyMatrix.setPolyToPoly(src, 0, src, 0, 4);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()){
            case MotionEvent.ACTION_MOVE:
                float tempX = event.getX();
                float tempY = event.getY();

                // 根据触控位置改变dst
                for (int i=0; i<testPoint*2; i+=2 ) {
                    if (Math.abs(tempX - dst[i]) <= triggerRadius && Math.abs(tempY - dst[i+1]) <= triggerRadius){
                        dst[i]   = tempX-100;
                        dst[i+1] = tempY-100;
                        break;  // 防止两个点的位置重合
                    }
                }

                resetPolyMatrix(testPoint);
                invalidate();
                break;
        }

        return true;
    }

    public void resetPolyMatrix(int pointCount){
        mPolyMatrix.reset();
        // 核心要点
        mPolyMatrix.setPolyToPoly(src, 0, dst, 0, pointCount);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.translate(100,100);

        // 绘制坐标系
        CanvasAidUtils.setCoordinateLen(900, 0, 1200, 0);
        CanvasAidUtils.drawCoordinateSpace(canvas);

        // 根据Matrix绘制一个变换后的图片
        canvas.drawBitmap(mBitmap, mPolyMatrix, null);

        float[] dst = new float[8];
        mPolyMatrix.mapPoints(dst,src);

        // 绘制触控点
        for (int i=0; i<testPoint*2; i+=2 ) {
            canvas.drawPoint(dst[i], dst[i+1],pointPaint);
        }
    }

    public void setTestPoint(int testPoint) {
        this.testPoint = testPoint > 4 || testPoint < 0 ? 4 : testPoint;
        dst = src.clone();
        resetPolyMatrix(this.testPoint);
        invalidate();
    }
}
```

*****

## 布局文件

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:gravity="center"
	tools:context="com.gcssloop.canvas.MainActivity">

	<com.gcssloop.canvas.SetPolyToPoly
		android:id="@+id/poly"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:layout_marginBottom="60dp"/>

	<LinearLayout
		android:orientation="horizontal"
		android:layout_alignParentBottom="true"
		android:layout_width="match_parent"
		android:padding="10dp"
		android:gravity="center"
		android:layout_height="60dp">

		<RadioGroup
			android:id="@+id/group"
			android:orientation="horizontal"
			android:gravity="center"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content">
			<RadioButton
				android:checked="true"
				android:id="@+id/point0"
				android:text="0"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"/>
			<RadioButton
				android:id="@+id/point1"
				android:text="1"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"/>
			<RadioButton
				android:id="@+id/point2"
				android:text="2"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"/>
			<RadioButton
				android:id="@+id/point3"
				android:text="3"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"/>
			<RadioButton
				android:id="@+id/point4"
				android:text="4"
				android:layout_width="wrap_content"
				android:layout_height="wrap_content"/>
		</RadioGroup>
	</LinearLayout>

</RelativeLayout>
```

*****

## MainActivity

``` java
setContentView(R.layout.activity_main);

final SetPolyToPoly poly = (SetPolyToPoly) findViewById(R.id.poly);

RadioGroup group = (RadioGroup) findViewById(R.id.group);
assert group != null;
group.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(RadioGroup group, int checkedId) {
        switch (group.getCheckedRadioButtonId()){
            case R.id.point0: poly.setTestPoint(0); break;
            case R.id.point1: poly.setTestPoint(1); break;
            case R.id.point2: poly.setTestPoint(2); break;
            case R.id.point3: poly.setTestPoint(3); break;
            case R.id.point4: poly.setTestPoint(4); break;
        }
    }
});
```

*****

## 依赖的库

绘制坐标系部分依赖了一个开源库。

* [ViewSupport](https://github.com/GcsSloop/ViewSupport )


