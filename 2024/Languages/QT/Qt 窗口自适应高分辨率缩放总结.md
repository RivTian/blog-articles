#Qt 

### 前言

windows下设置更改文本、应用等项目的大小为100%和125%时，Qt窗口显示正常，也能正常缩放。但是设置为150%和175%时，Qt窗口会出现大小自适应失效的问题。

即使设置了支持高分辨率：QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);也没有什么效果。

这个问题其实所有的程序都会有，不止Qt开发的程序会有这个问题。

  
### Qt支持高分辨率缩放历程

- Qt4时代的程序遇到高分屏缩放，不作任何处理。
- Qt5.6开始提供了高分屏缩放支持，需要在main函数前面设置 QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
- Qt5.14开始提供了高分屏缩放策略设置，需要在main函数前面设置 QApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);第二个参数用来控制缩放策略。
- Qt6.0开始默认开启高分屏属性Qt::AA_EnableHighDpiScaling，而且不允许关闭（所以你会发现程序用Qt6编译后界面变得很大）。可以通过setHighDpiScaleFactorRoundingPolicy函数设置策略。
- 如果不想要高分屏，希望程序永远保持默认的尺寸，你需要在main函数前面设置 QApplication::setAttribute(Qt::AA_Use96Dpi); 表示永远不缩放。

  
### Qt启动高分屏的方法

 1. 在QApplication a(argc, argv);之前设置Qt::AA_EnableHighDpiScaling和setHighDpiScaleFactorRoundingPolicy。
```cpp
int main(int args, char *argv[])
{
	QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
	QApplication::setHightDpiScaleFactorRoundingPolicy(Qt::HightDpiScaleFactorRoundingPolicy::PassThrough); // Parameters 2️⃣ 用于控制缩放策略
	QApplication a(argc, argv);
}
```

这样用法的缺点是图片容易发虚，比如复选框的边框，哪怕是Qt内置样式风格或者系统默认风格也一样。还有就是不完全适配各种高分设置，效果不好，容易失效。

2. 写个文本文件qt.conf（Qt程序默认的标准配置文件，必须是这个名字），写入配置内容，放到可执行文件同一目录即可，然后在pro文件中将此文件作为资源文件添加进来。此方法采用操作系统的策略进行缩放，推荐此方法，虽然看起来稍微有点模糊，但不会出现发虚等问题，整体一致。而且不容易失控。

**文件内写入：**

```conf
[Platforms]
WindowsArguments = dpiawareness = 0
```

**pro 文件写入：**

```pro
RESOURCE += \
	qt.conf
```
  
即使是这种方法也不是完美的高分屏支持方法，只能是尽量满足适配问题。  
哪怕是windows系统本身，在开启缩放的时候，任务管理器也是模糊的很（尽管改成124%可以改变，但总归不是好办法），还有很多其他知名软件也是如此。

还有需要**注意**的是，在使用配置文件设置时，确保在程序中不要再设置设置Qt::AA_EnableHighDpiScaling，否则会不生效。