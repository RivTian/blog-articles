---
title: "Web开发初探"
date: 2020-09-14T14:07:58+08:00
draft: false
tags:
  - Web
topics:
  - blog
---

## 一、Web开发介绍

我们看到的网页通过代码来实现的 ，这些代码由浏览器解释并渲染成你看到的丰富多彩的页面效果。 这个浏览器就相当于Python的解释器，专门负责解释和执行(渲染）网页代码。

写网页的代码是专门的语言， 主要分为Hmtl 、 CSS 和 JavaScript​, 被称为网页开发三剑客，分别作用如下：

#### Html

超文本标记语言（英语：HyperText Markup Language，简称：HTML）是一种用于创建网页的标准标记语言。

主要负责编写页面架构，有点像建房子时，造的毛坯房。

#### CSS

CSS (Cascading Style Sheets) 用于渲染HTML元素标签的样式。

让你的网页样式变的丰富多彩起来，可以改变字体、颜色、排列方式、背景颜色等

相当于给你的毛坯房做装修



#### JavaScript

网页脚本语言，可以让你的网页动起来，比如一张图片鼠标放上去自动变大、一个按钮自动变色、提交表单时少填或填错了字段会提示报错等，都是JavaScript实现的。

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122754.png)

以上3个 组件 是做网站开发都必须掌握的技能 ，我们接下来依次体验下~吧



## 二、HTML

### 2.1 HTML简介

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>路飞学城</title>
  </head>
  <body>

    <h1>我的第一个标题</h1>
    <p>我的第一个段落。</p>

  </body>
</html>
```

- `<!DOCTYPE html>`声明为 HTML5 文档
- `<html>`元素是 HTML 页面的根元素
- `<head>` 标签用于定义文档的头部，它是所有头部元素的容器。<head> 中的元素可以引用脚本、指示浏览器在哪里找到样式表、提供元信息等等。
- `<meta>`元素包含文档的元数据， 如定义网页编码格式为 **utf-8**、关键词啥的
- `<title>`元素里描述了文档的标题
- `<body>`元素包含了可见的页面内容
- `<h1>`元素定义一个大标题
- `<p>`元素定义一个段落

**注：**在浏览器的页面上使用键盘上的 F12 按键开启调试模式，就可以看到组成标签。

### 2.2  什么是HTML? 

HTML 是用来描述网页的一种语言。

- HTML 指的是超文本标记语言: **H**yper**T**ext **M**arkup **L**anguage
- HTML 不是一种编程语言，而是一种**标记**语言
- 标记语言是一套**标记标签** (markup tag)
- HTML 使用标记标签来**描述**网页
- HTML 文档包含了HTML **标签**及**文本**内容
- HTML文档也叫做 **web 页面**

#### HTML 标签

HTML 标记标签通常被称为 HTML 标签 (HTML tag)。

- HTML 标签是由*尖括号*包围的关键词，比如 <html>
- HTML 标签通常是**成对出现**的，比如`<b>`  和 `</b>`,标签对中的第一个标签是开始标签，第二个标签是结束标签

### 2.3 HTML网页结构

下面是一个可视化的HTML页面结构：

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122831.png)

注意： 只有 \<body> 区域 (白色部分) 才会在浏览器中显示。

## 三、HTML常用元素入门

### 3.1 HTML标题

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122841.png)

### 3.2 段落

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122848.png)

### 3.3 超链接

```html
<body>
	<a href="https://www.luffycity.com">这是一个链接使用了 href 属性</a>
</body>
```

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122855.png)

```html
<a href="https://www.luffycity.com" target="_blank" >访问路飞学城</a>
```

`target="_blank"`会打开一个新窗口

### 3.4 显示图片

```html
<img src="black_girl.jpg" width="600" height="500" >
```

### 3.5 HTML表格

想在页面上显示这种表格怎么做？

| First Name | Last Name | Points |
| :--------- | :-------- | :----- |
| Jill       | Smith     | 50     |
| Eve        | Jackson   | 94     |
| John       | Doe       | 80     |
| Adam       | Johnson   | 67     |



#### 先做个最简单的

```html
<table border="1">
    <tr>
        <td>row 1, cell 1</td>
        <td>row 1, cell 2</td>
    </tr>
    <tr>
        <td>row 2, cell 1</td>
        <td>row 2, cell 2</td>
    </tr>
</table>
```

输出

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122920.png)

#### 表格的表头

表格的表头使用 \<th> 标签进行定义。

```html
<table border="1">
    <tr>
        <th>Header 1</th>
        <th>Header 2</th>
    </tr>
    <tr>
        <td>row 1, cell 1</td>
        <td>row 1, cell 2</td>
    </tr>
    <tr>
        <td>row 2, cell 1</td>
        <td>row 2, cell 2</td>
    </tr>
</table>
```



大多数浏览器会把表头显示为粗体居中的文本

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122924.png)

可以加上边距

```html
<table border="1" cellpadding="10">
```

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122927.png)

### 3.6 列表

分为有序列表 和 无序列表

#### 有序列表

```html
<ol>
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ol>

<ol start="50">
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ol>
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122932.png)

#### 无序列表

```html
<ul>
  <li>Coffee</li>
  <li>Tea</li>
  <li>Milk</li>
</ul>
```

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122937.png)

#### 嵌套列表

```html
<ul>
  <li>Coffee</li>
  <li>Tea
    <ul>
      <li>Black tea</li>
      <li>Green tea</li>
    </ul>
  </li>
  <li>Milk</li>
</ul>
```



![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122940.png)



### 3.7 div区块元素

#### HTML 区块元素

大多数 HTML 元素被定义为**块级元素**或**内联元素**。

块级元素在浏览器显示时，通常会以新行来开始（和结束）。

实例: \<h1>,\<p>, \<ul>, \<table>



#### HTML 内联元素

内联元素在显示时通常不会以新行开始。

实例: \<b>, \<td>, \<a>, \<img>



#### HTML `<div>` 元素

HTML \<div> 元素是块级元素，它可用于组合其他 HTML 元素的容器。

`<div>` 元素没有特定的含义。如果与 CSS 一同使用，\<div> 元素可用于对大的内容块设置样式属性。

`<div>` 元素的另一个常见的用途是文档布局。它取代了使用表格定义布局的老式方法。使用 \<table> 元素进行文档布局不是表格的正确用法。



#### `<span>` 元素

HTML \<span> 元素是内联元素，可用作文本的容器

\<span> 元素也没有特定的含义。当与 CSS 一同使用时，\<span> 元素可用于为部分文本设置样式属性。



### 3.8 CSS样式初识

CSS (Cascading Style Sheets) 用于渲染HTML元素标签的样式。

CSS 可以通过以下方式添加到HTML中:

- 内联样式- 在HTML元素中使用"style" **属性**
- 内部样式表 -在HTML文档头部 \<head> 区域使用\<style> **元素** 来包含CSS
- 外部引用 - 使用外部 CSS **文件**



#### 3.8.1 内联样式

当特殊的样式需要应用到个别元素时，就可以使用内联样式。 使用内联样式的方法是在相关的标签中使用样式属性。样式属性可以包含任何 CSS 属性。以下实例显示出如何改变段落的颜色和左外边距。

```html
<p style="color:blue;margin-left:20px;">这是一个段落。</p>
```

##### 设置背景颜色

```html
<body style="background-color:yellow;">
  <h2 style="background-color:red;">这是一个标题</h2>
  <p style="background-color:green;">这是一个段落。</p>
</body>
```

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122951.png)

##### 设置字体, 字体颜色 ，字体大小

我们可以使用font-family（字体），color（颜色），和font-size（字体大小）属性来定义字体的样式:

```html
<h1 style="font-family:verdana;">一个标题</h1>
<p style="font-family:arial;color:red;font-size:20px;">一个段落。</p>
```



#### 3.8.2 内部样式表

当单个文件需要特别样式时，就可以使用内部样式表。你可以在\<head> 部分通过 \<style>标签定义内部样式表:

```html
<head>
  <style type="text/css">
    body {background-color:yellow;}
    p {color:blue;}
  </style>
</head>
```



CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明:

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914122957.jpg)

选择器通常是您需要改变样式的 HTML 元素。

每条声明由一个属性和一个值组成。

属性（property）是你希望设置的样式属性（style attribute）。每个属性有一个值。属性和值被冒号分开



#### 3.8.3 外部样式表

当样式需要被应用到很多页面的时候，外部样式表将是理想的选择。使用外部样式表，你就可以通过更改一个文件来改变整个站点的外观。

```html
<head>
	<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```



### 3.9 网页布局

网页布局主要使用div 

```html
<body>
<div id="header" style="background: darkorange; height: 100px;text-align: center ">
    Menu
</div>
<div id="content" style="background: aliceblue; height: 600px">

    <div id="left-menu" style="height: 100%; width: 15%; background: deepskyblue; float: left">
        菜单
        <ol>
            <li>HMTL</li>
            <li>CSS</li>
            <li>Javascript</li>
        </ol>
    </div>
    <div id="content-panel" style="background: aqua; height: 95%; width: 85%;float: left">
        <p>这里是真正写内容的地方。。。</p>
    </div>

</div>

<div style="background: greenyellow; height: 50px">
    footer
</div>
</body>
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123002.png)



### 3.10 Html 表单

当你想收集用户数据，提交到后台服务器时，你就可以用html 表单元素

表单元素分为 文本框、下拉列表、单选、复选 、按钮

#### 用户注册示例

```html
<form action="baidu_url">
     姓名: <input type="text" name="name"> <br>
     电话: <input type="number" name="phone"> <br>
      <button>注册</button>
</form>
```

一点击按钮，`<form>.....</form>` 表单里的数据都会被提交到`action="baidu_url"` 这个地址

#### 其它常用输入类型

```html
<body>
    <form action="baidu_url">

        姓名: <input type="text" name="name"> <br>
        电话: <input type="number" name="phone"> <br>
        姓别: <input type="radio" name="sex" value="男">男 <input type="radio" name="sex"  value="女">女  <br>
        爱好: <input type="checkbox" name="hobbie" value="girl">姑娘
             <input type="checkbox" name="hobbie" value="潜水">潜水
            <input type="checkbox" name="hobbie" value="Python">Python  <br>

        喜欢的姑娘类型：<br>
        <select name="gilr_type">
            <option value="1">胸大貌美大长腿</option>
            <option value="2">气质小可爱</option>
            <option value="3">妖娆性感</option>
            <optgroup label="按区域">
                <option value="4">欧美</option>
                <option value="5">日韩</option>
                <option value="6">河南开封</option>
            </optgroup>
        </select> <br>

        个人简介: <br>
        <textarea name="intro" placeholder="输入不省于50字的个人介绍" rows="3" cols="50" ></textarea> <br>

        输入密码：<input type="password" name="password"> <br>

        <button>注册</button>
    </form>

</body>
```

输出效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123007.png)

#### Fieldset 表单集合

```html
<form>
 <fieldset>
  <legend>Personalia:</legend>
  Name: <input type="text"><br>
  Email: <input type="text"><br>
  Date of birth: <input type="text">
 </fieldset>
</form>
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123011.png)

## 四、CSS样式基础

### 4.1 CSS id \ class 选择器

#### id选择器

id就像一个元素的身份证地址，可以在网页里，**唯一代表某个元素**，我们也可以通过这个id快速找到它对应的元素对象 

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123017.png)



输出

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123022.png)



#### class 类选择器

当你想给多个元素批量设置同样的一个样式的话，就可以使用类选择器

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123025.png)

输出

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123029.png)

### 4.2 直接通过元素名设置样式

若你想给页面所有的\<p> 或\<input>标签加上同样的样式，可以直接通过元素名批量设置

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123033.png)



**注意：id & class 选择器的样式优先级大于 元素名选择器**

### 4.3 组合选择器

为了测试效果，先准备好3层div

```html
<div id="layer1">
    <p>第1层</p>
    <div id="layer2">
        <p>layer 2</p>
        <h2>layer 2 h2</h2>
        <div id="layer3">
            <p>第3层</p>
            <h3>layer 3 h3</h3>
        </div>
    </div>

</div>
<p>我不在任何的div里</p>
```

设置好样式

```html
<head>
    <style type="text/css">
        #layer1 {
            height: 500px;
            padding: 30px;
            border: 1px dashed blue;
        }
        #layer2 {
            height: 400px;
            padding: 30px;
            border: 1px dashed red;
        }
        #layer3 {
            height: 300px;
            padding: 30px;
            border: 1px dashed black;
        }
    </style>
</head>
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123043.png)



#### 4.3.1 后代选择器

给指定元素的所有指定后代，设置样式

比如 ，我给上图第一层div下面所有的<p>标签设置颜色

```html
#layer1 p {
	color: red;
	background: yellow;
}
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123052.png)



#### 4.3.2 子元素选择器

又可称为儿子选择器，是指可给指定元素的下一层儿子元素设置格式 ，注意，只是儿子，孙子不管

给`layer1`的div的儿子设置样式

```html
#layer1 > p {
    color: red;
    background: yellow;
}
```

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123058.png)

#### 4.3.3 相邻兄弟选择器

可选择紧接在另一元素后的元素，且二者有相同父元素。

```html
#layer2 > p+h2 {
  color: red;
  background: yellow;
}
```

效果

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123103.png)

#### 4.3.4 多个元素组合

可同时给不相干的多个元素设置同一个样式 

比如 把整 个页面所有的<p> 和<h3>标签同时设置样式 

```html
p,h3 {
  color: red;
  background: yellow;
}
```

效果

<img src="https://gitee.com//riotian/blogimage/raw/master/img/20200914123110.png" alt="image-20200606180807866" style="zoom:50%;" />

### 4.4 盒子模型

所有HTML元素可以看作盒子，在CSS中，"box model"这一术语是用来设计和布局时使用。

CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：边距，边框，填充，和实际内容。

盒模型允许我们在其它元素和周围元素边框之间的空间放置元素。

下面的图片说明了盒子模型(Box Model)：

![CSS box-model](https://gitee.com//riotian/blogimage/raw/master/img/20200914123115.gif)

不同部分的说明：

- **Margin(外边距)** - 清除边框外的区域，外边距是透明的。
- **Border(边框)** - 围绕在内边距和内容外的边框。
- **Padding(内边距)** - 清除内容周围的区域，内边距是透明的。
- **Content(内容)** - 盒子的内容，显示文本和图像。

为了正确设置元素在所有浏览器中的宽度和高度，你需要知道的盒模型是如何工作的。

#### 元素的宽度和高度

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914124731.gif)**重要:** 当指定一个 CSS 元素的宽度和高度属性时，你只是设置**内容区域的宽度和高度**。要知道，完整大小的元素，你还必须添加内边距，边框和边距。

下面的例子中的元素的总宽度为300px：

```html
div {
    width: 300px;
    border: 25px dashed yellow;
    padding: 25px;
    margin: 25px;
}
```

```html
<body>
	<h2>盒子模型演示</h2>
	<p>CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：边距，边框，填充，和实际内容。</p>
	<div>这里是盒子内的实际内容。有 25px 内间距，25px 外间距、5px 黄色边框。</div>
</body>
```

效果

<img src="https://gitee.com//riotian/blogimage/raw/master/img/20200914123119.png" alt="image-20200606182354423" style="zoom:50%;" />

### 4.5 常用CSS属性

css有很多属性，我们先只讲几个基本的

更多的看这里：https://www.runoob.com/cssref/css-reference.html#box 

#### 4.5.1 background背景属性

| 属性                                                         | 描述                                                         | CSS  |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--- |
| [background](https://www.runoob.com/cssref/css3-pr-background.html) | 复合属性。设置对象的背景特性。                               | 1    |
| [background-attachment](https://www.runoob.com/cssref/pr-background-attachment.html) | 设置或检索背景图像是随对象内容滚动还是固定的。必须先指定background-image属性。 | 1    |
| [background-color](https://www.runoob.com/cssref/pr-background-color.html) | 设置或检索对象的背景颜色。                                   | 1    |
| [background-image](https://www.runoob.com/cssref/pr-background-image.html) | 设置或检索对象的背景图像。                                   | 1    |
| [background-position](https://www.runoob.com/cssref/pr-background-position.html) | 设置或检索对象的背景图像位置。必须先指定background-image属性。 | 1    |
| [background-repeat](https://www.runoob.com/cssref/pr-background-repeat.html) | 设置或检索对象的背景图像如何铺排填充。必须先指定background-image属性。 | 1    |
| [background-clip](https://www.runoob.com/cssref/css3-pr-background-clip.html) | 指定对象的背景图像向外裁剪的区域。                           | 3    |
| [background-origin](https://www.runoob.com/cssref/css3-pr-background-origin.html) | S设置或检索对象的背景图像计算background-position时的参考原点(位置)。 | 3    |
| [background-size](https://www.runoob.com/cssref/css3-pr-background-size.html) | 检索或设置对象的背景图像的尺寸大小。                         | 3    |

#### 4.5.2 边框Border 和轮廓Outline属性

| 属性                                                         | 描述                                                       | CSS  |
| :----------------------------------------------------------- | :--------------------------------------------------------- | :--- |
| [border](https://www.runoob.com/cssref/pr-border.html)       | 复合属性。设置对象边框的特性。                             | 1    |
| [border-bottom](https://www.runoob.com/cssref/pr-border-bottom.html) | 复合属性。设置对象底部边框的特性。                         | 1    |
| [border-bottom-color](https://www.runoob.com/cssref/pr-border-bottom-color.html) | 设置或检索对象的底部边框颜色。                             | 1    |
| [border-bottom-style](https://www.runoob.com/cssref/pr-border-bottom-style.html) | 设置或检索对象的底部边框样式。                             | 1    |
| [border-bottom-width](https://www.runoob.com/cssref/pr-border-bottom-width.html) | 设置或检索对象的底部边框宽度。                             | 1    |
| [border-color](https://www.runoob.com/cssref/pr-border-color.html) | 置或检索对象的边框颜色。                                   | 1    |
| [border-left](https://www.runoob.com/cssref/pr-border-left.html) | 复合属性。设置对象左边边框的特性。                         | 1    |
| [border-left-color](https://www.runoob.com/cssref/pr-border-left-color.html) | 设置或检索对象的左边边框颜色。                             | 1    |
| [border-left-style](https://www.runoob.com/cssref/pr-border-left-style.html) | 设置或检索对象的左边边框样式。                             | 1    |
| [border-left-width](https://www.runoob.com/cssref/pr-border-left-width.html) | 设置或检索对象的左边边框宽度。                             | 1    |
| [border-right](https://www.runoob.com/cssref/pr-border-right.html) | 复合属性。设置对象右边边框的特性。                         | 1    |
| [border-right-color](https://www.runoob.com/cssref/pr-border-right-color.html) | 设置或检索对象的右边边框颜色。                             | 1    |
| [border-right-style](https://www.runoob.com/cssref/pr-border-right-style.html) | 设置或检索对象的右边边框样式。                             | 1    |
| [border-right-width](https://www.runoob.com/cssref/pr-border-right-width.html) | 设置或检索对象的右边边框宽度。                             | 1    |
| [border-style](https://www.runoob.com/cssref/pr-border-style.html) | 设置或检索对象的边框样式。                                 | 1    |
| [border-top](https://www.runoob.com/cssref/pr-border-top.html) | 复合属性。设置对象顶部边框的特性。                         | 1    |
| [border-top-color](https://www.runoob.com/cssref/pr-border-top-color.html) | 设置或检索对象的顶部边框颜色                               | 1    |
| [border-top-style](https://www.runoob.com/cssref/pr-border-top-style.html) | 设置或检索对象的顶部边框样式。                             | 1    |
| [border-top-width](https://www.runoob.com/cssref/pr-border-top-width.html) | 设置或检索对象的顶部边框宽度。                             | 1    |
| [border-width](https://www.runoob.com/cssref/pr-border-width.html) | 设置或检索对象的边框宽度。                                 | 1    |
| [outline](https://www.runoob.com/cssref/pr-outline.html)     | 复合属性。设置或检索对象外的线条轮廓。                     | 2    |
| [outline-color](https://www.runoob.com/cssref/pr-outline-color.html) | 设置或检索对象外的线条轮廓的颜色。                         | 2    |
| [outline-style](https://www.runoob.com/cssref/pr-outline-style.html) | 设置或检索对象外的线条轮廓的样式。                         | 2    |
| [outline-width](https://www.runoob.com/cssref/pr-outline-width.html) | 设置或检索对象外的线条轮廓的宽度。                         | 2    |
| [border-bottom-left-radius](https://www.runoob.com/cssref/css3-pr-border-bottom-left-radius.html) | 设置或检索对象的左下角圆角边框。                           | 3    |
| [border-bottom-right-radius](https://www.runoob.com/cssref/css3-pr-border-bottom-right-radius.html) | 设置或检索对象的右下角圆角边框。                           | 3    |
| [border-image](https://www.runoob.com/cssref/css3-pr-border-image.html) | 设置或检索对象的边框样式使用图像来填充。                   | 3    |
| [border-image-outset](https://www.runoob.com/cssref/css3-pr-border-image-outset.html) | 规定边框图像超过边框的量。                                 | 3    |
| [border-image-repeat](https://www.runoob.com/cssref/css3-pr-border-image-repeat.html) | 规定图像边框是否应该被重复（repeated）、拉伸（stretched）  | 3    |
| [border-image-slice](https://www.runoob.com/cssref/css3-pr-border-image-slice.html) | 规定图像边框的向内偏移。                                   | 3    |
| [border-image-source](https://www.runoob.com/cssref/css3-pr-border-image-source.html) | 规定要使用的图像，代替 border-style 属性中设置的边框样式。 | 3    |
| [border-image-width](https://www.runoob.com/cssref/css3-pr-border-image-width.html) | 规定图像边框的宽度。                                       | 3    |
| [border-radius](https://www.runoob.com/cssref/css3-pr-border-radius.html) | 设置或检索对象使用圆角边框。                               | 3    |
| [border-top-left-radius](https://www.runoob.com/cssref/css3-pr-border-top-left-radius.html) | 定义左上角边框的形状。                                     | 3    |
| [border-top-right-radius](https://www.runoob.com/cssref/css3-pr-border-top-right-radius.html) | 定义右上角边框的形状。                                     | 3    |
| box-decoration-break                                         | 规定行内元素被折行                                         | 3    |
| [box-shadow](https://www.runoob.com/cssref/css3-pr-box-shadow.html) | 向方框添加一个或多个阴影。                                 | 3    |

#### 4.5.3 内边距Padding属性

| 属性                                                         | 说明                         | CSS  |
| :----------------------------------------------------------- | :--------------------------- | :--- |
| [padding](https://www.runoob.com/cssref/pr-padding.html)     | 在一个声明中设置所有填充属性 | 1    |
| [padding-bottom](https://www.runoob.com/cssref/pr-padding-bottom.html) | 设置元素的底填充             | 1    |
| [padding-left](https://www.runoob.com/cssref/pr-padding-left.html) | 设置元素的左填充             | 1    |
| [padding-right](https://www.runoob.com/cssref/pr-padding-right.html) | 设置元素的右填充             | 1    |
| [padding-top](https://www.runoob.com/cssref/pr-padding-top.html) | 设置元素的顶部填充           | 1    |

#### 4.5.4 外边距(Margin) 属性

| 属性                                                         | 说明                           | CSS  |
| :----------------------------------------------------------- | :----------------------------- | :--- |
| [margin](https://www.runoob.com/cssref/pr-margin.html)       | 在一个声明中设置所有外边距属性 | 1    |
| [margin-bottom](https://www.runoob.com/cssref/pr-margin-bottom.html) | 设置元素的下外边距             | 1    |
| [margin-left](https://www.runoob.com/cssref/pr-margin-left.html) | 设置元素的左外边距             | 1    |
| [margin-right](https://www.runoob.com/cssref/pr-margin-right.html) | 设置元素的右外边距             | 1    |
| [margin-top](https://www.runoob.com/cssref/pr-margin-top.html) | 设置元素的上外边距             | 1    |

#### 4.5.5 position 定位属性

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123125.png)

属性值

| 值                                                           | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [absolute](https://www.runoob.com/css/css-positioning.html#position-absolute) | 生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。 |
| [fixed](https://www.runoob.com/css/css-positioning.html#position-fixed) | 生成固定定位的元素，相对于浏览器窗口进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。 |
| [relative](https://www.runoob.com/css/css-positioning.html#position-relative) | 生成相对定位的元素，相对于其正常位置进行定位。因此，"left:20" 会向元素的 LEFT 位置添加 20 像素。 |
| [static](https://www.runoob.com/css/css-positioning.html#position-static) | 默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 声明）。 |
| [sticky](https://www.runoob.com/css/css-positioning.html#position-sticky) | 粘性定位，该定位基于用户滚动的位置。它的行为就像 position:relative; 而当页面滚动超出目标区域时，它的表现就像 position:fixed;，它会固定在目标位置。**注意:** Internet Explorer, Edge 15 及更早 IE 版本不支持 sticky 定位。 Safari 需要使用 -webkit- prefix (查看以下实例)。 |
| inherit                                                      | 规定应该从父元素继承 position 属性的值。                     |
|                                                              |                                                              |

#### 4.5.6 字体（Font） 属性

| 属性                                                         | 说明                                                      | CSS  |
| :----------------------------------------------------------- | :-------------------------------------------------------- | :--- |
| [font](https://www.runoob.com/cssref/pr-font-font.html)      | 在一个声明中设置所有字体属性                              | 1    |
| [font-family](https://www.runoob.com/cssref/pr-font-font-family.html) | 规定文本的字体系列                                        | 1    |
| [font-size](https://www.runoob.com/cssref/pr-font-font-size.html) | 规定文本的字体尺寸                                        | 1    |
| [font-style](https://www.runoob.com/cssref/pr-font-font-style.html) | 规定文本的字体样式                                        | 1    |
| [font-variant](https://www.runoob.com/cssref/pr-font-font-variant.html) | 规定文本的字体样式                                        | 1    |
| [font-weight](https://www.runoob.com/cssref/pr-font-weight.html) | 规定字体的粗细                                            | 1    |
| [@font-face](https://www.runoob.com/cssref/css3-pr-font-face-rule.html) | 一个规则，允许网站下载并使用其他超过"Web- safe"字体的字体 | 3    |
| [font-size-adjust](https://www.runoob.com/cssref/css3-pr-font-size-adjust.html) | 为元素规定 aspect 值                                      | 3    |
| [font-stretch](https://www.runoob.com/cssref/css3-pr-font-stretch.html) | 收缩或拉伸当前的字体系列                                  | 3    |

#### 4.5.7 文本（Text） 属性

| 属性                                                         | 说明                                                  | CSS  |
| :----------------------------------------------------------- | :---------------------------------------------------- | :--- |
| [color](https://www.runoob.com/cssref/pr-text-color.html)    | 设置文本的颜色                                        | 1    |
| [direction](https://www.runoob.com/cssref/pr-text-direction.html) | 规定文本的方向 / 书写方向                             | 2    |
| [letter-spacing](https://www.runoob.com/cssref/pr-text-letter-spacing.html) | 设置字符间距                                          | 1    |
| [line-height](https://www.runoob.com/cssref/pr-dim-line-height.html) | 设置行高                                              | 1    |
| [text-align](https://www.runoob.com/cssref/pr-text-text-align.html) | 规定文本的水平对齐方式                                | 1    |
| [text-decoration](https://www.runoob.com/cssref/pr-text-text-decoration.html) | 规定添加到文本的装饰效果                              | 1    |
| [text-indent](https://www.runoob.com/cssref/pr-text-text-indent.html) | 规定文本块首行的缩进                                  | 1    |
| [text-transform](https://www.runoob.com/cssref/pr-text-text-transform.html) | 控制文本的大小写                                      | 1    |
| [vertical-align](https://www.runoob.com/cssref/pr-pos-vertical-align.html) | 设置元素的垂直对齐方式                                | 1    |
| [white-space](https://www.runoob.com/cssref/pr-text-white-space.html) | 设置怎样给一元素控件留白                              | 1    |
| [word-spacing](https://www.runoob.com/cssref/pr-text-word-spacing.html) | 设置单词间距                                          | 1    |
| [text-emphasis](https://www.runoob.com/cssref/css3-pr-text-emphasis.html) | 向元素的文本应用重点标记以及重点标记的前景色。        | 1    |
| [hanging-punctuation](https://www.runoob.com/cssref/css3-pr-hanging-punctuation.html) | 指定一个标点符号是否可能超出行框                      | 3    |
| [punctuation-trim](https://www.runoob.com/cssref/css3-pr-punctuation-trim.html) | 指定一个标点符号是否要去掉                            | 3    |
| [text-align-last](https://www.runoob.com/cssref/css3-pr-text-align-last.html) | 当 text-align 设置为 justify 时，最后一行的对齐方式。 | 3    |
| [text-justify](https://www.runoob.com/cssref/css3-pr-text-justify.html) | 当 text-align 设置为 justify 时指定分散对齐的方式。   | 3    |
| [text-outline](https://www.runoob.com/cssref/css3-pr-text-outline.html) | 设置文字的轮廓。                                      | 3    |
| [text-overflow](https://www.runoob.com/cssref/css3-pr-text-overflow.html) | 指定当文本溢出包含的元素，应该发生什么                | 3    |
| [text-shadow](https://www.runoob.com/cssref/css3-pr-text-shadow.html) | 为文本添加阴影                                        | 3    |
| [text-wrap](https://www.runoob.com/cssref/css3-pr-text-wrap.html) | 指定文本换行规则                                      | 3    |
| [word-break](https://www.runoob.com/cssref/css3-pr-word-break.html) | 指定非CJK文字的断行规则                               | 3    |
| [word-wrap](https://www.runoob.com/cssref/css3-pr-word-wrap.html) | 设置浏览器是否对过长的单词进行换行。                  | 3    |



## 五、练习题

### 5.1 开发登录注册页面

请参考下面的注册页面，模仿开发一个一样的注册页面， 只写好静态页面样式就行，不需要能真的获取验证码等。

![](https://gitee.com//riotian/blogimage/raw/master/img/20200914123135.png)



> Ps：部分内容来自博主线上课程的教程的文件内容