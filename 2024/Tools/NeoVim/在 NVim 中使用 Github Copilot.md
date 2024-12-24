#Nvim 

在 NeoVim 中使用 GitHub Copilot 相当简单，没有太复杂的配置：

- 你需要有 Nodejs v12 以上的版本；
- 安装 NeoVim 的 0.6.0 的 pre 版本。如果是 macOS 就更简单：`brew install neovim --HEAD`；
- 使用你的 NeoVim 插件管理器安装 GitHub 官方的插件：`github/copilot.vim`。我用的是`packer.nvim`，就是添加一行`use 'github/copilot.vim'`即可；
- 安装好后，运行 NeoVim，然后执行命令：`:Copilot auth`，命令栏会出现相关说明，大概是获取授权之类，依照提示一步步执行操作就好。

这就配置好了！

使用更是容易，**见证奇迹的方法只有一种：敲入一定代码后，稍微停顿片刻，等到暗色提示代码显示后，按`tab`键就行。**有时候，你需要做的就是敲几个字母，按下`tab`，再敲几个字母，再按`tab`，不断继续……直到……世界末日。

如何破解：

1年授权码：D1DCA65A-C1EE-4A4A-85B1-ED8EB64A7D05.

End Time 2025.1.5

授权使用教程：https://dux6kdaf9m.feishu.cn/docx/QIkcdLncboJioExQKAzc3VsSnMb
