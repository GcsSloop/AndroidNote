# Markdown实用技巧－链接和图片

博客地址： http://www.gcssloop.com/markdown/markdown-links

Sloop 喝过半杯咖啡，涨红的脸色渐渐复了原，旁人便又问道，“ Sloop，你当真会写文章么？” Sloop 看着问他的人，显出不屑置辩的神气。他们便接着说道，“你怎的连半个赞也捞不到呢？” Sloop 立刻显出颓唐不安模样，脸上笼上了一层灰色，嘴里说些话；这回可是全是技术名词之类，一些不懂了。在这时候，众人也都哄笑起来：办公室内外充满了快活的空气。

在这些时候，我可以附和着笑，老板是决不责备的。而且老板见了 Sloop，也每每这样问他，引人发笑。Sloop 自己知道不能和他们谈天，便只好向实习生说话。有一回对我说道，“你会用 Markdown 么？” 我略略点一点头。他说，“会用 Markdown，……我便考你一考。Markdown 的链接，你是怎样写的？” 我想，码农一样的人，也配考我么？便回过脸去，不再理会。Sloop 等了许久，很恳切的说道，“不能写罢？……我教给你，记着！这些写法应该记着。将来做程序员的时候，写文档要用。”我暗想我和程序员的等级还很远呢，而且我们这里的程序员也从不写文档；又好笑，又不耐烦，懒懒的答他道，“谁要你教，不是一对方括号后面跟一对圆括号么？” Sloop 显出极高兴的样子，将两个指头的长指甲敲着电脑，点头说，“对呀对呀！……链接有4样基本写法，你知道么？” 我愈不耐烦了，努着嘴走远。Sloop 刚想打开编辑器，给我演示，见我毫不热心，便又叹一口气，显出极惋惜的样子。

*****

## 前言

**本文是适用于对 Markdown 有一定了解的魔法师，以帮助他们挖掘更多关于 Markdown 的可能性，例如：链接的不同类型以及使用方式，如何在新标签页打开链接，如何控制图片大小等，对 Markdown 还不了解的魔法师请参考 [Markdown快速入门][markdown-start] 和 [Markdown基础语法][markdown-grammar] 。**

**注意：以下的部分语法不属于标准语法，存在不兼容的问题，不能保证所有平台都能够使用。对于非标准语法(拓展语法)我会进行标注说明。**

*****

## 行内式链接:

```markdown
博客地址: [GcsSloop](http://www.gcssloop.com)  
博客地址: [GcsSloop](http://www.gcssloop.com "GcsSloop的博客")
```

博客地址: [GcsSloop](http://www.gcssloop.com)  
博客地址: [GcsSloop](http://www.gcssloop.com "GcsSloop的博客")

*****

## 参考式链接:

```markdown
[GcsSloop的博客][gcssloop]

[gcssloop]: http://www.gcssloop.com
// 或者
[gcssloop]: http://www.gcssloop.com "点击访问GcsSloop的博客"
```

[GcsSloop的博客][gcssloop]

**为什么要使用参考式呢？**   
在写文章的时候很可能会在文章不同的地方引用同一篇文章，使用参考式可以少写一点字符。  
更重要的是，参考文章的链接可能会改变，如果将参考链接统一写在文末的话，改起来会更容易。

*****

## 自动链接:

```markdown
<http://www.gcssloop.com>
```

<http://www.gcssloop.com>

*****

## 相对链接:

**如果你的内容是发布在 GitHub 或者自己的个人网站上，那么相对链接是一个很好用的东西**，例如引用本站的一张图片可以这样写:

```markdown
![头像](/assets/siteinfo/avatar.jpg)
```

![头像](/assets/siteinfo/avatar.jpg)

**相对链接优点:**

* 线下预览和线上效果相同，不受网络影响。
* 避免了分别上传图片导致的麻烦。
* GitHub复制仓库更方便，改仓库名字不会导致资源链接失效。

**相对链接缺点:**

* 不适用于需要在多平台发布的文章。
* 某些本地编辑器不支持读取相对链接。

*****

## 图片链接:

图片链接其实就是把图片和链接嵌套在一起:

> 顺便为 DiyCode 打一个小广告，欢迎更多小伙伴的加入。

```markdown
[![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9e6ii0bsgj30xc04gaau.jpg)](http://www.diycode.cc/wiki/encouragement)
```

[![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9e6ii0bsgj30xc04gaau.jpg)](http://www.diycode.cc/wiki/encouragement){:target="_blank"}

*****

IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII

**注意：从这里开始往下的部分，属于拓展语法，可能存在某些平台(编辑器)无法识别的问题，请亲自测试后再使用。**

IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII

## 在新标签页打开:

Markdown 的默认链接方式都是在当前标签页打开的，这就会导致打开链接之后原始页面被覆盖掉，如果想要让链接在新标签页打开可以在后面添加上 `{:target="_blank"}` 或者使用 HTML 语法。

```
[GcsSloop](http://www.gcssloop.com){:target="_blank"}

[GcsSloop的博客][gcssloop]{:target="_blank"}

<http://www.gcssloop.com>{:target="_blank"}

<a href="http://www.gcssloop.com/info/about" target="_blank">关于GcsSloop</a>
```

[GcsSloop](http://www.gcssloop.com){:target="_blank"}  <br/>
[GcsSloop的博客][gcssloop]{:target="_blank"}  <br/>
<http://www.gcssloop.com>{:target="_blank"} <br/>
<a href="http://www.gcssloop.com/info/about" target="_blank">关于GcsSloop</a>

注意: 部分平台可能不识别 `{:target="_blank"}` 标签，例如你正在看的这个 GitHub 就不识别。。

*****

## 注释(脚注):

注释一般用于解释一些专业名词或者难以理解的内容，由于注释的解释部分一般放在文末，所以又称为脚注:

```
GcsSloop[^1]是一个超级魔法师[^2] 。

[^1]: GcsSloop：非著名程序员。  
[^2]: 魔法师：会魔法的人类
```

GcsSloop[^1]是一个超级魔法师[^2] 。

注意：脚注不论在何处定义，最终都是显示在文末。部分平台不识别该语法，GitHub 依旧不识别。

*****

## 控制图片大小

使用 `{:width="300" height="100"}` 或者 HTML 格式可以控制图片显示大小，图片有宽度(width)和高度(height)两个属性，如果只指定了一个，另一个会按照比例缩放。

```
![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9k7b8a6vmj312w0rg143.jpg){:width="300"}

<img src="http://ww1.sinaimg.cn/large/005Xtdi2jw1f9k7b8a6vmj312w0rg143.jpg" width="300"/>
```

![](http://ww1.sinaimg.cn/large/005Xtdi2jw1f9k7b8a6vmj312w0rg143.jpg){:width="300"}

右键，新标签页打开图片，你会发现原图其实挺大的。  
注意: 部分平台可能不识别 `{:width="300" height="100"}` 标签，你正在看的这个 GitHub 依旧不识别。

*****

## 总结:

Markdown 存在很多的变种，对其语法进行了不同程度的拓展，使其更加的强大，但是使用拓展语法之前请三思。个人建议如下：

* 如果文章(文档)只在单一平台发布，使用任何该平台支持的拓展语法都没问题。
* 如果文章(文档)需要在多个平台发布，尽量使用标准语法，使用拓展语法之前请注意测试平台兼容性。
* 图片尽量使用图床管理，而且要进行本地备份。


*****

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

## 参考链接：

[Markdown－基础语法](http://www.markdown.cn/)  
[Markdown语法:在新窗口新标签页中打开](http://yinping4256.github.io/cn/Markdown%E8%AF%AD%E6%B3%95%E5%9C%A8%E6%96%B0%E7%AA%97%E5%8F%A3%E6%96%B0%E6%A0%87%E7%AD%BE%E9%A1%B5%E4%B8%AD%E6%89%93%E5%BC%80/)


### 注释:

[^1]: GcsSloop：非著名程序员。  
[^2]: 魔法师：会魔法的人类




[markdown-start]: http://www.gcssloop.com/markdown/markdown-star  "Markdown实用技巧－快速入门"

[markdown-grammar]: http://www.gcssloop.com/markdown/markdown-grammar "Markdown实用技巧－基础语法"

[gcssloop]: http://www.gcssloop.com	"点击访问GcsSloop的博客"
