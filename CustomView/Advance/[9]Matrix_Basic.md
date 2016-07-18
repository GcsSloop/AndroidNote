# Matrix基础

前面讲了四篇 Path 相关的内容，本次终于要到了大家期盼已久的黑客帝国!

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)

如题，本篇的主角是 Matrix(并不是黑客帝国)。

它在我们在之前的很多文章中都提及过，但并没有仔细的介绍过，从本篇开始终于要正式介绍它了，这个在2D绘图中十分重要的角色 -- Matrix。

>
#### Matrix 的翻译过来是矩阵，模型。和其释义相同，Matrix是一个矩阵，其作用则是一个模型，一个控制视图状态的模型。

也就是说， 我们进行界面视图等转换都是需要依靠 Matrix 的帮助的，例如我们之前在 [Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 中讲解过的画布操作，这些操作的核心就是改变 Matrix 的数值。

## Matrix方法表

Matrix 有很多常用和不常用的方法，在本篇中重点不在于这些方法的讲解，而是帮助大家理解 Matrix 的一些基本概念。

方法类别   | 相关API                                                 | 摘要
-----------|---------------------------------------------------------|------------------------
基本方法   | equals hashCode toString toShortString                  | 比较、 获取哈希值、 转换为字符串
数值操作   | set reset setValues getValues                           | 设置、 重置、 设置数值、 获取数值                    
设置(set)  | setConcat setRotate setScale setSkew setTranslate       | 设置变换
前乘(pre)  | preConcat preRotate preScale preSkew preTranslate       | 前乘变换
后乘(post) | postConcat postRotate postScale postSkew postTranslate  | 后乘变换
数值计算   | mapPoints mapRadius mapRect mapVectors                  | 计算变换后的数值
特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos     | 一些特殊操作
矩阵相关   | invert isAffine isIdentity                              | 求逆矩阵、 是否为仿射矩阵、 是否为单位矩阵 ...


## Matrix原理

Matrix 本质是一个 3x3 的矩阵，里面有9个数值，分别用于控制视图的不同属性，大致如下:

   | 0         | 1         | 2
---|-----------|-----------|----------
 0 | MSSCALE_X | MSKEW_X   | MTRANS_X
 1 | MSKEW_Y   | MSSCALE_Y | MTRANS_Y
 2 | MPERSRP_0 | MPERSRP_1 | MPERSRP_2 














