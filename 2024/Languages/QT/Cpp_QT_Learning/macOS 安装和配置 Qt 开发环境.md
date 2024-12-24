t> *介绍如何在 macOS 下使用 Qt Creator + Qt 进行开发。*

## 环境

* macOS 版本：macOS 13.3
* Qt 版本： Qt5
* Shell 版本：Zsh



## 安装相应工具

```bash
# 安装 Qt Creator
brew install --cask qt-creator
# /Applications/Qt Creator.app
# 使用 brew 安装的 qt 会缺少帮助文档，个人建议 dmg 装个 6.2.x 或者网络单独寻找 qch 文件

# 这里安装 Qt5，如果想要安装最新版本的 Qt 使用 qt 替换 qt@5
brew install qt@5
```

![271693621456_.pic](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/271693621456_.pic.jpg)
![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/p5uxDo.png)
## 配置 Zsh

在 `~/.zshrc` 中添加如下配置：

```bash
export PATH="/usr/local/opt/qt@5/bin:$PATH"
export LDFLAGS="-L/usr/local/opt/qt@5/lib"
export CPPFLAGS="-I/usr/local/opt/qt@5/include"
export PKG_CONFIG_PATH="/usr/local/opt/qt@5/lib/pkgconfig"
```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230422001944734.png)



> *如果使用的是 Bash，那么把上述配置写入到* `~/.bashrc` *文件中。*

> 以下暂未操作，还没找到 Clion qmake 位置

## 配置 Qt Version

在 `Qt Creator -> Preferences -> Kits -> Qt Versions -> Add` 添加 `qmake` 的位置，可以通过搜索 `qmake` 的位置。

*PS：这个吐槽下 Qt Creator 的设计， 不能让用户自己直接通过 Add 输入路径，非要通过搜索才可以。*

![macos-config-qt-00.png](https://cdn.jsdelivr.net/gh/shangzongyu/blog-image@main/hashnode/2022/05/macos-config-qt-00.png)

配置完 Qt Versions 后，在 `Qt Creator -> Preferences -> Kits -> Kits` 配置 Qt 的版本，如下图：

![macos-config-qt-01.png](https://cdn.jsdelivr.net/gh/shangzongyu/blog-image@main/hashnode/2022/05/macos-config-qt-01.png)

## 总结

macOS 需要安装 [Qt](https://www.qt.io/) 和 [Qt Creator](https://www.qt.io/product/development-tools):

- Qt - 提供相应的 GUI 框架。
- Qt Creator - 一个跨平台的 IDE 而已，理论上可以使用其他任意 IDE，但是为了方便还是使用这个。

---
## macOS 的 CLion 搭建 QT 环境

好了，现在qt已经安装完成，我们打开Clion进行验证，这里我们新建一个Clion的工程文件：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/1941682094467_.pic.jpg)

创建完Qt文件后的界面：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/1682094740566.jpg)

CmakeLists_.txt文件的配置：

```txt
cmake_minimum_required(VERSION 3.25)  
project(QT_Start)  
  
set(CMAKE_CXX_STANDARD 17)  
set(CMAKE_AUTOMOC ON)  
set(CMAKE_AUTORCC ON)  
set(CMAKE_AUTOUIC ON)  
  
  
find_package(Qt5 COMPONENTS  
Core  
Gui  
Widgets  
REQUIRED)  
  
add_executable(QT_Start main.cpp)  
target_link_libraries(QT_Start  
Qt5::Core  
Qt5::Gui  
Qt5::Widgets  
)
```

我们点击上面红圈中的绿色三角形运行Qt文件中默认给出的代码，如图所示：

![](https://img-blog.csdnimg.cn/a6b204c6d7f847bca3062ef0dae623b2.png)

关于 \*.ui 文件的打开

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230508234058.png)

**参考链接：**

* https://tomshine.hashnode.dev/macos-config-qt
* https://www.cnblogs.com/BinarySong/p/16267578.html
* win 端CLion配置QT教程 https://www.robotsfan.com/posts/113f8d22.html
* https://www.ithere.net/article/1544155826143694849