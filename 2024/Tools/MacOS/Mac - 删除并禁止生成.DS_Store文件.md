
Mac OS X 系统中文件夹中都包含 .DS_Store 隐藏文件，其主要保存了针对这个目录的特殊信息和设置配置，例如查看方式、图标大小以及这个目录的一些附属元数据. 类似于于Windows的desktop.ini.

终端进入文件夹会看到这些文件，会导致文件显示很乱，所以会想把 .DS_Store 文件删除，甚至一劳永逸的禁止生成该文件.

[1] - 删除已有的 .DS_Store 文件

```sh
find . -name .DS_Store -type f -delete ; find . -type d | xargs dot_clean
```

[2] - 禁止生成 .DS_Store 文件

运行，并重启 Mac 生效.

```sh
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE
```

[3] - 恢复生成 .DS_Store 文件

```sh
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```