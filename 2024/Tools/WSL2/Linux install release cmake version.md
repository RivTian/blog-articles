
#WSL2 

> 适用于 Deepin 系 Linux
> 
> Arch 系 Linux 使用 AUR yay 工具可直接安装

一、卸载系统老版本的 CMake

```sh
sudo apt autoremove cmake
```

二、下载最新版本 CMake

官网：[Download | CMake](https://cmake.org/download/)

![](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240122104107574-1444688774.png)

## 三、安装

```sh
sudo bash ./cmake-3.28.1-linux-x86_64.sh --prefix=/usr/ --skip-license
```
