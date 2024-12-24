
client 安装步骤记录：

## 一、补充依赖库

1. 安装依赖

```sh
sudo apt-get install flex
sudo apt-get install bison
sudo apt-get install gperf
sudo apt-get install build-essential
sudo apt-get install libgl1-mesa-dev
sudo apt-get install libglu1-mesa-dev
sudo apt-get install libegl1-mesa-dev
sudo apt-get install freeglut3-dev 
```

> 该步骤在执行安装某个库 `sudo apt-get install libxcb-xxx-dev` 时，报软件包间依赖关系错误

解决方案：在终端输入命令：

```sh
sudo aptitude install libxcb-xxx-dev
```

根据提示依次选择 n->n->y即可解决此问题

具体参考：[Here](https://blog.csdn.net/weixin_42417818/article/details/109696439)

2. 安装freetype开发库和fontconfig开发库

```sh
sudo apt-get install libfreetype6-dev libfontconfig1-dev
```

3. 给freetype头文件做软链接

```sh
sudo ln -s /usr/include/freetype2/freetype /usr/include/freetype
```

## 二、QT 编译构建以及安装

1. 执行configure 

```sh
mkdir build && cd build
../configure 
``` 

2. 执行 make 命令

```sh
sudo make
```

3. 执行 sudo make install， 完成所有过程（仍未到达）