# Matrix常用操作速查表

| 方法类别     | 相关API                                    | 摘要                         |
| -------- | ---------------------------------------- | -------------------------- |
| 基本方法     | equals hashCode toString toShortString   | 比较、 获取哈希值、 转换为字符串          |
| 数值操作     | set reset setValues getValues            | 设置、 重置、 设置数值、 获取数值         |
| 数值计算     | mapPoints mapRadius mapRect mapVectors   | 计算变换后的数值                   |
| 设置(set)  | setConcat setRotate setScale setSkew setTranslate | 设置变换                       |
| 前乘(pre)  | preConcat preRotate preScale preSkew preTranslate | 前乘变换                       |
| 后乘(post) | postConcat postRotate postScale postSkew postTranslate | 后乘变换                       |
| 特殊方法     | setPolyToPoly setRectToRect rectStaysRect setSinCos | 一些特殊操作                     |
| 矩阵相关     | invert isAffine(API21) isIdentity        | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ... |

## 相关文章

* [安卓自定义View进阶 - Matrix原理](http://www.gcssloop.com/customview/Matrix_Basic/)
* [安卓自定义View进阶 - Matrix详解](http://www.gcssloop.com/customview/Matrix_Method/)
* [安卓自定义View进阶 - Matrix Camera](http://www.gcssloop.com/customview/matrix-3d-camera)

