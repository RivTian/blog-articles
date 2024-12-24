
## 安装 （OSX平台）

```bash
brew install astyle
brew list astyle
```

## CLion 配置

**1、下载插件File Watchers**

**2、配置插件File Watchers：按照图中一摸一样填写即可**

Name：用户自己取个名字

File type：选C/C++

Scope：选择Open Files

Program：找到目录并选择上面 brew list astyle 的路径

Argument（附：博主最满意格式）：-i \$FileName$ --style=allman --indent=spaces=4 --align-pointer=type --attach-closing-while --indent-col1-comments --pad-oper --pad-comma --pad-header --add-brackets --mode=c

Output paths to refresh：\$FileName$

Working Directory：\$FileDir$

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230509165455.png)

