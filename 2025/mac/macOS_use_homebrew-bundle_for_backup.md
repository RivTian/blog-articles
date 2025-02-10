## 序

每个人都会积累一套自己习惯使用的 App。如果平时习惯使用 Time Machine 备份，那么在重装系统时，直接用它还原倒是一个不错的办法，不必再手动安装一个个 App。但是有些时候，我们可能想要一个更加「干净」的新系统，此时就需要依次手动安装。这显然不是个高效、省心的方法，可能还需要一个个回忆之前用的 App。这时候，我们往往希望有一份属于自己的 App 清单，最好还能在重装时一键安装，省心省力。无论是Windows还是macOS下我们都希望实现一键装机，而 homebrew-bundle 正是这样一款 Mac 下的备份恢复利器。

## 拓展阅读 🔗

- [开发必备：Mackup 将你的开发工具配置同步到云端](https://segmentfault.com/a/1190000002432158)
- [狡兔三窟——云备份软件列表与相应配置，补充 Time Machine](https://sspai.com/post/43479)
- [定期自动云备份 macOS 软件列表，维护一份属于自己的必备 App 清单](https://sspai.com/post/43265) 

## Time Machine

macOS 自带的 Time Machine 无疑是备份与还原的利器。无论是重装系统，还是新机配置，Time Machine 用起来都十分方便、省心。但是它存在以下不足：

1. 如果直接在本机硬盘上备份，Time Machine 动辄百 G 的硬盘占用，令小硬盘电脑用户望而却步。而且一般情况下，电脑内只有一块硬盘，如果系统和备份在同一硬盘上，那么硬盘挂了就两者皆挂。
2. 如果使用网络备份，带宽和网络空间费用可能都是问题。
3. 如果使用 NAS 或者使用树莓派架设 Time Capsule，则需要有一定的计算机相关基础和折腾能力。

即使上述情况对你来说都不是问题，多一种备份方式也是多一份安全和保障。

## homebrew-bundle

> Bundler for non-Ruby dependencies from Homebrew [🔗](https://github.com/Homebrew/homebrew-bundle)

1. Mac 上非常常用的包管理器 Homebrew, 我们经常用它来安装其他的软件包
2. 还有 Homebrew-cask, 可以用来安装图形界面的 App
3. homebrew-bundle 类似 node 中的 package.json 或者 Cocoapods 中的 Podfile
4. 我们将需要的包和 App, 声明在一个 Brewfile 中, 然后执行 brew bundle 即可安装所有包

### homebrew-bundle 如何备份

备份列表包含:

1. brew tap中的软件库
2. brew 安装的命令行工具
3. brew cask 安装的 App
4. Mac App Store 安装的 App

```sh
# 执行brew bundle dump备份命令
brew bundle dump --describe --force --file="~/.brewfile"

# 参数说明
# --describe：为列表中的命令行工具加上说明性文字。
# --force：直接覆盖之前生成的Brewfile文件。如果没有该参数，则询问你是否覆盖。
# --file="~/Desktop/Brewfile"：在指定位置生成文件。如果没有该参数，则在当前目录生成 Brewfile 文件。
```

该命令会在桌面上生成Brewfile文件，双击打开查看，其内容类似于

```sh
$ cat ~/.brewfile

## 该部分是 brew 中的 tap，相当于一个个软件库
tap "homebrew/bundle"
tap "homebrew/cask"

## 该部分是 brew 安装的命令行工具
# Mac App Store command-line interface
brew "mas"
# UNIX shell (command interpreter)
brew "zsh"
# Fish shell like syntax highlighting for zsh
brew "zsh-syntax-highlighting"

## 该部分是 brew cask 安装的 app
cask "mounty"
cask "dteoh/sqa/slowquitapps"

## 该部分是 Mac App Store 安装的 app
mas "ting_en", id: 734383760
mas "Xcode", id: 497799835

```

### homebrew-bundle 如何恢复

通过备份的软件列表文件批量安装软件

```sh
# 确保 homebrew 存在
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 安装 mas
brew install mas
# 批量安装软件
brew bundle --file="~/.brewfile"
```

分享一下自用的一套 .brewfile 文件: [🔗brewfile.zip](https://files.cnblogs.com/files/RioTian/brewfile.zip?t=1739153968&download=true)