# 优雅的发布Android开源库(论JitPack的优越性)
### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>
### [JitPack地址](https://jitpack.io)

## 前言
自从谷歌宣布不支持Eclipse之后，大批Android程序猿情愿或者不情愿的迁移到了AndroidStudio，从此过上了使用Gradle构建程序的"优越"生活。

> 关于Gradle的坑，就不吐槽了，我怕一会儿控制不住情绪。今天我们就谈一下Gradle的优越性。

说到Gradle的优越性，其中有一点比较明显的就是依赖开源库更加方便了，基本上一两行代码就能搞定。免去了还要手动下载自己配置的痛苦。

然而，这也仅仅是对使用者而言，而对于发布这些开源库的人就苦逼了,主要是上传太痛苦。

目前来说，比较常见的 Android 开源库托管地址有以下几类:

类型          |   吐槽
------------- | ------------
Maven Central | 发布过程繁杂冗长， 每次发布成功都应该感谢一下上苍的厚爱。
jCenter       | jCenter貌似稍微简单一点，但也不是省油的灯。 
自定义仓库    | 一般的猿猿玩不起，企业内部可能会见到。

在这些托管地址上面发布过项目的都应该能理解其中的痛苦，不说了，让我哭会儿(我就是那个每次发布都折腾半天的“bug狂魔”，从未一次发布就成功过)。

然而，现在福音来了，JitPack可以帮助你简单快速的发布开源库。

## 在正式讲解之前我们先了解一下JitPack

**JitPack是什么？**

> **JitPack是一个自定义的Maven仓库。**

**JitPack安全吗？**

> **个人还是比较安全的，毕竟开源库都是给大家用的，源码都能分享出来，如果你是担心它在里面插入恶意代码的话，在AndroidStudio的 External Libraies里面能够看到反编译的依赖库的源码，可以查看一下。**

**JitPack好处都有啥！说对我就给他用(金坷垃，雾)**

> **省时间，省时间，省时间，省下的时间都够你修复好几个bug了。**

简单的了解了JitPack之后，开始本篇的正文。


# 如何在JitPack上发布你的Library

**首先，假设大家已经具备了以下条件：**

序号 | 条件
:---:|---------
  1  | 会使用GitHub，能提交项目到GitHub上
  2  | 使用AndroidStudio，且Gradle版本在2.4以上

在具备了这些条件之后，正式开始发布一个项目(以我的一个[工具仓库Sutil](https://github.com/GcsSloop/SUtil)为例)。

## 第 1 步: 新建一个Project

在AndroidStudio中新建一个Project用于发布项目，新建完成之后结果是这样子:

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f239wl5amtj30rs0gon0g.jpg)

## 第 2 步: 在这个Project中添加一个Library

添加的这个Library就是我要发布的仓库，Library的名字无所谓,可以随便起(*我这里就叫library*)。添加完成之后是这样子：

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1f239xb835xj30rs0gowiv.jpg)

### 图中的几个标注

序号 | 解释
:---:|-------
  1  | 新添加的Library
  2  | Library的build.gradle 
  3  | Library的plugin

**其中library的plugin是下面这样子：**

``` gradle
apply plugin: 'com.android.library'
```

## 第 3 步: 给你的项目添加配置(重点)

你需要对你的项目简单的配置一下:

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f239y5xsj4j30rs0gowit.jpg)

**在你项目的根节点的 build.gradle(图示1) 中添加如下代码:**

``` gradle
buildscript { 
  dependencies {
    // 重点就是下面这一行(上面两行是为了定位这一行的添加位置)
    classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3' 
```
**[完整示例](https://github.com/GcsSloop/SUtil/blob/master/build.gradle)**

**在你要发布的library的 build.gradle(图示2) 中添加如下代码：**

``` gradle
 apply plugin: 'com.github.dcendents.android-maven'  

 group='com.github.YourUsername'
```
**[完整示例](https://github.com/GcsSloop/SUtil/blob/master/library/build.gradle)**

## 第 4 步: 提交项目到GitHub仓库

这一步就不多啰嗦了，不论你是用命令行还是客户端都可以。

**为了提交更加快速，你可以删除无用的文件(文件夹),至于需要保留哪些文件你可以参考官方给出的[示例仓库](https://github.com/jitpack/android-example)**

## 第 5 步: Release你的仓库或者给你的仓库打一个Tag(重点)

### 1.点击图示进入Release界面
![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f239yqr44cj30rs0goadk.jpg)

### 2.创建一个Release或Tag
![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f239z1dnr8j30rs0goaco.jpg)

### 3.填写基本信息
![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f239zctc5xj30rs0goq7w.jpg)

### 4.完成
![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f239zpfkcwj30rs0gogns.jpg)

## 第 6 步: 将你的仓库地址提交到JitPack(重点)

### 1.将你的仓库地址提交到[JitPack](https://jitpack.io)
**[JitPack地址戳这里](https://jitpack.io)**

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1f23a055uoej30rs0godi0.jpg)

序号 | 解释
:---:|--------
  1  | 粘贴你的仓库地址
  2  | 点击这里查看
  3  | 版本号
  4  | 点击这里提交该版本
  5  | 提交完成后自动生成的日志

### 2.JitPack自动生成的配置信息
在上传完成之后，JitPack会自动生成引用该仓库的配置信息，如下：

![](http://ww4.sinaimg.cn/large/005Xtdi2jw1f23a0fd5iyj30rs0gotb2.jpg)

**以上就是教程的全部内容，各位小伙伴可以回去愉快的发布自己的开源库了。**
### [JitPack地址](https://jitpack.io)

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300/> </a>








