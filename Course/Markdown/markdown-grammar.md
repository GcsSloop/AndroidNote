# Markdown 基础语法

为保证语法兼容性，本文只介绍基础语法，扩展语法等其它内容，会在后续的文章中单独介绍。  
**注意：所有的标记符号均使用英文，中文无效。**

****

## 标题

Markdown 支持多种标题格式。

利用 `=` (等号)和 `-`(减号)可以定义一级标题和二级标题，(任何数量的 `=` 和 `-` 都有效果) ：

<table>
    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
    <tr>
      <td>一级标题<br/>====</td>
      <td><h1>一级标题</h1></td>
    </tr>
    <tr>
      <td>二级标题<br/>----</td>
      <td><h2>二级标题</h2></td>
    </tr>
</table>

通过在行首插入 1 到 6 个 # ，来定义对应的 1 到 6 阶 标题，个人推荐使用这种，例如：

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
  <tr>
    <td>#### 四级标题</td>
    <td><h4>四级标题</h4></td>
  </tr>
  <tr>
    <td>##### 五级标题</td>
    <td><h5>五级标题</h5></td>
  </tr>
  <tr>
    <td>###### 六级标题</td>
    <td><h6>六级标题</h6></td>
  </tr>
</table>

****

## 段落和换行

在 Markdown 中段落由一行或者多行文本组成，相邻的两行文字会被视为同一段落，如果存在空行则被视为不同段落( Markdown 对空行的定义是看起来是空行就是空行，即使空行中存在 `空格` `TAB` `回车` 等不可见字符，同样会被视为空行)。

Markdown支持段内换行，如果你想进行段落内换行可以在上一行结尾插入两个以上的空格后再回车。

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
      第一行[空格][空格]<br/>
      上一行结尾存在两个空格，段内换行
      </td>
      <td>
        <p>
        第一行<br/>
        上一行结尾存在两个空格，段内换行。
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

******

## 强调

删除线的 `~` 符号一般位于键盘左上角位置。

<table>
    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
    <tr>
      <td>*倾斜*</td>
      <td>
        <i>倾斜</i>
      </td>
    </tr>
    <tr>
      <td>**粗体**</td>
      <td><b>粗体</b></td>
    </tr>
    <tr>
      <td>~~删除线~~</td>
      <td><s>删除线</s></td>
    </tr>
    <tr>
      <td>
        &gt; 引用
      </td>
      <td>
        <blockquote>引用</blockquote>
      </td>
    </tr>
</table>

******

## 链接和图片

为了规避某些平台的防盗链机制，图片推荐使用图床，否则在不同平台上发布需要重新上传很麻烦的，图床最好选大平台的图床，一时半会不会倒闭的那种，个人目前主要用的是微博图床 [Chrome插件－微博图床](https://chrome.google.com/webstore/detail/%E6%96%B0%E6%B5%AA%E5%BE%AE%E5%8D%9A%E5%9B%BE%E5%BA%8A/fdfdnfpdplfbbnemmmoklbfjbhecpnhf?hl=zh-CN) 以及 [Alfred脚本－微博图床](https://imciel.com/2016/07/17/weibo-picture-upload-alfred-workflow/)

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

******

## 列表

无序列表前面可以用 `*` `+` `-` 等，结果是相同的。  
有序列表的数字即便不按照顺序排列，结果仍是有序的。


<table>

    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
  <tr>
    <td>
      * 项目<br />
      * 项目<br />
      * 项目<br />
      &nbsp;&nbsp;* 子项目<br />
      &nbsp;&nbsp;* 子项目<br />
      * 项目
    </td>
    <td>
      <ul>
        <li>项目</li>
        <li>项目</li>
        <li>项目
          <ul>
            <li>子项目</li>
            <li>子项目</li>
          </ul>
        </li>
        <li>项目</li>
      </ul>
    </td>

  </tr>
  <tr class="alternate">
    <td>
      1. 项目<br />
      2. 项目<br />
      3. 项目<br />
      &nbsp;&nbsp;1. 子项目</span><br />
      &nbsp;&nbsp;2. 子项目<br />
      4. 项目
    </td>
    <td>
      <ol>
        <li>项目</li>
        <li>项目</li>
        <li>项目
          <ol>
            <li>子项目</li>
            <li>子项目</li>
          </ol>
        </li>
        <li>项目</li>
      </ol>
    </td>
  </tr>
</table>

****

## 下划线和特殊符号

由于 Markdown 使用一些特殊符号进行标记，当我们想要在文档中使用这些特殊符号并防止被 Markdown 转换的时候，可以使用 `\` (转义符) 将这些特殊符号进行转义。

<table>
    <tr>
      <th>Markdown</th>
      <th>预览</th>
    </tr>
  <tr>
    <td>
      在一行中用三个以上的星号、减号、下划线来建立一个分隔线<br />
      ---
    </td>
    <td>
      <hr />
    </td>
  </tr>
  <tr class="alternate">
    <td>可以利用反斜杠(转义字符)来插入一些在语法中有特殊意义的符号<br />
        \*Hi\*
    </td>
    <td>*Hi*</td>
  </tr>
</table>

****

## 代码

### 1.行内代码

行内代码可以使用反引号来标记(反引号一般位于键盘左上角，要用英文):

```
一句话 `行内代码` 一句话。
```

**预览:**

一句话 `行内代码` 一句话。

### 2.多行代码

多行代码使用 3 个反引号来标记(反引号一般位于键盘左上角，要用英文) ，在第一个 `｀｀｀` 后面可以跟语言类型，没有语言类型可以省略不写:

```
​``` java
// 我是注释
int a = 5;
​```
```

**预览:**

``` java
// 我是注释
int a = 5;
```



****

## 表格

**扩展的 Markdown 支持手写表格**，格式也非常简单，第二行分割线部分可以使用 `:` 来控制内容状态。  
**注意，Markdown 标准(原生)语法中没有表格支持，但现在多数平台已经支持了该语法，如 GitHub，CSDN，简书 等均支持，所以写在这里:**

**Markdown：**

```
| 默认  |   靠右 |  居中  | 靠左   |
| ---- |  ---: | :--:  | :---  |
| 内容  |   内容 |  内容  | 内容   |
| 内容  |   内容 |  条目  | 内容   |
```

**预览：**

| 默认   |   靠右 |  居中  | 靠左   |
| ---- | ---: | :--: | :--- |
| 内容   |   内容 |  内容  | 内容   |
| 内容   |   内容 |  条目  | 内容   |


## 参考资料

[Markdown－语法说明](http://www.markdown.cn/)


## About Me

### 作者微博: <a href="http://weibo.com/GcsSloop" target="_blank">@GcsSloop</a>

<a href="http://www.gcssloop.com/info/about" target="_blank"><img src="http://ww4.sinaimg.cn/large/005Xtdi2gw1f1qn89ihu3j315o0dwwjc.jpg" width="300" style="display:inline;" /></a>
