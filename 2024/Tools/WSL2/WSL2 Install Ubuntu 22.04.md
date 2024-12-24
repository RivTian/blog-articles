
#WSL2 

## Preferences

- [WSL2 + VsCode 配置 C++ 开发环境指南](https://lrl52.top/732/wsl2-vscode-c/)
- [WSL2 配置 C 语言开发环境](https://blog.huohaodong.com/blog/wsl2-development-environment-configuration)
- [WSL2+oh-my-zsh+VS Code 开发环境搭建获得Mac下相同的开发体验](https://juejin.cn/post/7064161133996802061)
- [Linux 安装最新版本的 CMake](https://www.cnblogs.com/RioTian/p/17979486)

![安装结果](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240115160103510-1833792624.png)


## 一、安装WSL2

按下 Win + X，点击 Windows PowerShell （管理员），在 PowerShell 中键入：

```bash
wsl --install --no-distribution
```

敲完以上命令，系统会自动安装不带任何发行版的 Ubuntu。

只想安装 wsl2 的人到这就结束了。有特殊需求也想同时安装 wsl1 的人可以往下接着看。

本人提供两种方式安装 wsl1 ，第一种是在上述命令的后面添加 `--enable-wsl1` 选项，系统会安装 wsl2 的同时也安装 wsl1，第二种是在安装完wsl2后接着键入：

```bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

上述命令需要手动重启才会生效，去掉/norestart意味着会自动重启。

同时安装了wsl1和wsl2的人，需要指定每次安装发行版时默认的wsl版本。本人目前只用wsl2，所以需要键入：

```bash
wsl --set-default-version 2
```

当然，在安装了发行版后也依然可以改变wsl版本，例如将wsl1版本的debian改为wsl2，只需键入:

```bash
wsl --set-version debian 2
```

如果想查看已安装的发行版及其wsl版本，需键入：

```bash
wsl --list --verbose
wsl -l -v
```

调整 wsl 默认启动的子系统

```powershell
wslconfig /setdefault <LinuxName>
```

## 二、安装Ubuntu22.04LTS

打开Microsoft Store，搜索ubuntu，打开Ubuntu 22.04.1 LTS，点击获取会自动下载ubuntu的安装包。想默认安装在c盘的话只需点击打开，根据提示创建此发行版中的第一个用户(具有root权限的非root用户)。如果想自定义安装路径并且有足够的耐心，可以往下接着看。

刚刚如果不小心点了打开也不用着急，键入以下命令即可卸载安装：

```bash
wsl --unregister Ubuntu-22.04
```

Ubuntu 的安装包存放在

```
C:\Program Files\WindowsApps\CanonicalGroupLimited.Ubuntu22.04LTS_2204.1.23.0_x64__79rhkp1fndgsc
```

路径下，由于目录 C:\Program Files\WindowsApps 是受限的（并且目录是被隐藏的，需要打开查看隐藏文件选项），需要参照以下解答：

![解决方法来自网络](https://pic1.zhimg.com/v2-132b1bdc19fc40e4bfda630fe5876c70_r.jpg)

接着在想要的地方创建文件夹 Ubuntu22.04LTS ，接着访问 

```bash
C:\Program Files\WindowsApps\CanonicalGroupLimited.Ubuntu22.04LTS_2204.1.23.0_x64__79rhkp1fndgsc
```

，将该路径下的 `install.tar.gz` 和 `ubuntu2204.exe` 粘贴到刚刚创建的文件夹下。

现在可以干净卸载 Store 下的 Ubuntu 了。

之后打开 ubuntu2204.exe，根据提示创建第一个用户即可。

## 三、设置根用户密码

wsl安装发行版后默认根用户是未设密码的，有时候要切换到根用户会不太方便，最好设置一下：

```bash
sudo passwd root
```

## 四、使用apt清华源

在 cmd 或者 powershell （不限制终端类型，但更推荐使用 powershell 可以使用下面 nano 右键快速粘贴功能）下键入以下命令打开ubuntu终端：

```bash
wsl -d Ubuntu-22.04
```

设置apt国内镜像源可以加速下载，使用nano编辑/etc/apt/sources.list：

```text
sudo nano /etc/apt/sources.list
```

利用方向键和shift选定所有内容，按ctrl+k删除后，在windows中复制以下内容：

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

在nano中按鼠标右键可粘贴剪切板中内容，然后按ctrl+s保存，再按ctrl+x退出即可。

更新镜像源和所有包：

```bash
sudo apt update
sudo apt upgrade
```

## 五、(可选)个性化终端

平时的主力电脑为 Mac 系统，家里的电脑为 Windows，希望在两种系统里能有同样的开发体验，

这里记录一下我的环境搭建过程，包括 **安装 oh-my-zsh**、**设置 powerlevel10k 主题**、**配置 VS Code** 等。

安装 ZSH

```bash
 # 安装
 sudo apt install zsh -y
 
 # 查看 shells
 cat /etc/shells
 
 # 设置默认 shell
 chsh -s /usr/bin/zsh
```

设置默认 shell 时要求输入密码后，输入后。**重开一个窗口**，zsh 会询问用户配置问题，选择2推荐配置，后续如果需要自己修改~/.zshrc。

#### 修改 hosts 访问 github（github 访问不畅，否则跳过此步）

- 创建 /etc/wsl.conf

  ```bash
  sudo touch /etc/wsl.conf
  
  # 输入以下内容，阻止自动生成 hosts 文件
  [network]
  generateHosts = false
  ```

- 访问 [GitHub最新hosts](https://github.com/maxiaof/github-hosts)

- 将 Github hosts 追加到 /etc/hosts

  ```bash
  sudo vim /etc/hosts
  ```

- ping

  ```bash
  ping github.com
  ping raw.githubusercontent.com
  ```

### 安装 oh-my-zsh

```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### 配置 [powerlevel10k](https://github.com/romkatv/powerlevel10k) 主题

```bash
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ~/powerlevel10k
echo 'source ~/powerlevel10k/powerlevel10k.zsh-theme' >>~/.zshrc
```

**配置主题**

新打开窗口且没有配置时会询问用户一些问题做配置，选自己喜欢的配置，后续可以通过 `p10k configure` 命令重新配置。

##### 下载 zsh-syntax-highlighting 和 zsh-autosuggestions

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

编辑 `~/.zshrc`

```bash
plugins=(zsh-syntax-highlighting zsh-autosuggestions)

# 退出后更新 zshrc
source ~/.zshrc
```

此时输入命令即可看到高亮和建议

<u>更多插件请自行探索。</u>

## 六、(可选)使用clash for windows代理

wsl2的本质是虚拟机，网关和dns指向的都是windows，因此如果要实现“翻墙”，设置WSL2的proxy为win的代理ip+端口即可。

> 另外本机的 Clash 需要开启 Lan 口，不然会一致拦截请求。
>
> 端口号提前看下本机的 Clash 使用的是哪个端口，一般默认使用 7890

推荐配置到当前用户下：

```bash
nano ~/.profile
```

在文件末尾粘贴以下内容：

```bash
# set proxy
proxy=http://$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*'):7890
export http_proxy=$proxy
export https_proxy=$proxy
export all_proxy=$proxy
export ALL_PROXY=$proxy
export no_proxy="localhost,127.0.0.1"
```

```bash
# zshrc 中可以添加的内容 (optional)
# WSL通过Win访问网络，所以WSL的网关指向的是Windows，DNS服务器指向的也是Windows，设置WSL的proxy为win的代理ip+端口即可
# WSL中的DNS server在/etc/resolv.conf中查看，该文件是由/etc/wsl.conf自动生成的。
# 如果关闭了wsl.conf中自动生成resolve.conf并自行修改了resolve.conf，DNS nameserver并不是本机win ip
# 需要开启wsl.conf的自动生成，再运行以下命令
# https://zhuanlan.zhihu.com/p/153124468

# 添加到环境变量设置中，例如 ~/.zshrc
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890"
export http_proxy="http://${hostip}:7890"
```

让设置立刻生效：

```bash
source ~/.profile
# and (optional)
source ~/.zshrc
```

测试一下是否能访问google：

```bash
wget www.google.com
```

如果有类型的返回，则表示成功。

```bash
--2022-06-30 16:36:53--  http://www.google.com/
Connecting to 172.20.0.1:7890... connected.
Proxy request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html                        [ <=>                                              ]  14.75K  --.-KB/s    in 0.06s

2022-06-30 16:36:53 (247 KB/s) - ‘index.html’ saved [15099]
```



## 七、配置 GCC 开发环境

### 安装 C/C++工具链

工具链的配置和原生 Linux 没有什么区别，根据自身需求安装好 cmake、make、gdb、gcc 等工具即可。

```bash
sudo apt install cmake make gdb gcc
```

### IDE 配置

#### VisualStudioCode

VSCode 对 WSL 的本地支持通过插件实现，在扩展商店搜索 `Remote Development` 关键字即可看到扩展包合集，按需安装即可（其中 Remote-WSL 必须安装）。

![](https://blog.huohaodong.com/_next/image?url=%2Fstatic%2Fimages%2Fwsl2-development-environment-configuration%2F%E6%8F%92%E4%BB%B6%E5%B8%82%E5%9C%BA.png&w=1920&q=75)

安装完成后，就可以通过左下角的按钮来进入 WSL 环境进行开发了。

#### CLion

CLion 已经对 WSL 提供了原生支持，本质上其实是对原本 SSH 的开发方式进行了二次封装。在配置 CLion 前需要先配置 WSL 中的 SSH 服务，手动配置的话稍微有点麻烦，好在 Jetbrains 公司提供了一个 [**脚本**](https://github.com/JetBrains/clion-wsl/blob/master/ubuntu_setup_env.sh) 进行自动化配置。

> 在最新版本的 Clion 中无需脚本再次配置了，但也推荐在 WSL2 中运行一下。避免出现奇怪的bug

在终端里自建`.sh`格式的脚本文件，复制下面的脚本内容并保存然后执行即可。

```sh
#!/bin/bash
set -e

SSHD_LISTEN_ADDRESS=127.0.0.1

SSHD_PORT=2222
SSHD_FILE=/etc/ssh/sshd_config
SUDOERS_FILE=/etc/sudoers

# 0. update package lists
sudo apt-get update

# 0.1. reinstall sshd (workaround for initial version of WSL)
sudo apt remove -y --purge openssh-server
sudo apt install -y openssh-server

# 0.2. install basic dependencies
sudo apt install -y cmake gcc clang gdb valgrind build-essential

# 1.1. configure sshd
sudo cp $SSHD_FILE ${SSHD_FILE}.`date '+%Y-%m-%d_%H-%M-%S'`.back
sudo sed -i '/^Port/ d' $SSHD_FILE
sudo sed -i '/^ListenAddress/ d' $SSHD_FILE
sudo sed -i '/^UsePrivilegeSeparation/ d' $SSHD_FILE
sudo sed -i '/^PasswordAuthentication/ d' $SSHD_FILE
echo "# configured by CLion"      | sudo tee -a $SSHD_FILE
echo "ListenAddress ${SSHD_LISTEN_ADDRESS}"	| sudo tee -a $SSHD_FILE
echo "Port ${SSHD_PORT}"          | sudo tee -a $SSHD_FILE
echo "UsePrivilegeSeparation no"  | sudo tee -a $SSHD_FILE
echo "PasswordAuthentication yes" | sudo tee -a $SSHD_FILE
# 1.2. apply new settings
sudo service ssh --full-restart

# 2. autostart: run sshd
sed -i '/^sudo service ssh --full-restart/ d' ~/.bashrc
echo "%sudo ALL=(ALL) NOPASSWD: /usr/sbin/service ssh --full-restart" | sudo tee -a $SUDOERS_FILE
cat << 'EOF' >> ~/.bashrc
sshd_status=$(service ssh status)
if [[ $sshd_status = *"is not running"* ]]; then
  sudo service ssh --full-restart
fi
EOF


# summary: SSHD config info
echo
echo "SSH server parameters ($SSHD_FILE):"
echo "ListenAddress ${SSHD_LISTEN_ADDRESS}"
echo "Port ${SSHD_PORT}"
echo "UsePrivilegeSeparation no"
echo "PasswordAuthentication yes"
```

保存后执行该脚本

```shell
sh ./ubuntu_setup_env.sh
```

该脚本实际上是配置了一个端口号为`2222`的本地 SSH 服务。

之后打开一个 CLion 项目并进入设置界面，找到`Toolchains`选项，点击`+`号选择`WSL`即可出现下面的窗口

> 老版本 Clion 需要的操作

![](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240122113304638-1636563079.webp)

点击`Credentials`最右边的设置按钮进入`SSH Configurations`界面，输入 WSL 的用户名和密码即可。

![](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240122113317376-37928213.webp)




## 八、安装 LLVM，Clang，Clangd工具链

安装 clang、lld 和 libc++:

```bash
sudo apt install clang clangd llvm liblldb-dev ninja-build -y
 
# 查看安装版本, 此时默认的安装版本是 14.0.0-1ubuntu1.1, Target: x86_64-pc-linux-gnu
clang --version
clangd --version
```

安装完以后只需要设置以下三个环境变量就是默认使用 clang 了：

> 一般 zsh 安装会自带识别到 clang 版本，也就不一定需要添加环境便利了
>
> 另外 clang-14 是我本机安装的版本，请读者根据自己版本来填写。
>
> 更详细的请参考： https://www.gnu.org/software/make/manual/html_node/Implicit-Variables.html

```bash
export CC=/usr/bin/clang-14
export CPP=/usr/bin/clang-cpp-14
export CXX=/usr/bin/clang++-14
export LD=/usr/bin/ld.lld-14
```



Linux 程序编译：

```bash
clang++ --std=c++17 -o main main.cpp
```

Windows 程序（交叉）编译：

```bash
clang++ --std=c++17 -target x86_64-pc-windows-gnu -mconsole -o main.exe main.cpp
```

Sample Code

```tree
.
├── main.cpp
└── pocketpy.h
```

```cpp
// main.cpp
#include <iostream>
#include "pocketpy.h"

using namespace pkpy;
using namespace std;

int main(int argc, char** argv)
{
  cout << "C++: hello world!" << endl;

  VM* vm = new VM(true);

  // Hello world!
  vm->exec("print('Python: Hello world!')", "main.py", EXEC_MODE);

  // Create a list
  vm->exec("a = [1, 2, 3]", "main.py", EXEC_MODE);

  // Eval the sum of the list
  PyVar result = vm->exec("sum(a)", "<eval>", EVAL_MODE);
  std::cout << py_cast<i64>(vm, result) << endl;   // 6

  pkpy_delete(vm);

  return 0;
}
```

[`pocketpy.h`](https://github.com/blueloveTH/pocketpy/releases/download/v0.9.3/pocketpy.h) 来自项目 [blueloveTH/pocketpy](https://github.com/blueloveTH/pocketpy):

> PocketPy is a lightweight(~6000 LOC) Python interpreter for game engines.
> PocketPy是一个轻量级的Python解释器，为嵌入至游戏引擎而设计。

### Clion

按图中配置即可，主要是手动添加 C/C++ 编译器 位置。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_240105085124_%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_17038172926587.png)

配置完成后即可正常编写 C/C++代码。如检测到缺少诸如 cmake、gcc 等工具，请在 WSL 中自行安装。



## 九、使用 WSL2 和 VS2022 生成和调试 C++

> 参考链接：https://learn.microsoft.com/zh-cn/cpp/build/walkthrough-build-debug-wsl2?view=msvc-170