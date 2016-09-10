# Matrix-Camera

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
### 相关文章: [自定义View目录](http://www.gcssloop.com/customview/CustomViewIndex/)

本篇依旧属于Matrix，主要讲解Camera，Android下有很相机应用，其中的美颜相机更是不少，不过今天这个Camera可不是我们平时拍照的那个相机，而是graphic包下的Camera，专业给Matrix拍照的相机，不过既然是相机，作用都是类似的，主要是将3D的内容拍扁变成2D的内容。

众所周知，我们的手机屏幕是一个2D的平面，所以也没办法直接显示3D的信息，因此我们看到的所有3D效果都是3D在2D平面的投影而已，而本文中的Camera主要作用就是这个，将3D信息转换为2D平面上的投影，实际上这个类更像是一个操作Matrix的工具类，使用Camera和Matrix可以在不使用OpenGL的情况下制作出简单的3D效果。



## Camera常用方法表

| 方法类别 | 相关                                       | 简介             |
| ---- | ---------------------------------------- | -------------- |
| 基本方法 | save、restore                             | 保存、 回滚         |
| 常用方法 | getMatrix、applyToCanvas                  | 获取Matrix、应用到画布 |
| 旋转   | rotat (API 21)、rotateX、rotateY、rotateZ   | 各种旋转           |
| 平移   | translate                                | 位移             |
| 相机位置 | setLocation (API 12)、getLocationX (API 16)、getLocationY (API 16)、getLocationZ  (API 16) | 设置与获取相机位置      |

> Camera的方法并不是特别多，很多内容与之前的讲解的Canvas和Matrix类似，不过又稍有不同，之前的画布操作和Matrix主要是作用于2D空间，而Camera则主要作用于3D空间。



## 基础概念

在具体讲解方法之前，先补充几个基础概念，以便于后面理解。

#### 3D坐标系

我们Camera使用的3维坐标系是**左手坐标系，即左手手臂指向x轴正方向，四指弯曲指向y轴正方向，此时展开大拇指指向的方向是z轴正方向**。

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f7mruav2nhj308c05iglp.jpg)

> 至于为什么要用左手坐标系呢？~~大概是因为赶工的时候右手不方便比划吧，大雾。~~实际上不同平台上使用的坐标系也有不同，有的是左手，有的是右手，貌似并没有统一的标准，只需要记住 Android 平台上面使用的是左手坐标系即可。

不过此处需要注意一下 Android 中 2D坐标系 和 3D坐标系 的一些差别。

| 坐标系     | 2D坐标系 | 3D坐标系  |
| ------- | :---: | :----: |
| 原点默认位置  |  左上角  |  左上角   |
| X 轴默认方向 |   右   |   右    |
| Y 轴默认方向 |   下   |   上    |
| Z 轴默认方向 |   无   | 垂直屏幕向内 |

3D坐标系在屏幕中各个坐标轴默认方向展示:

> 注意y轴默认方向是向上，而2D则是向下。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f7nxn8hcqqj308c0ea74a.jpg)

#### 三维投影

**三维投影**是将三维空间中的点映射到二维平面上的方法。由于目前绝大多数图形数据的显示方式仍是二维的，因此三维投影的应用相当广泛，尤其是在计算机图形学，工程学和工程制图中。





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

```java
void getMatrix (Matrix matrix)
```

计算当前状态下矩阵对应的状态，并将计算后的矩阵赋值给参数matrix。

#### applyToCanvas

```java
void applyToCanvas (Canvas canvas)
```

计算当前状态下单矩阵对应的状态，并将计算后的矩阵应用到指定的canvas上。



## 旋转方法

旋转是Camera制作3D效果的核心(其实从)，不过它制作出来的并不能算是真正的3D，而是伪3D，因为没有厚度，

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300/> </a>

