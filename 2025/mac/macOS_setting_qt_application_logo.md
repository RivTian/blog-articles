# mac下Qt设置应用程序图标

一、设置应用程序图标：

1、桌面新建一个文件夹，命名为 logo.iconset。将png图标（原图只能是.png文件）放进去。

2、打开终端，cd到这个文件夹，依次执行下面的语句：

```sh
sips -z 16 16 logo.png --out icon_16.png
sips -z 16 16 logo.png --out icon_16@2x.png
 
sips -z 32 32 logo.png --out icon_32.png
sips -z 32 32 logo.png --out icon_32@2x.png
 
sips -z 64 64 logo.png --out icon_64.png
sips -z 64 64 logo.png --out icon_64@2x.png
 
sips -z 128 128 logo.png --out icon_128.png
sips -z 128 128 logo.png --out icon_128@2x.png
 
sips -z 256 256 logo.png --out icon_256.png
sips -z 256 256 logo.png --out icon_256@2x.png
 
sips -z 512 512 logo.png --out icon_512.png
sips -z 512 512 logo.png --out icon_512@2x.png
```

我的图片名称为 logo.png。

命名一定要以 `icon_**.png`及 `icon_**@2x.png` 为模版，否则会出现 fail to generate  icns 错误。

3、cd到上一层：cd ../ （logo.iconset文件夹所在目录，我的是在桌面），执行：

```sh
iconutil -c icns logo.iconset
```

执行完会在logo.iconset文件夹所在目录生成logo.icns图标文件。将这个文件复制到项目下，可以和pro文件放在同一目录下。

4、pro文件中添加一行：

```pro
ICON = ./logo.icns
```

5、如果没效果，可以删掉可执行程序和Makefile文件，再执行qmake、构建。

二、设置运行时在程序坞中的图标：

```cpp
setWindowIcon(QIcon(":/res/Resource/logo.ico"));
```

