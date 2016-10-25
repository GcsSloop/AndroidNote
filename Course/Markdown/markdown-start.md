# Markdown 快速入门

自从接触了 Markdown 之后，就一直用 Markdown 作为自己的主要书写工具，不论是平时做一些简单的纪录，还是用来写博客，写文档都是非常方便。你现在看到的这篇文章就是用 Markdown 进行书写的。

> 我最早因为 GitHub 而了解到 Markdown，当是支持 Markdown 的平台并不多，现在发现很多平台都已经开始支持 Markdown了，不论是老牌的 CSDN 还是比较新的 简书、掘金、DiyCode 等都支持使用Markdown进行写作，借此趋势，赶紧向还不了解 Markdown 的魔法师强势安利一波。
>
> **如果你已经开始使用 Markdown了，那么本文作用对你可以能并不大，请看后续文章。**



****

## 什么是 Markdown ?

**Markdown 是一种轻量级标记语言，创始人为約翰·格魯伯（John Gruber）。 它允许人们“使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML(或者HTML)文档”。**

相比于 HTML (~~How To Make Love~~ 大雾)， **Markdown 更加精简，更加注重内容，其主要宗旨是「易读易写」** 一般 Markdown 最终都是要转换为 HTML 的，可用于书写博客或者网页，但借助某些工具，可以讲 Markdown 转换为 pdf，word，Latex 等其他常见的文件格式。



****

## 为什么选择 Markdown ?

**选择 Markdown 但理由只有一个：方便，节省时间!**

至于为什么这样说，请看下面内容：

* **语法简洁**，没有任何编程基础的人十几分钟语言即可入门。
* **注重内容**，专注于内容编写，不再因为格式拍版而苦恼 (word格式刷工具哭晕在厕所)。
* **易阅读性**，即便是没有经过转换的 Markdown 文件，大部分文字内容仍可阅读。
* **易编辑性**，任何文本编辑器都能编辑 Markdown 文件。
* **跨平台性**，任何平台均能打开 Markdown 文件，由于是纯文本文件，不存在格式兼容的问题。
* **导出方便**，支持导出为 HTML，PDF，Word(.docx)，LaTex 等常见格式(需要工具支持)。

在 Windows 上编写的文档，非常方便的就能在 Mac 上继续编辑，方便数据迁移，降低沟通成本。



****

## Markdown 存在的问题

前面吹嘘了 Markdown 的那么多优点，下面就说一下其中的不足：

* **图片问题**，很多人都觉得 Markdown 文件插入图片麻烦，还要自己上传找链接。
* **语法兼容**，基础语法是兼容的，但不同工具(平台)的扩展语法不兼容(由于没有统一标准)。
* **细节控制**，Markdown只提供最基础的格式，其显示样式主要由CSS控制，很难针对性的控制部分内容。

以上应该是 Markdown 最常见的一些麻烦，不过不必担心，**后续文章会教大家来如何解决这些问题，取其精华，去其糟粕，让 Markdown 运用得心应手**。

****

## Markdown 编辑器推荐

俗话说，工欲善其事，必先利其器，虽然 Markdown 用任何文本编辑器都能打开编辑，但仍需要专业工具进行转化，常见 Markdown 编辑器我基本上都尝试用过，在此简单推荐几种，大家找适合自己的就行。

**仅推荐本地编辑器，在线编辑器根据需要自己选择，很多平台都已经支持直接用 Markdown 进行编辑了。**

| 编辑器                                      | 支持平台                        | 支持导出格式                                   |
| ---------------------------------------- | --------------------------- | ---------------------------------------- |
| [**Typora**](http://www.typora.io/) <br/> 正在开发， 界面简洁， 对 HTML 语法支持较弱， 支持导出文件类型较多。 | Mac、<br> Windows、<br> Linux | HTML、 <br>Word(.docx)、 <br>PDF、 <br>LaTex 等 |
| [**EME**](https://eme.moe/) <br/> 正在开发， 界面简洁，对 HTML 语法支持友好，支持导出文件类型较少。 | Mac、 <br>Windows、 <br>Linux | 仅 PDF                                    |
| [**Mou**](http://25.io/mou/) <br/> 停止开发， 界面简洁， 对 HTML 语法支持友好， 但支持导出文件类型较少。 | Mac                         | HTML、 <br>PDF                            |
| [**Sublime Text**](http://www.sublimetext.com/3) <br/> 神兵利器，需要安装插件才能使用， 相对比较麻烦， 适合高级魔法师。 | Mac、 <br>Windows、 <br>Linux | 多种格式(插件)                                 |
| [**Atom**](https://atom.io/) <br/> 神兵利器， 支持多种常见的编程语言， 如果仅仅是为了写 Markdown 不推荐安装。 | Mac、 <br>Windows、 <br>Linux | HTML、 <br>PDF(插件)                        |

****

## 快速入门

本文是为了帮助还不了解 Markdown 的魔法师有一个简单的认知，所以快速入门只说最基本的一些内容。



### 标题

通过在行首插入 1 到 6 个 `#` ，来定义对应的 1 到 6 阶 标题：

<table>
    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
  <tr>
    <td># 一级标题</td>
    <td><h1>一级标题</h1></td>
  </tr>
  <tr>
    <td>## 二级标题</td>
    <td><h2>二级标题</h2></td>
  </tr>
  <tr>
    <td>### 三级标题</td>
    <td><h3>三级标题</h3></td>
  </tr>
</table>



### 分段

在 Markdown 中段落由一行或者多行文本组成，相邻的两行文字会被视为同一段落，如果存在空行则被视为不同段落( Markdown 对空行的定义是看起来是空行就是空行，即使空行中存在 `空格` `TAB` `回车` 等不可见字符，同样会被视为空行)。

<table>
    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
    <tr>
      <td>
      第一行<br/>
      相邻被视为同一段落。
      </td>
      <td>
        <p>
        第一行 相邻被视为同一段落。
        </p>
      </td>
    </tr>
    <tr>
      <td>
      第一行<br/>
      <br/>
      两行之间存在空行，视为不同段落。
      </td>
      <td>
        <p>
        第一行
        </p>
        <p>
        两行之间存在空行，视为不同段落。
        </p>
      </td>
    </tr>
</table>



### 链接和图片

<table style="word-break:break-all;">
    <tr>
      <th width="200">Markdown</th>
      <th width="200">预览</th>
    </tr>
  <tr>
    <td>[GcsSloop](http://www.gcssloop.com)</td>
    <td><a href="http://www.gcssloop.com" target="_blank">GcsSloop</a></td>
  </tr>
  <tr class="alternate">
    <td>![GcsSloop Blog](http://www.gcssloop.com/assets/siteinfo/friends/gcssloop.jpg)</td>
    <td><img src="http://www.gcssloop.com/assets/siteinfo/friends/gcssloop.jpg" alt="GcsSloop Blog" /></td>
  </tr>
</table>



知道了上面这些内容就已经算是入门了，可以用 Markdown 进行快乐的写作，如果想了解更多语法相关内容，请看下一篇 [Markdown实用技巧－基础语法][markdown-grammar]


## 参考资料

[Markdown－基础语法](http://www.markdown.cn/)


## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>

[markdown-grammar]: /markdown/markdown-grammar	"Markdown实用技巧－语法"



