#Manjaro 

参考 Wiki：[https://wiki.archlinuxcn.org/zh-hans/Pacman](https://wiki.archlinuxcn.org/zh-hans/Pacman)

yay 命令参考：[Here](obsidian://open?vault=Obsidian_Notes&file=Tools%2FWSL2%2FArch(Manjaro)%20Linux%20yay%20%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3)

Pacman 是一个软件包管理器，作为ArchLinux发行版的一部分。简单来说，就是和apt-get之于Ubuntu一样，pacman就是Arch的apt-get。要想轻松玩转Arch，学会pacman是必需的。

Pacman包管理器是ArchLinux的一大亮点。它汲取了其他Linux版本软件管理的优点，譬如Debian的APT机制、Redhat的 Yum机制、 Suse的Yast等，对于安装软件提供了无与伦比的方便。另外由于ArchLinux是一个针对i686架构优化的发行版，因此对于软件的效率提高也有一定的帮助。pacman可以说是ArchLinux的基础，因为ArchLinux默认安装非常少的软件，其他软件都是使用pacman通过网络来安装的。它将一个简单的二进制包格式和易用的构建系统结合了起来。Pacman使得简单的管理与自定义软件包成为了可能，而不论他们来自于官方的Arch软件库或是用户自己创建的。Pacman可以通过和主服务器同步包列表来进行系统更新，这使得注重安全的系统管理员的维护工作成为轻而易举的事情。

要完全了解pacman可以做什么，请阅读man pacman。以下只是一些pacman的简单操作实例。

## 一、更新系统

在 Arch Linux 中，使用一条命令即可对整个系统进行更新：

`pacman -Syu`

如果你已经使用 `pacman -Sy` 将本地的包数据库与远程的仓库进行了同步，也可以只执行：

`pacman -Su`

开始强制滚动更新

`pacman -Syyu`

## 二、安装包

```sh
pacman -S <pkg_name> # 执行包的安装, 可通过空格间隔多个包名
pacman -Sy <pkg_name> # 该命令是先同步包数据后再执行安装
pacman -Sv <pkg_name> # 在显示一些操作信息后执行安装
pacman -Su            # 更新所有软件
pacman -Syu           # 更新软件源并更新软件
pacman -Syyu          # 强行更新一遍再更新软件
pacman -U <local_pkg> # 安装本地包, 其拓展名为 pkg, tar.gz etc.
```

## 三、删除包

```sh
pacman -R <pkg_name> # 该命令只删除包, 不删除其依赖
pacman -Rs <pkg_name> # 删除包的同时, 删除其依赖
pacman -Rd <pkg_name> # 删除包的同时, 不检查其依赖
pacman -Rns <pkg_name> # 删除包的同时，删除其依赖, 并删除 <pkg> 全局配置文件, Recommands
```

## 四、搜索包

```sh
pacman -Q  # 列出所有软件包
pacman -Qe # 查询安装的软件
pacman -Qea # 查询安装的软件, 但不显示 version
pacman -Qs <pkg_name> # 查询本地安装的所有名称带 <pkg_name> 的软件
pacman -Qdt # 查询所有孤儿软件, 不再需要的
pacman -Qdtq # 查询所有不再被依赖的包名
pacman -Ss <keyword> # 搜索含关键字的包, 并且查询名支持 Regex
pacman -Qi <pkg_name> # 查看有关包的信息
pacman -Ql <pkg_name> # 列出该包的文件
```

## 五、其他用法

```sh
pacman -Sw <pkg_name> # 只下载包,不安装
pacman -Sc  # Pacman 下载的包文件位于 /var/cache/pacman/pkg/ 目录, 该命令将清理未安装的包文件
pacman -Scc # 清理所有的缓存文件
```


## 附注

Arch Linux 的版本库里面包括：

1. Core - 核心软件包
2. extra - 其他常用软件
3. community - 社区软件包，比如 MySQL etc. 
4. testing - 正在测试阶段，还没有正式加入源的软件包。通常软件版本会比较新，不一定 stable
5. release - 已经发布的软件包
6. unstable - 非正式的软件包，可能包括以前版本的软件或者测试软件

因为 Pacman 的软件都是从源里面更新，因此在 `/etc/pacman.d` 里面配置这些软件源的地址。

在 `/etc/pacman.d` 目录里面分别有上面几种软件类型对应的文件名，可以自己手工配置这些软件源的地址。

```sh
$ uname -r # 查询 Linux API 头文件, Manjaro Linux内核头文件, 前两位是关键: 5.15
5.15.133.1-microsoft-standard-WSL2
```
