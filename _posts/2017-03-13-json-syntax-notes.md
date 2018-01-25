---
layout: post
title: JSON语法笔记
---

# JSON语法笔记

JSON即是JavaScript对象表示法，就是用对象的形式来表示数据。学习JSON不需要JavaScript的基础，当然有也更好，不过我认为单纯把JSON看成一门独立的数据交换格式来看更好，不然与JavaScript混淆的话会很痛苦，因为在一些细节上JSON与JavaScript会有点不同。

这篇博文是我看[《JSON必知必会》]这本书的笔记，提取了JSON数据格式的重要语法，去除了我认为不太重要的部分，如JSON Scheme等的内容，但还是建议大家去看一看，补充一下自己的知识广度还是很好的。

## JSON基本语法

JSON是用对象的形式来表示数据，JSON中的对象是指用`{}`包裹起来的一个或多个键值对，多个键值对之间用逗号（`,`）分隔。

键值对是一个键名和值组成的序列，键名和值之间用冒号（`:`）分隔。

```
{
	"animal": "cat",
	"color": "orange"
}
```

+  键名必须用双引号（`"`）包裹起来，不能没有，也不能使用单引号（`'`）。
+  对象的最后一个键值对末尾不要使用逗号（`,`），因为JSON认为逗号（`,`）是一个新部分的开始，如另一个键值对，但后面却什么都没有，就会报错。
+  键名最好只使用`A~Z`、`a~z`和`0~9`的组合，因为键名载入系统后会成为“属性”，应该尽量减少可能发生的错误。

## JSON中值的数据类型

JSON键值对中值的数据类型可以是以下类型：

### 字符串类型

```
{ "animal": "cat" }
```

上面是一个简单的JSON例子，其中`"cat"`就是一个字符串类型，字符串类型的两端必须被双引号（`"`）包裹，也建议大家在JSON中用双引号（`"`）取代单引号（`'`），这样更不容易犯语法错误。

JSON中字符串可以由任何Unicode字符构成，单引号（`'`）也可以包含在字符串中，所以下面的例子也是合法的：

```
{ "text": "JSON是一个常用的'数据格式'" }
```

那如果我们的字符串值中包含双引号（`"`）呢？

```
{ "promo": "Say "Hello world!"." }  # 错误
```

由于值里面有双引号，解析器在读取第一个双引号之后，会把`Hello`前面的双引号当成字符串的结尾，然后解析器发现后面还有许多不属于任何一个键值对的字符`Hello world!"."`，就会报错。

如果要在字符串中使用双引号，就要在双引号（`"`）前加入反斜杠（`\`）来对双引号进行转义，如：

```
{ "promo": "Say \"Hello world!\"." }
```

反斜杠告诉解析器这个双引号并不意味着字符串的结束。字符串也会如期输出。

双引号也不是JSON中唯一需要转义的字符，因为我们使用反斜杠来转义，我们也要对字符串中使用的反斜杠进行转义——在反斜杠前加入一个反斜杠`\\`。

```
{ "location": "C:\\Program Files" }
```

在Python语言中解析上述json，看看会发生什么，先创建一个`test.txt`文件，里面写入json：

```
$ cat test.txt
{ "location": "C:\\Program Files" }
```

然后使用Python的json标准库进行解析，结果如下：

```
>>> import json
>>>
>>> with open("test.txt", "r") as f:
...     text = ''.join(f.readlines())
...     obj = json.loads(text)
...     obj["location"]
...     print(obj["location"])
...
'C:\\Program Files'
C:\Program Files
```

可以看到，在JSON中输入的字符串其实不是最终的字符串，字符串还要经过转义之后才得到最终的字符串，所以在手动处理JSON数据时必须要十分小心，也建议使用库来进行JSON数据的读写操作，这样就不用在意语法问题了。

除了双引号和反斜杠，还要转义以下字符：

+  \/  （正斜杠）
+  \b  （退格符）
+  \f  （换页符）
+  \t  （制表符）
+  \n  （换行符）
+  \r  （回车符）
+  \u 后面跟十六进制字符  （如笑脸表情 \u263A）

如果想输出一下字符串：

```
>>> print(obj["story"])
\t Once upon a time, in a far away land \n there lived a princess.
```

JSON中的写法如下：

```
{ "story": "\\t Once upon a time, in a far away land \\n there lived a princess." }
```

### 2.2 数字类型

数字是一种常见的数据类型。库存数目、金额、经度、纬度以及地球质量等都可以用数字来表示，如下：

```
{
	"widgetInventory": 289,
	"sadSavingsAccount": 22.59,
	"seattleLatitude": 47.606209,
	"seattleLongitude": -122.332071,
	"earthsMass": 5.97219e+24
}
```

### 2.3 布尔类型

在计算机编程中，布尔类型也是十分常用的，在JSON中布尔类型只有两个值：true 和 false，注意，两个值都必须是小写字母，否则会报错。

```
# 错误

{ "breadWithLunch": True }

{ "breadWithLunch": TRUE }

# 正确

{ "breadWithLunch": true }
```

### 2.4 null类型

有时候用0来表示某样东西不存在可能不太合适，因为0是一个数值，本质上是在计数。这时使用null类型就比较合适了。null类型只有`null`一个值，null类型和布尔类型一样只能是小写字母表示。

```
{
	"animal": "snake",
	"footCount": null
}
```

### 2.5 对象类型

在JSON中，键值对的值可以是对象，这意味着对象可以多重嵌套：

```
{
	"person": {
		"name": "Lindsay Bassett",
		"heightInInches": 66,
		"head": {
			"hair": {
				"color": "light blond",
				"length": "short",
				"style": "A-line"
			},
			"eyes": "green"
		}
	}
}
```

### 2.6 数组类型

键值对中的值可以是数组类型，数组用`[]`符号来包裹，每个数组元素用逗号`,`间隔，数组元素可以是以上所有的数据类型，也可以是数组本身，即多维数组。

```
{
	"test": [
		{
			"question": "The sky is blue.",
			"answer": true
		},
		{
			"question": "The earth is flat.",
			"answer": false
		},
		{
			"question": "A cat is a dog.",
			"answer": false
		}
	]
}
```

## 使用库函数转换JSON

现在主流的编程语言通常都提供处理JSON数据格式的库，建议大家使用这些库函数来处理JSON对象，这样就可以减少JSON语法错误的发生。



<!-------------- 链接 ------------------------->

[《JSON必知必会》]: https://book.douban.com/subject/26789960/
