#Go

## 一、语言环境

```sh
brew info go
brew install go 
# or
brew install golang
```

![info go](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/BHHcGY.png)

### 验证结果

```sh
go version
# go version go1.21.6 darwin/arm64 in 2024/2/6
```

## 二、Nvim 环境配置

基于 dotfiles 的 nvim 配置：[Github](https://github.com/RivTian/dotfiles/tree/main/tag-nvim/config/nvim)

```lua
-- lazy.lua add the code
{ import = "lazyvim.plugins.extras.lang.go" },
```

```lua
-- mason add the lsp: gopls and goformat
```

可参考博客：[Here](https://juejin.cn/post/7188853844476952632)

Go Module 加速：[https://goproxy.io](https://goproxy.io)

```sh
# .zshrc
# Set the GOPROXY environment variable
export GOPROXY=https://goproxy.io,direct
# Set environment variable allow bypassing the proxy for specified repos (optional)
export GOPRIVATE=git.mycompany.com,github.com/my/private
```

All done