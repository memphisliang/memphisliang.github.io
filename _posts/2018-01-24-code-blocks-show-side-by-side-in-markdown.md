---
layout: post
title: 网页中代码块并排显示
---

# 网页中代码块并排显示

最近在撰写博客文章时遇到一个问题，就是在文章中编写代码示例的时候，看到别的网站把对应的代码示例并排显示在一起，显得十分方便，如下：

~~~ example
# foo
## foo
### foo
#### foo
##### foo
###### foo
<<<<<<<<<<>>>>>>>>>>
<h1>foo</h1>
<h2>foo</h2>
<h3>foo</h3>
<h4>foo</h4>
<h5>foo</h5>
<h6>foo</h6>
~~~

可以看到，左边的是Markdown代码，而右边显示的是对应的HTML代码，这在某些场合是十分方便直观的。

那么这里有什么问题呢？主要是笔者是使用Markdown来编写文章的，要实现这样的效果，最直接的方法是在Markdown文档代码块对应的位置插入大量的HTML代码，但这样就破坏了Markdown的初衷——在文字格式下直观的看到结果，而且代码重复出现也不好，所以这篇文章的目的就是美观的实现代码的并排显示效果。

## 解决方案

在解决方法中会使用到一种Markdown语法，在这里先介绍这种语法的用途。

在CommonMark的Markdown语法中，只要在代码块开始处加上描述字符串，就可以在生成的HTML代码中加上对应的`class`属性，属性值为`language-`前缀后接描述字符串：

~~~~~~~~~~~ example
``` python
if TRUE:
    print("Hello, world!")
```
<<<<<<<<<<>>>>>>>>>>
<pre><code class="language-python">if TRUE:
    print("Hello, world!")
</code></pre>
~~~~~~~~~~~

根据这种语法，可以给代码块中`code`元素的`class`属性设置为`language-example`，然后就是对这个块进行处理了。

### 解决思路

笔者的思路是这样的，首先把要并排显示的代码内容写在一个代码块里，中间用字符串（`<<<<<<<<<<>>>>>>>>>>`）来分隔它们（后期使用这个字符串来区分两段代码，可以使用其他字符串来分隔，但要修改JavaScript代码），之后就是使用JavaScript脚本来对这个代码进行处理了，具体例子如下：

```
~~~ example
# foo
## foo
### foo
<<<<<<<<<<>>>>>>>>>>
<h1>foo</h1>
<h2>foo</h2>
<h3>foo</h3>
~~~
```

然后就是对这个代码进行处理，具体步骤如下：

+ 使用JavaScript脚本找到`class`属性为`language-example`的`<code>`块。
+ 获取`<code>`中的内容，然后删除该`<code>`块。
+ 根据文本中的`·`符号分隔成两段文本（代码）。
+ 在`<pre>`块中新增两个`<code>`块，分别填入两段文本。
+ 对两个`<code>`块设置CSS规则，使其并排显示。

要注意的是，虽然上面的步骤中写明要删除原本的`<code>`块，但实现中不会删除，而是在这个块后面新增一个`<code>`来存放第二段代码，这样做的原因是为了实现方便。

笔者在Markdown文档的最后添加JavaScript代码，这样在转换为HTML文档之后，可以使JavaScript脚本只对这个HTML文档生效，而且在阅读Markdown文档时不会让Markdown和JavaScript代码混淆在一起，影响阅读。

### 实现

具体的JavaScript代码如下：

```
// 分隔字符串，后面要加'\n'
split_str = "<<<<<<<<<<>>>>>>>>>>" + '\n'
// 获取所有需要处理的<code>块
codes = document.getElementsByClassName("language-example");

for(i = 0; i < codes.length; i++) {
    // 获取两段代码，之后用来设置两个<code>的文本。
    text_array = codes[i].innerText.split(split_str);
    text1 = text_array[0];
    text2 = text_array[1];
    // 在<pre>块中设置两个<code>块来放置两段代码。
    codes[i].innerText = text1;
    // 生成一个新的<code>块。
    new_code = document.createElement("code");
    new_code.innerText = text2;
    codes[i].parentElement.appendChild(new_code);
    codes[i].parentElement.className += "clear-float"; //清除浮动
}
```

上面的JavaScript代码是用来把每一个带`language-example`的`class`属性的`<code>`块分隔成两个，记住，只有带有`example`描述字符串的代码块会被这样处理，其他的代码块不会收到影响。

然后是给这两个`<code>`块设置CSS规则：

```
style_str = `
code.language-example,
code.language-example + code {
    width: 50%;
    display: block;
    margin: 0;
    padding: 0;
    float: left;
    background-color: #C9CACE;

    white-space: pre-wrap;       /* css-3 */
    white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
    white-space: -pre-wrap;      /* Opera 4-6 */
    white-space: -o-pre-wrap;    /* Opera 7 */
    word-wrap: break-word;       /* Internet Explorer 5.5+ */
}

code.language-example {
    background-color: #D3E1E4;
}
.clear-float {
    overflow: auto;
}`;

style_node = document.createElement("style");
style_node.innerText = style_str;
document.getElementsByTagName("head")[0].appendChild(style_node);
```

上面的CSS规则中的一系列`white-space`是使代码内容超过容器宽度的时候，会自动换行，详细作用可以看这篇博客[《英文单词断行问题》]。

### 结果

全部代码如下：

```
<script>
split_str = "<<<<<<<<<<>>>>>>>>>>" + '\n'
codes = document.getElementsByClassName("language-example");
for(i = 0; i < codes.length; i++) {
    text_array = codes[i].innerText.split(split_str);
    text1 = text_array[0];
    text2 = text_array[1];
    codes[i].innerText = text1;
    new_code = document.createElement("code");
    new_code.innerText = text2;
    codes[i].parentElement.appendChild(new_code);
    codes[i].parentElement.className += "clear-float";
}

style_str = `
code.language-example,
code.language-example + code {
    width: 50%;
    display: block;
    margin: 0;
    padding: 0;
    float: left;
    background-color: #C9CACE;
    white-space: pre-wrap;       /* css-3 */
    white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
    white-space: -pre-wrap;      /* Opera 4-6 */
    white-space: -o-pre-wrap;    /* Opera 7 */
    word-wrap: break-word;       /* Internet Explorer 5.5+ */
}
code.language-example {
    background-color: #D3E1E4;
}
.clear-float {
    overflow: auto;
}`;
style_node = document.createElement("style");
style_node.innerText = style_str;
document.getElementsByTagName("head")[0].appendChild(style_node);
</script>
```

在Markdown文档的最后添加上面的代码，然后使用Markdown解释器转换成HTML文档，在打开就可以看到结果。

例如，Markdown文档如下：

```
~~~ example
# foo
## foo
### foo
<<<<<<<<<<>>>>>>>>>>
<h1>foo</h1>
<h2>foo</h2>
<h3>foo</h3>
~~~

// 这里有一大段的代码，就不贴出来了。
```

然后是最后结果：

![最终结果]

注意：笔者只在Firefox-57.0.4和Chrome63.0.3239这两个浏览器中进行了测试，不能保证其他浏览器中不会出现问题，希望大家见谅。

### 其他问题

![空白节点导致不并排显示]

在一些浏览器中可能会出现代码没有并排显示的问题，这有可能是因为在两个`<code>`之间有其他的空白节点导致的，例如Firefox浏览器就可能在两者之间插入一个`#text`的空白节点，建议在浏览器的控制台中使用JavaScript来查看是否有这样的问题。





<!--------------- 链接 --------------->

[《英文单词断行问题》]: https://blog.brain1981.com/706.html

[最终结果]: https://i.loli.net/2018/01/24/5a67fcd63dff9.jpg "最终结果"

[空白节点导致不并排显示]: https://i.loli.net/2018/01/24/5a67fc92478a8.jpg "空白节点导致不并排显示"

<!--------------- 脚本 --------------->

<script>
split_str = "<<<<<<<<<<>>>>>>>>>>" + '\n'
codes = document.getElementsByClassName("language-example");
for(i = 0; i < codes.length; i++) {
    text_array = codes[i].innerText.split(split_str);
    text1 = text_array[0];
    text2 = text_array[1];
    codes[i].innerText = text1;
    new_code = document.createElement("code");
    new_code.innerText = text2;
    codes[i].parentElement.appendChild(new_code);
    codes[i].parentElement.className += "clear-float";
}

style_str = `
code.language-example,
code.language-example + code {
    width: 50%;
    display: block;
    margin: 0;
    padding: 0;
    float: left;
    background-color: #C9CACE;
    white-space: pre-wrap;       /* css-3 */
    white-space: -moz-pre-wrap;  /* Mozilla, since 1999 */
    white-space: -pre-wrap;      /* Opera 4-6 */
    white-space: -o-pre-wrap;    /* Opera 7 */
    word-wrap: break-word;       /* Internet Explorer 5.5+ */
}
code.language-example {
    background-color: #D3E1E4;
}
.clear-float {
    overflow: auto;
}`;
style_node = document.createElement("style");
style_node.innerText = style_str;
document.getElementsByTagName("head")[0].appendChild(style_node);
</script>
