# Matrix-Camera

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### 相关文章: [自定义View目录](http://www.gcssloop.com/customview/CustomViewIndex/)

本篇依旧属于Matrix，主要讲解Camera，Android下有很相机应用，其中的美颜相机更是不少，不过今天这个Camera可不是我们平时拍照的那个相机，而是graphic包下的Camera，专业给Matrix拍照的相机，不过既然是相机，作用都是类似的，主要是将3D的内容拍扁变成2D的内容。

众所周知，我们的手机屏幕是一个2D的平面，所以也没办法直接显示3D的信息，因此我们看到的所有3D效果都是3D在2D平面的投影而已，而本文中的Camera主要作用就是这个，将3D信息转换为2D平面上的投影，实际上这个类更像是一个操作Matrix的工具类，使用Camera和Matrix可以在不使用OpenGL的情况下制作出简单的3D效果。



## Camera常用方法表

| 方法类别 | 相关                                       | 简介             |
| ---- | ---------------------------------------- | -------------- |
| 基本方法 | [save](#基本方法)、[restore](#基本方法)           | 保存、 回滚         |
| 常用方法 | getMatrix、applyToCanvas                  | 获取Matrix、应用到画布 |
| 旋转   | rotate、rotateX、rotateY、rotateZ           | 各种旋转           |
| 平移   | translate                                | 位移             |
| 相机位置 | setLocation、getLocationX、getLocationY、getLocationZ | 设置与获取相机位置      |

> Camera的方法并不是特别多，很多内容与之前的讲解的Canvas和Matrix类似，不过又稍有不同，之前的画布操作和Matrix主要是作用于2D空间，而Camera则主要作用于3D空间。



## 基本方法

基本方法就有两个`save` 和`restore`，主要作用为`保存当前状态和恢复到上一次保存的状态`，通常成对使用，常用格式如下:

```java
camera.save();		// 保存状态
... 				// 具体操作
camera.retore();	// 回滚状态
```



## 常用方法

这两个方法是Camera中最基础也是最常用的方法。

####  getMatrix

获取当前状态下的矩阵。



## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300/> </a>

