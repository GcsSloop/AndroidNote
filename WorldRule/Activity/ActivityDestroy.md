# Activity结束情况

## 1.正常结束

应用程序退出或者调用`finish()`方法正常结束的Activity。

## 2.系统配置改变导致Activity被销毁并重建

如手机旋转屏幕，弹出键盘等导致Activity销毁并重建。

如果想避免这种情况，可以在`AndroidManifest.xml`文件中为Activity进行配置，如下:

```
<activity
    android:name=".TestActivity"
    android:configChanges="keyboardHidden|orientation">
</activity>
```

> 屏蔽了键盘状态改变和屏幕方向改变导致Activity销毁并重建。

可配置属性有很多，想要配置多个属性可以用 `|` 连接起来。

属性               | 摘要
-------------------|------------------------------------------------
mcc                | SIM卡唯一标识码IMSI(国际移动用户识别码)中等国家代码发生改变，由三位数组成，中国为460。
mnc                | SIM卡唯一标识码IMSI(国际移动用户识别码)中等运营商代码发生改变，由两位数组成，中国移动为00，中国连通为01，中国电信为03。
locale             | 设备本地位置发生改变，一般指切换了系统语言。
touchscreen        | 触摸屏发生改变(通常不会发生，可以忽略)。
keyboard           | 
keyboardHidden     | 
navigation         | 
screenLayout       | 
fontScale          | 
uiMode             | 
orientation        | 
screenSize         | 
smallestScreenSize | 
layoutDirection    | 






