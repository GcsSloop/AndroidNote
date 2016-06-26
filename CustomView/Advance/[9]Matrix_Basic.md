# Matrix基础

前面扯了四篇 Path 相关的内容，本次终于要到了大家期盼已久的 Matrix，没错，就是传说中的黑客帝国:

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4oyx5i8wbj308c0bj3zz.jpg)

勇敢的骚年啊，你是选择红药丸还是选择蓝药丸呢？

> 
我选蓝色的，不要问我为什么。

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f4pji1l0lej308c04pwer.jpg)

好了，言止于此，不能说更多了，下面开始我们本篇的正题。

## 前言

如题，本篇的主角是 Matrix，在 Canvas 中很多地方都有 Matrix 的身影，如: 画布操作，drawBitmap，path 等。几乎无处不在，那么 Matrix 到底是干什么的呢？

>
**Matrix 的翻译过来是矩阵，模型。和其释义相同，Matrix是一个矩阵，其作用则是一个模型，一个控制视图状态的模型。**

我们之前在 [Canvas之画布操作](https://github.com/GcsSloop/AndroidNote/blob/master/CustomView/Advance/%5B3%5DCanvas_Convert.md) 中讲解过不少画布相关操作，这些操作的核心就是是操作 Matrix

## Matrix方法表

Matrix 有很多常用和不常用的方法，在本篇中重点不在于这些方法的讲解，而是帮助大家理解 Matrix 的一些基本概念。

方法类别   | 相关API                                                 | 摘要
-----------|---------------------------------------------------------|-------------------------------------
比较方法   | equals hashCode                                         | 比较、 获取哈希值
基本方法   | set reset setValues getValues                           | 设置、 重置、 设置数值、 获取数值
矩阵相关   | invert isAffine isIdentity                              | 计算逆矩阵、 是否是仿射矩阵、 是否是单位矩阵
数值计算   | mapPoints mapRect mapVectors                            | 计算 点、 矩形、 阵列 经过变换后点位置
设置(set)  | setConcat setRotate setScale setSkew setTranslate       | 设置变换
前乘(pre)  | preConcat preRotate preScale preSkew preTranslate       | 前乘变换
后乘(post) | postConcat postRotate postScale postSkew postTranslate  | 后乘变换















