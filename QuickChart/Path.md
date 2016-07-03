# Path常用操作速查表

> 为了兼容性(_偷懒_) 本表格中去除了在API21(即安卓版本5.0)以上才添加的方法。忍不住吐槽一下，为啥看起来有些顺手就能写的重载方法要等到API21才添加上啊。宝宝此刻内心也是崩溃的。

作用            | 相关方法        | 备注
----------------|-----------------|------------------------------------------
移动起点        | moveTo          | 移动下一次操作的起点位置
设置终点        | setLastPoint    | 重置当前path中最后一个点位置，如果在绘制之前调用，效果和moveTo相同
连接直线        | lineTo          | 添加上一个点到当前点之间的直线到Path
闭合路径        | close           | 连接第一个点连接到最后一个点，形成一个闭合区域
添加内容        | addRect, addRoundRect,  addOval, addCircle, 	addPath, addArc, arcTo | 添加(矩形， 圆角矩形， 椭圆， 圆， 路径， 圆弧) 到当前Path (注意addArc和arcTo的区别)
是否为空        | isEmpty         | 判断Path是否为空
是否为矩形      | isRect          | 判断path是否是一个矩形
替换路径        | set             | 用新的路径替换到当前路径所有内容
偏移路径        | offset          | 对当前路径之前的操作进行偏移(不会影响之后的操作)
贝塞尔曲线      | quadTo, cubicTo | 分别为二次和三次贝塞尔曲线的方法
rXxx方法        | rMoveTo, rLineTo, rQuadTo, rCubicTo | **不带r的方法是基于原点的坐标系(偏移量)， rXxx方法是基于当前点坐标系(偏移量)**
填充模式        | setFillType, getFillType, isInverseFillType, toggleInverseFillType   | 设置,获取,判断和切换填充模式
提示方法        | incReserve      | 提示Path还有多少个点等待加入**(这个方法貌似会让Path优化存储结构)**
布尔操作(API19) | op              | 对两个Path进行布尔运算(即取交集、并集等操作)
计算边界        | computeBounds   | 计算Path的边界
重置路径        | reset, rewind   | 清除Path中的内容<br/> **reset不保留内部数据结构，但会保留FillType.**<br/> **rewind会保留内部的数据结构，但不保留FillType**
矩阵操作        | transform       | 矩阵变换

## 相关文章

* [Path基本操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B5%5DPath_Basic.md)
* [贝塞尔曲线](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B6%5DPath_Bezier.md)
* [Path完结篇(伪)](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B7%5DPath_Over.md)
* [Path玩出花样](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B8%5DPath_Play.md)

