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

| 属性                 | 摘要                                       |
| ------------------ | ---------------------------------------- |
| mcc                | SIM卡唯一标识码IMSI(国际移动用户识别码)中等国家代码发生改变，由三位数组成，中国为460。 |
| mnc                | SIM卡唯一标识码IMSI(国际移动用户识别码)中等运营商代码发生改变，由两位数组成，中国移动为00，中国连通为01，中国电信为03。 |
| locale             | 设备本地位置发生改变，一般指切换了系统语言。                   |
| touchscreen        | 触摸屏发生改变(通常不会发生，可以忽略)。                    |
| keyboard           | 键盘类型发生改变，例如用户使用了外插键盘。                    |
| keyboardHidden     | 键盘的可访问性发生变化，例如用户调出了键盘。                   |
| navigation         | 系统导航方式发生了改变，例如使用了轨迹球(现在设备上已经很少见到轨迹球了)。   |
| screenLayout       | 屏幕布局发生了改变，例如用户激活了另外一个显示设备。               |
| fontScale          | 系统字体缩放比例发生改变，例如用户选择了一个新字号。               |
| uiMode             | 用户界面模式发生改变，例如开启了夜间模式(API 8 新添加)。         |
| orientation        | 屏幕方向发生改变，例如用户旋转了屏幕(横屏与竖屏切换)。             |
| screenSize         | 屏幕尺寸信息发生改变，当旋转设备屏幕时，屏幕尺寸会发生变化，这个选项与编译选项也有关，当编译选项中的 miniSdkVersion 和 targetSdkVersion 均低于 13 时，次选项不回导致Activity重启，否则会导致Activity重启(API 13 新添加)。 |
| smallestScreenSize | 设备无力屏幕尺寸发哼变化，这个与屏幕方向无关，仅表示实际的物理屏幕尺寸改变时变化，例如用户切换到了外部显示设备，这个选项和 screenSize 一样，当编译选项中的 miniSdkVersion 和 targetSdkVersion 均低于 13 时，次选项不回导致Activity重启，否则会导致Activity重启(API 13 新添加)。 |
| layoutDirection    | 当布局方向发生变化，很少用。修改布局的 layoutDirection 属性会引起该变化(API 17 新添加)。 |






