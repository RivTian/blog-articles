---
title: "如何使用 Python 進行字串格式化"
date: 2020-09-27T14:17:55+08:00
draft: false
tags:
  - Python
topics:
  - 
---

![](https://static.coderbridge.com/img/techbridge/images/kdchang/python101/mac-cover.jpg)

## 前言：

> Python有几种方法可以显示程序的输出；数据可以以人类可读的形式打印出来，或者写入文件以供将来使用。

在开发应用程式时我们往往会需要把变数进行字串格式化，也就是说把字串中的变数替换成变量值。事实上，在 Python 中有许多方式可以进行，其中最常见的有四种方式：

1. 旧式字串格式化
2. 新式字串格式化
3. 字串插值
4. 样板字串

字串插值是在 Python 3.6 之后有支援的方法，若是你的版本是在 Python 3.6 之后的话建议可以使用。若是需要让使用者可以输入变数来转换成字串格式化的话，建议可以使用样板字串来避免一些资讯安全上的问题。

以下就上述提到的四种方法来各自说明其特色和使用方式：

## 旧式字串格式化（%）

相对于Python 版本之后推荐使用的新式字串格式化，旧式版本使用 `%` 运算子来进行字串格式化，若是有C 语言撰写经验的读者或许会觉得的似曾相似（是不是有点像 printf ？） 。使用 `%` 格式是告诉 Python 直译器要在那边替换文字 text 并使用字串呈现。这就是所谓的旧式字串格式化（%s 是以字串输出，%f 是以浮点数输出、%d 是以十进位整数输出）：

```python
text = 'world'
print('hello $s' % text)
# hello world
```

若是希望把內容轉成十六進位的話可以使用：

```python
print('%x' % 23)
# 17
```

若是有多個變數要替換則使用 tuple 傳遞需要替代的內容值：

```python
print('hello %s %s' % ('world', 'go'))
# hello world go
```

## 新式字串格式化（format()）

在Python3以后，开始引入新串格式化，也就是使用 ==format（）==函式来让字串格式化，其功能和旧式格式化相差无几，但主要是舍去 `％` 让字串格式化使用上可以更加 正常，正常，预期性也相对提升。

一般基本用法：

```python
text = 'world'
print('hello {}'.format(text))
# hello world
```

也可以使用名称来指定变数变换顺序：

```py
name = 'Jack'
text = 'world'

print('hello {name}, hello {text}'.format(name=name, text=text))
# hello Jack, hello world
```

若是希望把内容转成十六进位的话可以使用 format spec 在 `{}` 新增 `:x`：

## 更漂亮的输出格式：字串插值（Formatted String Literal）

虽然已经有了新式字串格式化，而在Python 3.6又添加了格式字串字面值（Formatted String Literal）此一作法可以把Python运算式嵌入在字串常数中。
眼尖的读者可能会发现，咦，怎么跟隔壁棚的 **JavaScript ES6**字串模版有点像呀？

现在我们来看一下一般的使用方式：

```python
text = 'world'
print(f'Hello, {text}')
```

新的字串插值语法相当强大的点是，可以在里面嵌入**任何** Python的运算式，表示来说，我们想要呈现整数相加：

```python
x = 10
y = 27

print(f'x + y = {x + y}')
# 37
```

同样，若是希望把内容转成十六进位的话可以使用 format spec 在 `{}` 新增 `:x`：

```python
print('{:x}'.format(23))
# 17
```

读者可能会觉得很字串插值神奇，但实际上其背后原理是由Python语法解析器把f-string字串插值格式字串转成一连串的字串常数和运算式，最后结合成最终的字串 。

```python
def hello(text, name):
    return f'hello {text}, hello {name}'

# 实际上Python会把它变成字串常数和变数（过程中有优化）

def hello(text, name):
    return 'hello ' + text + ', hello' + name
```



## 模板字串（Template String）

模板字串（Template String）机制相对简单，也比较安全。

以下是一般的使用情境，需要从Python内建模组string 引入：

```python
from string import Template

text = 'world'
t = Template('hello, $text')
t.substitute(text=text)
# hello, world
```

然而若是希望把内容变成成十六进位的话需要自己使用hex函式自己转换：

```python
from string import Template

number = 23
t = Template('hello, $number')
t.substitute(number=hex(number))
# hello, 0x17
```

由于其他的字串格式化功能强大，所以反而会造成恶意用户输入变数替换成字串时造成不可预期的错误（一般来说使用者的输入都是不可信的，要进行过滤）。

体现为恶意使用者可能可以透过字串格式的恶意输入来获取敏感资讯（例如：密码，令牌，金钥等）？

```python
SECRET_TOKEN = 'my-secret-token'

# Error func
class Error:
    def __init__(self):
        pass

err = Error()
malicious_input = '{error.__init__.__globals__[SECRET_TOKEN]}'
malicious_input.format(error=err)
# my-secret-token
```

没想到，透过字串格式的方式竟然可以透过 `__globals__` 字典检索我们的SECRET_TOKEN，若是一不留神，很可能机密资料就泄漏出去。此时若是使用范本String发生错误，是比较安全的选项 ：

```python
from string import Template

SECRET_TOKEN = 'my-secret-token'

# Error func
class Error:
    def __init__(self):
        pass

err = Error()
malicious_input = '${error.__init__.__globals__[SECRET_TOKEN]}'
t = Template(malicious_input)
t.substitute(error=err)
# ValueError: Invalid placeholder in string: line 1, col 1
```

## 总结

虽然Python相信是能用简单唯一的方式来完成任务，而字串格式化却有多种方式，也各有其优缺点，其本身或许在于版本不同变迁所致。所以你有可能在公司内部 专案不同专案看到使用不同的字串格式化方式，若是看到同一个专案使用不同字串格式化方式也不要混淆。

一般情况我们会根据不同Python版本和使用情境去使用不同字串格式化方式，例如：若是使用Python 3.6之后的话建议可以使用字串插值，若版本比3.6旧，则使用新式字串格式化（format （））。若是需要让使用者可以输入变数来转换成字串格式化的话，建议可以使用样板字串来避免一些资讯安全上的问题。

## 參考文件

1. [python string — Common string operations](https://docs.python.org/3/library/string.html)
2. [(那些過時的) Python 字串格式化以及 f-string 字串格式化](https://blog.louie.lu/2017/08/08/outdate-python-string-format-and-fstring/)
3. [字串格式化](https://openhome.cc/Gossip/Python/StringFormat.html)
4. [Python String Formatting Best Practices](https://realpython.com/python-string-formatting/)
5. [Python 字串格式化教學與範例](https://officeguide.cc/python-string-formatters-tutorial/)
6. [A Quick Guide to Format String in Python](https://www.techbeamers.com/python-format-string-list-dict/)
7. [菜鸟教程python格式化问题](https://www.runoob.com/python/att-string-format.html)

