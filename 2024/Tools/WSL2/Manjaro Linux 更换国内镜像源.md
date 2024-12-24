
#Manjaro 

建议直接执行下面的命令：

```sh
sudo pacman-mirrors --country China 
sudo pacman -Syu 
```

自己用的 Manjaro Linux 在使用 `pacman -Syu` 更新系统时出现了连接超时的问题，看来又需要换个镜像源了。趁着今天还没想好要分享的内容，那就干脆以此为主题，总结一下如何给 Manjaro Linux 系统更换国内镜像源。

## 手动更换

这里说的「手动」是相对于后面要介绍的命令方式而言，是比较基础的镜像源更换方法。大致分为两步：

1. 找一个可用的 Arch(Manjaro) Linux 镜像源地址。
2. 编辑 `/etc/pacman.d/mirrorlist` 文件，把新地址按格式写入其中。

第一步找镜像地址有两种方法：

1. 通过搜索引擎搜索。
2. 访问 Arch Linux 的官网镜像地址库：[Mirror Overview](https://archlinux.org/mirrors/)。

我推荐选择第二种方式，简单高效。打开页面后可以按国家排序，所有能用的镜像地址一览无余。

![看到了某 jxust.edu.cn, (笑)](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240125172551049-1850026365.png)


根据域名大致能看出所属的组织或公司。选择一个放心的点击后就能看到该镜像源的地址和状态详情。我点开的这个一看就是江西理工大学的：

![Arch Linux - aliyun Mirror Details](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240125172655896-1345905639.png)


页面中 Available URLs 下有两个镜像地址，分别是 http 和 https 协议的。推荐复制 https 的地址，然后用编辑器打开 **/etc/pacman.d/mirrorlist** 文件，按以下格式粘贴并编辑镜像地址：

```ini
Server = https://mirrors.jxust.edu.cn/archlinux/$repo/os/$arch
```

注意只需要把镜像地址放到 `Server =` 和 `$repo/os/$arch` 之间就可以了。保存后用 `pacman -Syu` 命令更新一下本地软件库。

## 使用命令更换

我在去年刚接触并学习如何安装 Arch Linux 时分享过一篇文章：《[VirtualBox 虚拟机体验 Arch Linux 基础安装小记](https://www.zzxworld.com/posts/install_arch_linux_on_virtual_box)》。在这篇文章的安装系统阶段，有一步操作是使用 `reflector` 命令设置镜像源，这里要用到的就是这个命令。

Arch Linux 在安装时提供了这个命令，但在安装好的系统中并没有它，所以需要先安装：

```shell
sudo pacman -S reflector
```

安装好后就可以用这个命令来更换镜像源了。直接通过命令选项指定国家，协议和数量：

```shell
sudo reflector \
    --country China \
    --protocol https \
    --latest 3 \
    --save /etc/pacman.d/mirrorlist
```

上面这个命令会查询国内支持 **https** 协议的镜像源，并且是最近刚从官方同步过的 3 个地址，然后保存到 **/etc/pacman.d/mirrorlist** 镜像配置文件。等命令执行完，镜像源就更换成功了。

## 选择建议

图省事可以选择命令更换方式，我还是比较青睐于手动。因为更换镜像源不算是一个高频操作，可能几个月甚至半年左右才来这么一次。在熟悉了流程后也并没有觉得复杂，感觉没必要为此多安装一个软件。