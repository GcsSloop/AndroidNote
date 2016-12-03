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

``` java
// 3.0+ 
getSupportActionBar().setDisplayHomeAsUpEnabled(true);
// If your minSdkVersion is 11 or higher, instead use:
// getActionBar().setDisplayHomeAsUpEnabled(true);
```
监听返回按钮事件

> 它的 id 是 home，可以使用重载 `onOptionsItemSelected` 的方式进行监听。
``` java
@Override
public boolean onOptionsItemSelected(MenuItem item)
{
    // TODO Auto-generated method stub
    if(item.getItemId() == android.R.id.home)
    {
        finish();
        return true;
    }
    return super.onOptionsItemSelected(item);
}
```
