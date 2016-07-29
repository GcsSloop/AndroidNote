# Android中dip、dp、sp、pt和px

概念区别：

单位 | 含义
--- | ---
dip | device independent pixels(设备独立像素). 不同设备有不同的显示效果,这个和设备硬件有关，一般我们为了支持WVGA、HVGA和QVGA **推荐使用这个，不依赖像素**。
dp | 同上，和dip一样。
px | pixels(像素). 不同设备显示效果相同，一般我们HVGA代表320x480像素。
sp | scaled pixels(放大像素). 主要用于字体显示best for textsize。
pt | point，是一个标准的长度单位，1pt＝1/72英寸，用于印刷业，非常简单易用。
in | （英寸）：长度单位。 
mm | （毫米）：长度单位。

## 工具包

在 [ViewSupport](https://github.com/GcsSloop/ViewSupport) 支持包中可以找到该工具。

## 单位转换代码：
``` java
/**
 * dp、sp 转换为 px 的工具类
 * 
 * @author fxsky 2012.11.12
 *
 */
public class DisplayUtil {
	/**
	 * 将px值转换为dip或dp值，保证尺寸大小不变
	 * 
	 * @param pxValue
	 * @param scale
	 *            （DisplayMetrics类中属性density）
	 * @return
	 */
	public static int px2dip(Context context, float pxValue) {
		final float scale = context.getResources().getDisplayMetrics().density;
		return (int) (pxValue / scale + 0.5f);
	}

	/**
	 * 将dip或dp值转换为px值，保证尺寸大小不变
	 * 
	 * @param dipValue
	 * @param scale
	 *            （DisplayMetrics类中属性density）
	 * @return
	 */
	public static int dip2px(Context context, float dipValue) {
		final float scale = context.getResources().getDisplayMetrics().density;
		return (int) (dipValue * scale + 0.5f);
	}

	/**
	 * 将px值转换为sp值，保证文字大小不变
	 * 
	 * @param pxValue
	 * @param fontScale
	 *            （DisplayMetrics类中属性scaledDensity）
	 * @return
	 */
	public static int px2sp(Context context, float pxValue) {
		final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
		return (int) (pxValue / fontScale + 0.5f);
	}

	/**
	 * 将sp值转换为px值，保证文字大小不变
	 * 
	 * @param spValue
	 * @param fontScale
	 *            （DisplayMetrics类中属性scaledDensity）
	 * @return
	 */
	public static int sp2px(Context context, float spValue) {
		final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;
		return (int) (spValue * fontScale + 0.5f);
	}
}
```

