## Pre-face

自`OS X Lion 10.7`以来，苹果对`OS X`系统自带输入法做出了许多更新与调整，这些改动大多延续至后续版本的`macOS`系统。其中就包括：在使用系统自带英文输入法的情况下，长按一个元音字母，系统会自动弹出元音音标提示框。

苹果「Apple」设计此功能的初衷当然是为了提高输入便利性，优化用户输入体验。但是，这会对开发人员的日常工作（Coding）带来困扰，因为，我们不能**长按连续输入字符**了。



![uCfEBL](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/uCfEBL.jpg)

> 图：`macOS`系统长按元音字母弹出提示框



# `macOS`长按连续输入的简单设置方法



实际上，苹果「Apple」已经在**用户系统配置项「Mac OS X User Defaults System」**里预留出了长按连续输入的设置选项。我们可以方便地**使用`defaluts`命令行工具关闭`ApplePressAndHoldEnabled`功能**，设置完成后，**注销或重启使其生效**即可。

```
$ defaults write -g ApplePressAndHoldEnabled -bool false
# or
$ defaults write NSGlobalDomain ApplePressAndHoldEnabled -boolean false
```

> 代码清单：`defaults`全局`-g`设置长按连续输入

甚至，我们还可以做到仅针对某个应用程序单独设置：

```
$ defaults write 'com.microsoft.VSCode' ApplePressAndHoldEnabled -bool false
```

> 代码清单：`defaults`针对应用程序设置长按连续输入

我们也可以通过命令来查看当前系统`ApplePressAndHoldEnabled`配置的启用情况。

```
$ defaults read | grep ApplePressAndHoldEnabled
```

> 代码清单：检查`ApplePressAndHoldEnabled`启用情况



# 参考资料

- 知乎问答「20589636」：https://www.zhihu.com/question/20589636