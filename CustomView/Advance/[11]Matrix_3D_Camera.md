# Matrix-Camera

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### 相关文章: [自定义View目录](http://www.gcssloop.com/1970/01/CustomViewIndex/)

本篇依旧属于Matrix，主要讲解Camera，Android下有很相机应用，其中的美颜相机更是不少，不过今天这个Camera可不是我们平时拍照的那个相机，而是graphic包下的Camera，专业给Matrix拍照的相机，不过既然是相机，作用都是类似的，主要是将3D的内容拍扁变成2D的内容。

众所周知，我们的手机屏幕是一个2D的平面，所以也没办法直接显示3D的信息，因此我们看到的所有3D效果都是3D在2D平面的投影而已，而本文中的Camera主要作用就是这个，将3D信息转换为2D平面上的投影，我们应用中大多数3D效果都离不开这个类，而它将3D转换为2D投影的桥梁则是Matrix。



## Camera常用方法表

按照惯例，先放表。



| 返回值  | 方法                                       |
| ---- | ---------------------------------------- |
|      | Camera() <br/>构造方法，创建一个空的Camera。         |
| void | applyToCanvas(Canvas canvas)<br/>将当前的Camera变换应用到Canvas。 |
|      |                                          |
|      |                                          |
|      |                                          |




## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300/> </a>

