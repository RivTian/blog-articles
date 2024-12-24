#Win

几乎每个 Linux 发行版都会自带包管理工具，比如 CentOS 的 yum、Debian 家族的 apt、Arch Linux 的 pacman 等；此外，FreeBSD 系统有 pkg 和 ports；macOS 上有 Homebrew。而 Windows 长久以来都是通过安装包来进行软件分发部署，官方没有提供任何包管理工具，且 Win 10 自带的微软应用商店一直被人诟病。直到 2020 年，微软通过抄袭另一款开源 Windows 包管理工具 AppGet 之后，开发出了一款叫做 WinGet 的包管理工具，但微软的这款工具只能运行在 Win 10 系统上。令人遗憾的是，由于微软的抄袭行为，[AppGet](https://appget.net/)[](https://web.archive.org/web/2/https://appget.net/ "互联网存档") 的作者已经停止维护这个项目。

除了官方的 [WinGet](https://docs.microsoft.com/zh-cn/windows/package-manager/)[](https://web.archive.org/web/2/https://docs.microsoft.com/zh-cn/windows/package-manager/ "互联网存档")，Windows 上比较流行的第三方的包管理工具有：

- [Chocolatey](https://chocolatey.org/)[](https://web.archive.org/web/2/https://chocolatey.org/ "互联网存档")由商业公司运作的[开源项目](https://github.com/chocolatey/choco/)[](https://web.archive.org/web/2/https://github.com/chocolatey/choco/ "互联网存档")，且提供付费的专业版和商业版。包含命令行工具和图形用户界面的 Chocolatey GUI。
- [Scoop](https://scoop.sh/)[](https://web.archive.org/web/2/https://scoop.sh/ "互联网存档") 个人维护的[开源项目](https://github.com/ScoopInstaller/Scoop)[](https://web.archive.org/web/2/https://github.com/ScoopInstaller/Scoop "互联网存档")。只提供命令行工具，有第三方维护的安装源。除了常用软件外，还包含大量开发工具，比较适合程序员。

## Chocolatey

Chocolatey 的社区仓库里有 9000 多个软件，几乎涵盖了所有的主流软件。Chocolatey 还提供一个 GUI 程序，方便普通用户使用。

由于 Chocolatey 采用常规全局模式安装软件，所以需要管理员权限。如果采用非管理员模式，Chocolatey 只能安装 Portable 版本的软件，可用软件数量就少了很多。

### 安装

可以参考官方的安装方法：[https://chocolatey.org/install#individual](https://chocolatey.org/install#individual)[](https://web.archive.org/web/2/https://chocolatey.org/install#individual "互联网存档")。如果操作系统是 Windows 7，则需要先安装 [PowerShell](https://docs.microsoft.com/zh-cn/powershell/scripting/install/installing-powershell-on-windows)[](https://web.archive.org/web/2/https://docs.microsoft.com/zh-cn/powershell/scripting/install/installing-powershell-on-windows "互联网存档")。

打开 PowerShell 的命令行界面，输入如下命令：

```shell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

等待在线安装完毕，运行 `choco -?` 命令来确定 Chocolatey 是否安装成功。

### 常用命令

建议日常操作使用 Chocolatey GUI。可以手动安装 GUI 程序：

```powershell
choco install chocolateygui
```

等待在线安装完毕，运行 `choco -?` 命令来确定 Chocolatey 是否安装成功。

### 常用命令

建议日常操作使用 Chocolatey GUI。可以手动安装 GUI 程序：

```powershell
choco install chocolateygui
```

#### 安装软件

使用 `choco install <软件名>` 命令来安装软件。下面的命令用来安装压缩软件 7zip：

```powershell
choco install 7zip -y
```

参数 `-y` 表示安装过程不需要确认。

同一个软件包，有时候会有 Install 和 Portable 两种版本，在软件包的名字后通过 `.install` 和 `.portable` 后缀来区分。以 7zip 为例，仓库中有三个版本：

1. 7zip
2. 7zip.portable
3. 7zip.install

如果不确定要安装哪个版本，安装时可以不需要指定后缀，Chocolatey 会自动安装默认依赖的版本。比如 7zip 软件包默认依赖 7zip.install。所以安装完 7zip 后，在已安装列表中会看到 7zip 和 7zip.install 两个软件包。

#### 卸载软件

```powershell
choco uninstall 7zip
```

#### 更新软件

更新指定的软件：

```powershell
choco upgrade 7zip
```

或者使用如下命令更新所有已安装软件：

```powershell
choco upgrade all
```

#### 列出已安装的软件

```powershell
choco list -l
```

#### 查找软件

```powershell
choco search 7zip
```

#### 查看需要更新的包

```powershell
choco outdated
```

### 设置代理

很多软件的安装包都需要从 GitHub 上下载，出于你懂的原因，需要设置代理。

Chocolatey 默认使用系统代理。另外可以通过命令行参数、环境变量和 `choco config` 命令来设置代理。

使用 `--proxy` 命令行参数：

```powershell
choco upgrade all --proxy=http://127.0.0.1:8080
```

设置环境变量：

```shell
set http_proxy=http://127.0.0.1:8080
set https_proxy=http://127.0.0.1:8080
```

显式配置 Chocolatey 的代理：

```powershell
choco config set proxy http://127.0.0.1:8080
```

Chocolatey 不支持 SOCKS 代理。如果你使用 SOCKS 代理，可以通过 [Privoxy](https://www.privoxy.org/)[](https://web.archive.org/web/2/https://www.privoxy.org/ "互联网存档") 转换成 HTTP 代理。

## Scoop

Scoop 可以添加第三方的 bucket，也就是安装源。默认的 main bucket 里有 900 多个软件，extras bucket 里有 1300 多个软件。值得一提的是，还有个 versions bucket，里面包含了一些软件的历史版本。虽然 Scoop 上的软件数量不多，但是涵盖了大多数开发工具。对于软件开发人员来说，使用 Scoop 来搭建开发环境相当方便。

与 Chocolatey 采用系统全局安装方式不同的是，Scoop 安装的软件都保存在 `%userprofile%\scoop\apps` 路径下，不会污染全局环境，不需要管理员权限就可以安装软件。

### 安装

安装 Scoop 前，需要先安装 [PowerShell 5.1](https://aka.ms/wmf5download)[](https://web.archive.org/web/2/https://aka.ms/wmf5download "互联网存档") 及以上版本，和 [.NET Framework 4.5](https://www.microsoft.com/net/download)[](https://web.archive.org/web/2/https://www.microsoft.com/net/download "互联网存档") 及以上版本。

打开 PowerShell 的命令行界面，输入如下命令：

```powershell
Set-ExecutionPolicy RemoteSigned -Scope Process -Force; iwr -useb get.scoop.sh | iex
```

如果安装过程提示网络错误，可以设置系统代理：

```powershell
irm get.scoop.sh -Proxy 'http://<ip:port>' | iex
```

Scoop 默认的应用安装目录为 “%USERPROFILE%\scoop” ，可以在安装 Scoop 时指定应用安装目录：

```shell
irm get.scoop.sh -outfile 'install.ps1'  
.\install.ps1 -ScoopDir 'D:\Soft\Language\Scoop' -ScoopGlobalDir 'D:\Soft\Language\GlobalScoopApps'
```

具体可见官方文档：[https://github.com/ScoopInstaller/Install](https://github.com/ScoopInstaller/Install)[](https://web.archive.org/web/2/https://github.com/ScoopInstaller/Install "互联网存档")

### 常用命令

安装完毕后先运行环境检查命令：

```powershell
scoop checkup
```

根据提示信息，完成系统配置即可。例如下面的错误消息意味着需要安装 `dark` 或 `wixtools` 才能解压安装文件：

> WARN This version of Windows does not support configuration of LongPaths.  
> ERROR ‘dark’ is not installed! It’s required for unpacking installers created with the WiX Toolset. Please run ‘scoop install dark’ or ‘scoop install wixtoolset’.  
> WARN Found 2 potential problems.

如果在 PowerShell 中运行 scoop 报如下错误：

> scoop : 无法加载文件 C:\Users\XXX\scoop\shims\scoop.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 https:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。

则需要以管理员身份运行 PowerShell 并执行下面的命令：

```powershell
Set-ExecutionPolicy RemoteSigned
```

#### 安装软件

使用 `scoop checkup` 命令检查后，一般会要求安装三个包，分别是：7zip、dark 和 innounp。使用如下命令安装：

```powershell
scoop install 7zip dark innounp
```

#### 卸载软件

```powershell
scoop uninstall 7zip
```

#### 更新软件

更新指定软件：

```powershell
scoop update 7zip
```

或者使用如下命令更新所有已安装软件：

```powershell
scoop update -a
```

更新 scoop：

```powershell
scoop update
```

禁止更新指定软件：

```powershell
scoop hold 7zip
```

允许更新指定软件：

```powershell
scoop unhold 7zip
```

#### 列出已安装软件

```powershell
scoop list
```

#### 查找软件

```powershell
scoop search 7zip
```

#### 查看需要更新的包

```powershell
scoop status
```

#### 软件仓库

> [给 Scoop 加上这些软件仓库，让它变成强大的 Windows 软件管理器](https://sspai.com/post/52710)

在 Scoop 中，bucket 代表软件仓库。使用如下命令列出官方支持的 bucket：

```powershell
scoop bucket known
```

> main extras versions nirsoft php nerd-fonts nonportable java games

其中 `extras` 是对 `main` 仓库的补充，`versions` 提供各软件的历史版本。上述两个 bucket 比较常用，使用如下命令添加：

```shell
scoop bucket add extras
scoop bucket add versions
```

还可以添加第三方的 bucket：

```powershell
scoop bucket add dorado https://github.com/chawyehsu/dorado
```

访问下面的网址查看 Scoop 的第三方 bucket：

[https://rasa.github.io/scoop-directory/by-apps](https://rasa.github.io/scoop-directory/by-apps)[](https://web.archive.org/web/2/https://rasa.github.io/scoop-directory/by-apps "互联网存档")

#### 软件版本切换

Scoop 可以安装同一个软件的多个版本，并支持在不同的版本之间切换。比如同时安装了 Python 3.X 和 Python 2.7，那么可以通过 `scoop reset` 命令在两个版本的 Python 间切换。安装命令如下：

```powershell
scoop install python27 python
```

将系统默认的 Python 切换到 Python 2.7：

```powershell
scoop reset python27
```

将系统默认的 Python 切换到 Python 3.X：

```powershell
scoop reset python
```

### 设置代理

基于相同的原因，scoop 也需要设置代理才能正常使用：

```powershell
scoop config proxy 127.0.0.1:8080
```

需要注意的是， Scoop 仅支持 HTTP 代理。此外，由于 Scoop 通过 Git 命令访问 GitHub 来更新软件仓库，还需要为 Git 设置全局代理：

```powershell
git config --global http.https://github.com.proxy http://127.0.0.1:8080
```

### 下载加速

Scoop 默认配置启用 aria2 作为下载客户端，支持多线程下载和断点续传。只要安装 aria2 即可解锁该特性：

```powershell
scoop install aria2
```

还可以通过配置开关禁用此功能：

```powershell
scoop config aria2-enabled false
```

WinGet 部分待续：[Here](https://www.fournoas.com/posts/windows-package-managers/#winget)


## 总结

如果仅需要安装和管理普通应用软件，推荐使用 Chocolatey；如果需要安装各种开发工具配置开发环境，推荐使用 Scoop；Windows 10 / 11 用户可以使用 WinGet 代替 Chocolatey。

此外，如果要安装的应用软件需要文件关联或者和 Windows Shell 进行集成的，则应该用 Chocolatey 或 Winget 安装。因为 Scoop 安装的版本大多是 Portable 版本的，不能实现文件关联和系统集成。