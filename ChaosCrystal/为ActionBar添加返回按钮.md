# 为 ActionBar 添加返回按钮

在 AndroidManifest.xml 文件中配置

``` xml
<activity android:name="com.gcssloop.test.viewsupport.TestCustomViewActivity"
  android:parentActivityName="com.gcssloop.test.MainActivity">
  <!-- 支持 4.0 以下的设备 -->
  <meta-data
    android:name="android.support.PARENT_ACTIVITY"
    android:value="com.gcssloop.test.MainActivity" />
</activity>
```
在 onCrate 中设置

```
// 3.0+ 
getSupportActionBar().setDisplayHomeAsUpEnabled(true);
// If your minSdkVersion is 11 or higher, instead use:
// getActionBar().setDisplayHomeAsUpEnabled(true);
```
