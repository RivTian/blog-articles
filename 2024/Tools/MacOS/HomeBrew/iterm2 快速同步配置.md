
> iterm2 + Oh  My Zsh 的配置教程
> https://zhuanlan.zhihu.com/p/550022490
> https://sspai.com/post/63241

新设备或重装之后，需要快速恢复之前的 iTerm2 设置。

可以配置 iTerm2 将设置保存到文件，将文件保存到远端或复制到新机器后，就可以方便地恢复 iTerm2 的设置。

## 配置的保存

首先需要将现有的配置导出到文件。

操作路径：

- Preferences
- General
- Preferences

![iTerm2 设置](https://user-images.githubusercontent.com/3783096/93177628-b04fd180-f765-11ea-8839-9126261fc8f4.png)
- 勾选 「Load preferences from a custom folder or URL」
- 指定保存配置的位置
- 指定保存位置后，会询问是否将现有配置复制到文件，在弹出的对话框中选择「Copy」

![复制 iTerm2 配置到文件](https://user-images.githubusercontent.com/3783096/93177650-b645b280-f765-11ea-8470-0d95a38f5c1e.png)
- 这样就会得到一个配置文件 `~/.config/item2/com.googlecode.iterm2.plist`。

## 配置的恢复

讲道理，新环境下，配置 iTerm2 从包含上述 plist 配置的目录加载即可。亲测没没成功，不过通过命令行的方式成功了。

- 将上述配置保存到一个位置，比如 `~/.config/iterm2`
- 打开自带的 Terminal 执行如下命令：

```bash
# 配置 iTerm2 从目录加载配置
$ defaults write com.googlecode.iterm2.plist PrefsCustomFolder -string "~/.config/iterm2"

# 读取配置
$ defaults write com.googlecode.iterm2.plist LoadPrefsFromCustomFolder -bool true
```

启动 iTerm2，此时就会应用上期望的配置了。

## 相关资源

- [Sync iTerm2 Profile With Dotfiles Repository](http://stratus3d.com/blog/2015/02/28/sync-iterm2-profile-with-dotfiles-repository/)