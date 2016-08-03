# 在AndroidStudio中使用PlantUML

### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)

## 前言

### 这是Windows平台设置教程，Mac平台戳这里：<a href="https://github.com/GcsSloop/AndroidNote/blob/master/Course/HowToUsePlantUMLInAS(Mac).md">在AndroidStudio中使用PlantUML(Mac)</a>

**Unified Modeling Language (UML)又称统一建模语言或标准建模语言，用来描述 类（对象的）、对象、关联、职责、行为、接口、用例、包、顺序、协作，以及状态。是用来帮助自己理清众多类之间复杂关系的不二利器，也能帮助别人快速理解你的设计思路。**

那么，我们怎么在AndroidStudio中创建自己的UML类图呢？接下来我就教大家如何用正确的姿势创建UML类图。

## 一.用正确的姿势安装panltUML插件
### 1.File->Settings->Plugins->Browse repositories
![这里写图片描述](http://img.blog.csdn.net/20151130192101011)
### 2.在搜索框输入plantUML
![这里写图片描述](http://img.blog.csdn.net/20151130192547549)
### 3.导入插件
#### (ps：由于我已经安装过了，所以没有Install plugin 按钮，未安装的都有这样一个按钮，如下，点击安装即可。)
![这里写图片描述](http://img.blog.csdn.net/20151130192907006)

#### 如果以上步骤正确的完成，重启AndroidStudio 右键->new 的时候你会发现多了这么一堆东西，如果出现了这些说明plantUML已经正确的安装了。
![这里写图片描述](http://img.blog.csdn.net/20151130193249965)

#### 当然了，所有事情都不会是一帆风顺的，当你迫不及待的想创建一个文件试试的时候你会发现下面的情况。
![这里写图片描述](http://img.blog.csdn.net/20151130193752721)
#### 想必此时你的内心一定和我当时一样，一万头草泥马奔腾而过，这都是什么东西！！！
#### 一切事情都是有原因的，而这个因为你还缺少一个必要的东西，就是大名鼎鼎的贝尔实验室开发的一个工具包：Graphviz。
## 二，用正确的姿势安装Graphviz
### 1.下载Graphviz
###[【下载地址戳这里】](http://www.graphviz.org/Download_windows.php)
![这里写图片描述](http://img.blog.csdn.net/20151130194703804)
### 2.安装
#### 安装过程我就不详细讲解了，点击next后要记住安装的目录。之后下一步，直到完成就行。
![这里写图片描述](http://img.blog.csdn.net/20151130195454083)
	
## 三.用正确的姿势设置plantUML
### 1.点击右上角的设置按钮或进入File->Settings->Other Settings ->PlantUML
### 2.将文件路径填写为刚刚Graphviz的目录下bin目录中dot.exe文件。
#### (我的为：D:/Program/Graphviz/bin/dot.exe)
![这里写图片描述](http://img.blog.csdn.net/20151130200308586)
### 3.点击OK 刷新一下界面就能看到这个了。
![这里写图片描述](http://img.blog.csdn.net/20151130200452927)
#### 讲到这里，就已经安装完成了，可以愉快的用代码来书写UML图了。
#### 什么？你说你还不会书写的语法？没关系，其实我也不会，不过我有一个好的教程推荐给你，相信你看完就明白啦。
## 四.用正确的姿势学习使用UML
### 1.[【PlantUML快速指南戳这里】](http://archive.3zso.com/archives/plantuml-quickstart.html)
### 2.注意，这个教程中的语法和AndroidStudio中基本一致，区别就是开始和结束标志不同。
	
####好了，到这里该教程正式结束，祝各位小伙伴能愉快的使用plantUML玩耍。

## About Me
### 作者微博: [@GcsSloop](http://weibo.com/GcsSloop)
<a href="https://github.com/GcsSloop/AndroidNote/blob/magic-world/FINDME.md" target="_blank"> <img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width=300 height=100 /> </a>



