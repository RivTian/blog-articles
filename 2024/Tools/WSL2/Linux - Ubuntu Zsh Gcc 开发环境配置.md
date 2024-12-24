
```bash
# 修改镜像源
# 此时还没有安装 VSC
sudo gedit /etc/apt/sources.list 
# 删除文件内所有内容后，使用阿里云镜像
# 清华镜像
# https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
```

```bash
# 阿里云镜像
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
# deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

```bash
sudo apt update
sudo apt upgrade

# 安装 VSCode snap 版
# 自带 code 环境配置
sudo snap install --classic code 

sudo apt-get install openssl git zsh
sh -c "$(wget https://gitee.com/Devkings/oh_my_zsh_install/raw/master/install.sh -O -)"
chsh -s /usr/bin/zsh # 使用 zsh

# zsh 配置 安裝 Oh My Zsh
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 字体配置教程
# https://blog.51cto.com/bianchengsanmei/4980827
# 字体本体
# https://github.com/seanghay/JetBrainsMono-Powerline/blob/master/JetBrains%20Mono%20Regular%20for%20Powerline.ttf
# 主题
git clone https://github.com/spaceship-prompt/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt" --depth=1

# 软连接
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"

sudo apt-get install python
# 安装一些 zsh 插件
# https://zhuanlan.zhihu.com/p/550022490
# https://sspai.com/post/63241
git clone https://gitee.com/mirror-github/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting

git clone https://gitee.com/mirror-github/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions

git clone https://gitee.com/mirror-github/autojump $ZSH_CUSTOM/plugins/autojump

# ~/.zshrc 最后加入
[[ -s /home/koshkaaa/.autojump/etc/profile.d/autojump.sh ]] && source /home/koshkaaa/.autojump/etc/profile.d/autojump.sh

plugins=(git zsh-syntax-highlighting zsh-autosuggestions autojump)

source ~/.zshrc


# 如果 autojump clone 不下来直接去 Github 下源码

# 安装基本编译环境
sudo apt-get install build-essential cmake libc6-dev-i386

# sudo -i 进入 root
# 拷贝arm_linux_4.8.tar.gz、Tools.4.9.3.tar.gz、FriendlyARM.tar.gz
# 至/usr/local/tool-chain目录下
# 在 /etc/environment 中添加 3个文件夹的 bin 目录

PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/tool-chain/arm_linux_4.8/bin:/usr/local/tool-chain/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin:/usr/local/tool-chain/Tools.4.9.3/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin"

# 即添加
# /usr/local/tool-chain/arm_linux_4.8/bin
# /usr/local/tool-chain/gcc-linaro-6.3.1-2017.05-x86_64_aarch64-linux-gnu/bin
# /usr/local/tool-chain/Tools.4.9.3/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin
# 三项环境变量配置
# X90A平台：
aarch64-linux-gnu-gcc -v
# X70平台
arm-linux-gcc -v
# X9平台
arm-linux-gnueabihf-gcc -v

# 安装基础编译库
sudo apt-get install pkg-config libasound2-dev libva-dev libgles2-mesa-dev nasm yasm libssl-dev

# 虚拟机安装ssh
sudo apt-get install ssh

# 本地去生成 密钥
ssh-keygen -t rsa -C 'xxx@xxx.com'

# 用记事本打开id_rsa.pub文件，复制其内容至虚拟机~/.ssh/authorized_keys
# VSC 安装 remote ssh 插件
```

```bash
# 在SSH配置文件中加入如下配置并保存
Host xxx	 # 虚拟机的名称
    IdentityFile C:\Users\uid21433\.ssh\id_rsa
    HostName *.*.*.8 # 虚拟机的IP
    User xxx # 登录的用户名（虚拟机中的linux用户名）
```

[ShangHai Sansi@内部参考文档](https://git.sansi.net:6101/common_display/Embedded_Software/x90/xstudiopro/xstudiopro-doc/-/blob/master/design/xStudioPro3.x/%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/xStudioPro3.x%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA%E6%B5%81%E7%A8%8B.md)
