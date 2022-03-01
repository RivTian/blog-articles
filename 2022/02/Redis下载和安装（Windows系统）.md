#  在 Windows 上安装 Redis

Redis 官方不建议在 windows 下使用 Redis，所以官网没有 windows 版本可以下载。但可以通过 GitHub 来下载 Windows 版 Redis 安装包，下载地址：[点击前往](https://github.com/tporadowski/redis/releases)。

> 注意：Windows 安装包是某位民间“大神”根据 Redis 源码改造的，并非 Redis 官方网站提供。

在 Windows 系统下安装 Redis 要比 Linux 系统安装稍微复杂一些，故本篇详细介绍如何在 Windows 系统上如何安装 Redis。

打开上述的下载链接，Redis 支持 32 位和 64 位的 Window 系统，大家根据个人情况自行下载，如下图所示：

<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223214324.png" alt="Github 下载页面"></center>

下载并解压完成后，打开相应的文件夹（如：`D:\Soft\Redis`），您会看到如下图所示的文件目录：

<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223214607.png" alt="Redis 安装目录解析"></center>


## 创建Redis临时服务

#### 1) 启动服务端程序

在文件管理器的导航栏打开 `CMD` 进行命令行模式后键入 

```sh
redis-server.exe redis.windows.conf
```

> 或者双击 Redis 服务端启动程序 redis-server.exe

<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223214926.png"></center>


上图中显示一些 Redis 的相关信息，比如 Redis 的版本号以及默认端口号(6379)。注意，为了实现后续操作，请您保持服务端开启状态，否则客户端无法正常工作 （即另开一个 CMD 进程并切换至 Redis 文件夹）。 

#### 2) 启动客户端程序

启动服务端后，在另外的 CMD 进程中键入 `redis-cli.exe -h 127.0.0.1 -p 6379` 后得到如下界面

> 或者双击客户端启动程序 redis-cli.exe，

<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223215119.png"></center>

得到如上界面，说明 Redis 本地客户端与服务端连接成功。

## 命令创建Redis服务

上述方式虽然简单快捷，但是显然不是程序员的操作，下面介绍，通过命令启动 Redis 服务端，并将 Redis 服务添加到 Windows 资源管理器，实现开机后自动启动。

#### 1) 注册Redis服务

通过 CMD 命令行工具进入 Redis 安装目录，将 Redis 服务注册到 Windows 服务中，执行以下命令：

```sh
redis-server.exe --service-install redis.windows.conf --loglevel verbose
```

执行完后，得到以下输出，说明注册成功。

```sh
[12392] 23 Feb 21:56:35.197 # Granting read/write access to 'NT AUTHORITY\NetworkService' on: "D:\Soft\Redis" "D:\Soft\Redis\"
[12392] 23 Feb 21:56:35.197 # Redis successfully installed as a service.
```

#### 2) 启动Redis服务

执行以下命令启动 Redis 服务，命令如下：

```sh
redis-server --service-start
```

<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223215740.png"></center>

注意：此时 Redis 已经被添加到 Windows 服务中，因此不会再显示 Redis 服务端的相应的信息，如下图所示：



<center><img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220223215842.png"></center>



#### 3) 启动Redis客户端

在 CMD 命令行键入 `redis-cli` 命令启动客户端

#### 4) 检查是否连接成功

测试客户端和服务端是否成功连接。输出 `PING` 命令，若返回 `PONG` 则证明成功连接。



通过上面的操作，我们完成了 Redis 的安装。当然，您也可以将 Redis 加入到环境变量中 （添加完环境变量后就可以不必每次都切换到 Redis 的目录下了）

> <font color="#cc0000"><b>注意：根据自己的安装路径添加环境变量。</b></font>

## 总结

下面对安装过程中涉及到的命令进行总结，主要包括以下命令：

```sh
安装服务：redis-server --service-install
卸载服务：redis-server --service-uninstall
开启服务：redis-server --service-start
停止服务：redis-server --service-stop
服务端启动时重命名：redis-server --service-start --service-name Redis1
```

## 参考

* 菜鸟教程：[https://www.runoob.com/redis/redis-install.html](https://www.runoob.com/redis/redis-install.html)
* Redis 配置文件详解：[https://juejin.cn/post/6844904032268451853](https://juejin.cn/post/6844904032268451853)
