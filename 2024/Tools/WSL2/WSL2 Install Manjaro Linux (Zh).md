
#WSL2 #Manjaro 

> 先贴个结果图

![](https://img2024.cnblogs.com/blog/1849509/202401/1849509-20240124162848425-33885856.png)


## Manjaro 执行文件下载

- [Download | Github](https://github.com/sileshn/ManjaroWSL2/releases)

  根据个人系统架构下载 x64_86 或者 arm64 的对应 zip 包

解压文件到自定义位置，并双击运行 exe 文件


```sh
wsl --unregister Manjaro
```

## 配置 Manjaro Linux 用户组

安装完成后，我们需要配置 Manjaro Linux 系统才能开始使用。

在命令行执行 `passwd` 以设置 `root` 的密码。根据[最小权限原则(opens in a new tab)](https://zh.wikipedia.org/wiki/最小权限原则)，日常使用中，不能所有命令中都使用 root 权限进行操作，因此我们需要创建一个新的用户以供日常使用。

参照 ArchWSL 文档，在设置完成后，执行以下命令创建新用户并将其设为默认用户：

**（将 `{username}` 替换成你要使用的用户名，注意请不要使用中文）**

Tips：**以下命令均可点击右侧的复制按钮一键复制哦！**

首先设置 `sudoers` 文件：

```
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
```



然后添加用户

```
useradd -m -G wheel -s /bin/bash {username}
```



接着，设置默认用户密码

```
passwd {username}
```



设置完成后执行 `exit` 退出 Arch，在 Windows 的命令行内执行以下命令来设置默认用户：

```
.\Manjaro.exe  config --default-user {username}
```



## 配置镜像源

```sh
sudo pacman-mirrors --country China # 由于网络因素可能部分镜像网站显示 error, 但运行结束后会自动更新可用的镜像列表
sudo pacman -Syu # 选择2 dbus-daemon-units
sudo pacman -S yay # 安装 AUR Helper, 它可以执行pacman的几乎所有操作，并在此基础上添加了很多额外用法
yay -S neovim
yay -S hyfetch # Usage sample (install)
# Dark transgender <enter> <enter> 
```



## 配置代理

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



```sh
# 添加到环境变量设置中，例如 ~/.zshrc
export hostip=$(cat /etc/resolv.conf |grep -oP '(?<=nameserver\ ).*')
export https_proxy="http://${hostip}:7890"
export http_proxy="http://${hostip}:7890"

# source ~/.zshrc
```

若不使用 zsh 的情况，可新建一个 `~/.proxyrc` 

```sh
#!/bin/bash
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
export ALL_PROXY="http://$host_ip:7890"

# source ~/.proxyrc
```



## 配置 Oh My Zsh 环境

```sh
$ yay -S zsh
$ chsh -s /bin/zsh

# 安装 Oh My Zsh
$ sh -c "$(wget https://gitee.com/Devkings/oh_my_zsh_install/raw/master/install.sh -O -)"

# 更新 oh my zsh 主题
$ git clone https://github.com/spaceship-prompt/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1

$ ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"

# clone zsh-syntax-highlighting 和 zsh-autosuggestions 到 Oh My Zsh 插件仓库位置
$ git clone https://gitee.com/yuxiaoxi/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

$ git clone https://gitee.com/githubClone/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# $ git clone https://gitee.com/lxgyChen/autojump ~/temp/autojump/
$ git clone https://gitee.com/lxgyChen/autojump ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/autojump
# $ cd temp/autojump
$ cd ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/autojump
$ ./install.py
# ~/.zshrc 中添加
        [[ -s /home/riotian/.autojump/etc/profile.d/autojump.sh ]] && source /home/riotian/.autojump/etc/profile.d/autojump.sh

        autoload -U compinit && compinit -u
# 更新插件列表
plugins = (git sudo copypath copyfile autojump zsh-syntax-highlighting zsh-autosuggestions)
# source ~/.zshrc

# Set ZSH_THEME="spaceship" in your .zshrc.
# (Optional)
# ~/.zshrc
SPACESHIP_TIME_SHOW="true"
SPACESHIP_USER_SHOW="always"
SPACESHIP_USER_COLOR="212" #粉色
SPACESHIP_RUST_VERBOSE_VERSION=true

alias c="clear"  
alias nv="nvim"

$ source ~/.zshrc
```



## 配置开发环境

```sh
# 配置 Clang 环境
$ yay -S cmake make gdb
$ yay -S clang llvm ninja

# 配置 Rust 环境 https://rsproxy.cn/
# ~/.zshrc
export RUSTUP_DIST_SERVER="https://rsproxy.cn"
export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"
# source ~/.zshrc

$ curl --proto '=https' --tlsv1.2 -sSf https://rsproxy.cn/rustup-init.sh | sh

# 设置 crates.io 镜像， 修改配置 ~/.cargo/config，已支持git协议和sparse协议， >=1.68 版本建议使用 sparse-index，速度更快
[source.crates-io]
replace-with = 'rsproxy-sparse'
[source.rsproxy]
registry = "https://rsproxy.cn/crates.io-index"
[source.rsproxy-sparse]
registry = "sparse+https://rsproxy.cn/index/"
[registries.rsproxy]
index = "https://rsproxy.cn/crates.io-index"
[net]
git-fetch-with-cli = true

# 配置 neovim 优化 vim 体验
# 终端 Nerd Font 请自行配置
$ yay -S fd ripgrep lazygit# yay 会安装最新版本的 neovim(0.9.5 update 24/01/25)
	# 配置 Lazy.vim: https://www.lazyvim.org/
# $ yay -S git

# Make a backup of your current Neovim files:
# required, new machine maybe don't
$ mv ~/.config/nvim{,.bak}
rm -rf ~/.config/nvim

# optional but recommended
$ mv ~/.local/share/nvim{,.bak}
$ mv ~/.local/state/nvim{,.bak}
$ mv ~/.cache/nvim{,.bak}

rm -rf  ~/.local/share/nvim
rm ~/.local/state/nvim
rm ~/.cache/nvim

# Clone the starter
$ git clone https://github.com/LazyVim/starter ~/.config/nvim
#  git clone https://github.com/Innei/nvim-config-lua.git ~/.config/nvim
# Remove the .git folder, so you can add it to your own repo later
$ rm -rf ~/.config/nvim/.git

# Start Neovim!
$ nvim # nv
```


## WSL Tips (备份与恢复)

1. 备份 Manjaro

```powershell
# wsl --export <自定义名称> <备份文件路径>
wsl --export Manjaro D:\Soft\WSL2\BackUp\Manjaro.tar # will be take a minute that running backup
```

2. 恢复备份

```powershell
# wsl --import <自定义名称> <安装系统的位置> <备份文件路径>
wsl --import Manjaro D:\Soft\WSL2\Manjaro D:\Soft\WSL2\BackUp\Manjaro.tar
```

3. 其他操作

```shell
wsl --shutdown <name>

# in Linux
cd /etc/
explorer.exe . 
# 注意后面的点, 可以使用 Windows 的文件管理器打开 /etc/ 目录

wsl --unregister Manjaro
wsl --set-default Manjaro
wsl -d Manjaro

# 该命令将会更新系统的所有软件包和依赖项，并且也会更新内核和其他系统组件。
sudo pacman -Syu # 保持系统更新
# 如果有需要，你可以在更新之前先同步软件包数据库
sudo pacman -Syyu
```
