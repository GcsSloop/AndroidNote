# Markdown实用技巧－编辑器(Typora)

本次的安利对象是一个 Markdown 编辑器，是会长见过的最简单，最优雅的编辑器，先来看一下它的界面吧：

![QQ20161207-0](http://ww3.sinaimg.cn/large/006y8lVajw1fahmo8d9udj30ph0brgo0.jpg)



它的界面非常简单，有多种主题可选，更重要的是**它的预览界面和编辑界面是一体的，而不像其他编辑器那样是左右分开的。**

![](http://ww2.sinaimg.cn/large/005Xtdi2jw1fahnsrelggg30fp0dwakd.gif)

从上面的示例中可以看出，其输入模式支持多种，不论是手动输入语法还是使用快捷键，都非常的流畅，实时看到效果变化，除此之外 Typora 还有很多优点。

## Typora 的优点

* 预览和编辑界面一体。
* 强大的快捷键。
* 兼容常见扩展语法。
* 兼容 HTML (新版进行了完善)。
* 支持 YAML 格式。
* 支持公式编辑(LaTeX公式)。
* 表格和代码块编辑起来非常方便。
* 支持本地图片相对链接。
* 支持将本地图片拖进编辑页面。
* 支持多种导出格式(PDF，HTML，Word，LaTeX 等)。
* 有多种主题可选。

虽然上面的内容在其他编辑器上也可以见到，但做到这么完善的并不多，想要尝试的小伙伴赶紧来试试吧，相信你也会爱上它的。

#### [点击这里进入 Typora 官网](http://www.typora.io/) 

既然 Typora 这么好，为什么不在一开始就推荐呢，这是因为在 Markdown 系列文章第一篇发布的时候，Typora 还有一些小瑕疵(HTML语法兼容部分)让会长不满意，所以只是放在了推荐首位，在最近更新了新版本之后，这一部分已经修复完善了，基本算是完美了，所以会长我才特地写一篇文章来推荐。

**Typora 支持在 WIndows，Linux，和 Mac 上使用，如果你正在使用的是 Mac 的话，那么恭喜你，除了上述的优点，你可以用到一些 Mac 系统才有的福利。**

## 版本回溯功能

这是 Mac 系统提供的一个小功能，使用 Mac 的时候你可能会注意到一个小细节，**使用系统文本编辑器进行编辑文本的时候，从来不会提示你保存文本，即便编辑后不点保存立即退出里面的内容也不会丢失。**你可能会觉得这不就是一个自动保存功能么，有啥稀奇的？

那么你是否遇到过这一种情况，编辑了半天的内容觉得不太好想放弃保存，继续使用之前的内容，这在Windows上很容易实现，只要编辑的时候没有手动保存，直接点击关闭不保存，再打开的时候就是编辑之前的内容。

然而在 Mac 上自动保存了，如果关闭后再打开，点 `Command + Z` 也没用了，这不就尴尬了，难道说遇到这种情况只能关闭前狂点 `Command + Z` 回退？

作为一个有情怀的操作系统，自然不会意识不到这个问题，它提供了更强大的方法，那就是版本回溯，相当于一个自动的 Git 系统，这就厉害了。

Trpora 也用到了这一特性，点击 `File > Revert To > Browse All Versions` 就能看到了。以我之前发布的一篇文章为例：

![](http://ww2.sinaimg.cn/large/006y8lVajw1facti4okblj31400p0gs8.jpg)

 **上图中左边为当前版本，右边为选中版本，最右侧是时间线，使用这一功能可以将文章回溯到任意时间点，妈妈再也不怕我的文章内容丢失啦，也成功避免了以下这种情况 ▼**

![](http://ww3.sinaimg.cn/large/005Xtdi2jw1factpcyrfqj30rs0ja40l.jpg)

由于这一功能是 Mac 系统提供的，所以不仅对纯文本有效，对 iWork 也是有效的，iWork 上面也有版本回溯功能，像 key，numbers，pages 都支持版本回溯，中文版点 `文件 > 复原 > 浏览所有版本` 即可看到过去保存的数据。

当然了，这只是福利之一。

## 更便捷的图片插入和上传方式

众所周知，Markdown 插入图片是一个大问题，尤其是在本地编辑的时候，但是这些在遇到 Typora 后就变的越来越简单了。

**对于本地图片，我们可以直接拖进来，再也不用记复杂的语法和查图片地址了，没错，只要拖进来就行，Typora 会自动识别图片并进行插入。**

不过还有一个问题，我们的很多文章都是要发布的，直接拖进来这种方式在本地看起来没问题，但上传到网站上是读取不到本地图片的，通常解决方案就是先上传到图床上，之后再将链接插入到 Markdown 中，这样发布就没问题了。

但是，当一篇文章需要插入很多图片的时候，一次次的上传，一次次的复制链接，一次次的粘贴，也是很麻烦的，虽然有一些脚本可以通过快捷键进行上传，但终归还是多了一些步骤，很不方便，而且对新手很不友好，更何况很多新手根本不知道图床所谓何物。

这时候就要祭出 Mac 上另一个神器了: iPic [(点击此处进行下载)](https://itunes.apple.com/cn/app/ipic-tu-chuang-shen-qi-zhong/id1101244278?mt=12)，它是一个专业图床管理工具，貌似出来还没多久，目前免费版只能上传到微博图床，专业版支持大部分图床，专业版一年 30RMB，如果经常使用的话，倒也不算贵。

**话说不还是要用图床么？**

**但 Typora 的方便之处就在于可以和 iPic 无缝结合，纯傻瓜式一键操作，能一次性的将文章中所有本地图片上传到图床上。**

**首先安装 iPic，运行 iPic，之后点击 `Edit > Image Tools > Upload Local Images via iPic` 就像下面这样就行了。**

![![QQ20161207-0](../Downloads/QQ20161207-0.png)QQ20161202-8](http://ww1.sinaimg.cn/large/006y8lVajw1facuf1yjjkj30fo0dmq4v.jpg)

**如果你觉得这样还是很麻烦的话，还有另外的选项，可以选择插入本地图片的时候自动上传到服务器，当然了，为了保护个人隐私，这一个选项默认是关闭的，首先你要找到首选项，点击 `Typora > Preferences` 或者快捷键 `Command + ,` 都行，之后打开以下两个选项，其中一个选项是后面用到的。**

![QQ20161202-9](http://ww3.sinaimg.cn/large/006y8lVajw1fahlaeo8pej30by0ddgn8.jpg)

**打开之后继续选择 `Edit > Image Tools > When Insert Local Images > Upload Image via iPic` 它的意思是当插入本地图片时自动通过 iPic 上传到服务器上。**

![QQ20161203-0](http://ww2.sinaimg.cn/large/006y8lVajw1fahlajkq1gj30qe0hwtas.jpg)

这种方案虽然方便，但是会长并不推荐大家这么做，主要还是不安全，误操作的话可能会将一些包含个人隐私的照片上传上去。

会长推荐另一种方案，就是 `Copy Image File to Folder`，操作步骤和上面类似，它的意思是插入本地图片时自动归档到某一个文件夹，在文章编辑完成后在点击上传按钮统一上传，这样有以下几点好处：

* 安全，不会因为误操作将涉及隐私照片传到服务器上。
* 相当于自动建立了一个本地图片档案，方便备份保存。
* 如果图床上的图片丢失，可以从快速从本地备份中找到并恢复。


关于 Typora 的推荐内容暂时就到这里结束啦，更多关于 Typora 的多详情请参阅 [Support](http://support.typora.io/)。关于 Markdown 的更多内容请参见之前的文章:

[Markdown实用技巧－快速入门](https://www.gcssloop.com/markdown/markdown-start)  
[Markdown实用技巧－基础语法](https://www.gcssloop.com/markdown/markdown-grammar)  
[Markdown实用技巧－链接和图片](https://www.gcssloop.com/markdown/markdown-links)   

## 结语

最后稍微夹带一点个人建议，个人觉得 Typora 除了界面简洁之外，最大的优点就是快捷键方便了，然而，快捷键那么多，要不要专一去记呢？

个人的建议是不要专一去记忆这些快捷键，用到的时候打开菜单看一眼快捷键是什么，之后用快捷键按出来，这样用过几次之后自然就会记住常用的快捷键，而且会记得很牢固，不然每个应用都有快捷键，一个一个的记多麻烦。

本文中涉及到的两个软件下载地址：

[**Typora**](http://www.typora.io/)  
[**iPic**](https://itunes.apple.com/cn/app/id1101244278?ls=1&mt=12)

**最后留一个课后作业，关注会长的 [新浪微博](http://weibo.com/GcsSloop) 。**

## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

