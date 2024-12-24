#brew 

> 参考博客：[https://wsgzao.github.io/post/homebrew-bundle/](https://wsgzao.github.io/post/homebrew-bundle/)

[Homebrew](https://brew.sh)实在是太好用，几乎所有命令行都可以通过它来安装，不知不觉中，我们就安装了一堆命令行，然后安装这些命令行的时候可能需要一些特殊的选项，如果有一天我们需要再一台全新的macbook安装这些命令行，那么这将非常考验你的记忆力，所以我们需要备份。

这里的备份不是简单把这些安装包备份到硬盘，而是备份安装安装了哪些命令行，安装的时候用了什么选项。

  

homebrew为我们提供了非常方便的子命令：**bundle**

  

```bash
brew bundle dump
```

首次执行上面这条命令，将会自动tap **homebrew/bundle，**然后将以往安装的命令以及安装的命令行选项保存在当前的路径下的**Brewfile。**用编辑器打开来看其实就是一个脚本包含了所有已经安装的命令行以及相应的选项。你要备份的文件就是这个Brewfile

  

在全新的macOS上想要恢复的话，就使用该Brewfile.

```bash
brew bundle
```

该命令会在当前文件夹下寻找Brewfile文件然后开始执行。

  

更多关于bundle自命令的帮助请执行，enjoy。

```text
brew help bundle
```

