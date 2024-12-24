#WSL2 

```sh
12:59:55 with riotian in nvim/lua/plugins took 1m 15.9s …
➜ npm --version
zsh: /mnt/d/Soft/nodejs//npm: bad interpreter: /bin/sh^M: no such file or directory
# 然后再显示 wsl 的npm 版本
```

## Why

原因是windows也安装了npm，npm直接调用到了 windows 的，然后可能是因为 windows 和 Manjaro 的换行符不一样导致 sh 运行不了。


## Solve

直接下载已经编译好的 node 的 linux 安装包 Current Version(v20.11.1) 

Download: [nodejs](https://nodejs.org/en/download/)

我把软件都放在了 `/usr/local/software/` 下

```sh
cd /usr/local && mkdir software && cd software
wget https://nodejs.org/download/release/v20.11.1/node-v20.11.1-linux-x64.tar.xz

# unzip
sudo tar xvf node-v20.11.1-linux-x64.tar.xz

# rename
sudo mv node-v20.11.1-linux-x64 ./node-v20.11.1

# 强迫症
sudo rm -rf node-v20.11.1-linux-x64.tar.xz
```

配置环境变量

```sh
sudo vim /etc/profile

# 添加在末尾
export NODE_HOME=/usr/local/software/node-v20.11.1 export PATH=$NODE_HOME/bin:$PATH 

# 保存，使配置生效 
source /etc/profile
```

如果和我一样用的是  [oh-my-zsh](https://so.csdn.net/so/search?q=oh-my-zsh&spm=1001.2101.3001.7020)

```sh
sudo vim ~/.zshrc

# 在末尾加上
export NODE_HOME=/usr/local/software/node-v20.11.1
export PATH=$NODE_HOME/bin:$PATH

# 保存，使配置生效
source /etc/profile
```

#### 这里还有一个小细节！！

`export PATH=$NODE_HOME/bin:$PATH` 我的 `$PATH` 是在 bin 后面的

平常配置环境变量通常都是前面的`export PATH=$PATH:$NODE_HOME/bin`

那就会引发这个问题：node 版本正常。可是 npm 却始终到 window 的环境变量去找。所以 `$PATH` 写的位置一定要非常注意！

#### 接下来就是一样的命令操作了~

在 window 下安装过的全局 npm 包，在 wsl 就不用重新安装了，全局 npm 包倒是通用的