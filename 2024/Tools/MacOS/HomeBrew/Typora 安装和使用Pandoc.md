在Typora中，某些文件格式（包括docx，odt，rtf，epub，LaTeX和wiki）的导入功能和导出功能由名为Pandoc的第三方软件提供支持。这些功能需要安装Pandoc（≥v1.16）。 请注意，对于Typora，Pandoc的安装是可选的，如果您不需要Typora中的高级导入/导出支持，则不必在计算机上安装Pandoc。 本文档将展示如何安装Pandoc以及将Typora与Pandoc一起使用以实现完整的导入/导出功能。

## 什么是Pandoc

[Pandoc](http://pandoc.org/) 是通用文档文本转换器。Typora使用它来支持几种文件类型的文件导入/导出功能。

## 安装Pandoc

### 对于Mac用户

简而言之，有两种推荐方法。

##### 从下载的软件包安装程序中安装

从Pandoc的[下载页面](https://github.com/jgm/pandoc/releases/latest)下载软件包安装程序 ，打开它，然后按照说明进行安装。

![](https://www.typora.net/wp-content/uploads/2019/10/4dfd2-Snip20160502_1.png)

##### 从Homebrew安装 （推荐）

对于使用[Homebrew的](http://brew.sh/)开发人员，安装Pandoc可以从终端启动一行：

    brew install pandoc

