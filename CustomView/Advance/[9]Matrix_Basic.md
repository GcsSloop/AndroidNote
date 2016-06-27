# Matrix基础

前面扯了四篇 Path 相关的内容，本次终于要到了大家期盼已久的 Matrix(传说中的黑客帝国!!!):

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)

如题，本篇的主角是 Matrix(其实并不是黑客帝国来着)。

仔细回想一下，我们在很多地方都见过 Matrix(画布操作，drawBitmap，path 等)， 感觉它就像是一个烦人家伙，什么事情都想插上一脚。 然而实际情况却并非如此， Matrix 一直都在我们看不到的地方默默付出着， 如果没有了 Matrix， 我们之前的很多方法都是无法运行的。 

这么说来 Matrix 也算个厉害人物，那么这个 Matrix 到底是何方神圣呢？

>
#### Matrix 的翻译过来是矩阵，模型。和其释义相同，Matrix是一个矩阵，其作用则是一个模型，一个控制视图状态的模型。

也就是说， 我们进行界面视图等转换都是需要依靠 Matrix 的帮助的，例如我们之前在 [Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 中讲解过的画布操作，这些操作的核心就是改变 Matrix 的数值。

## Matrix方法表

Matrix 有很多常用和不常用的方法，在本篇中重点不在于这些方法的讲解，而是帮助大家理解 Matrix 的一些基本概念。

方法类别   | 相关API                                                 | 摘要
-----------|---------------------------------------------------------|------------------------
比较方法   | equals hashCode                                         | 比较、 获取哈希值
基本方法   | set reset setValues getValues                           | 设置、 重置、 设置数值、 获取数值                    
数值计算   | mapPoints mapRadius mapRect mapVectors                  | 计算变换后的数值
设置(set)  | setConcat setRotate setScale setSkew setTranslate       | 设置变换
前乘(pre)  | preConcat preRotate preScale preSkew preTranslate       | 前乘变换
后乘(post) | postConcat postRotate postScale postSkew postTranslate  | 后乘变换
特殊方法   | setPolyToPoly setRectToRect rectStaysRect setSinCos  | 一些特殊操作
其它方法   | invert isAffine isIdentity  toString toShortString | 逆矩阵、 是否是仿射矩阵、 是否是单位矩阵 ...

## Matrix原理

Matrix 本质是一个 3 x 3 的矩阵。














