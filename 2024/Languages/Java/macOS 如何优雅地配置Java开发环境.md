## 一、准备

确保 HomeBrew 存在

以下命令即可安装 HomeBrew

```sh
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/bottles"

/bin/bash -c "$(curl -fsSL https://gitee.com/ineo6/homebrew-install/raw/master/install.sh)"
```

## 二、安装并配置 JDK

### 2.1 安装 OpenJDK

`brew search openjdk` 查看版本信息(如下图)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/pGb6mD.png)

`brew info openjdk@8`，查看 openjdk@8 的安装描述信息

>  **描述信息中清楚的知道我的当前mac环境不支持安装**
>
> 所以下面安装 openjdk@21 版本

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/image-20241108171646845.png)

1. Dependencies: 安装openjdk@8需要依赖其他package
2. Analytics: openjdk@8现阶段的安装统计相关信息

`brew install openjdk@21`，开始安装

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/b4LeUR.png)

如上图所示，openjdk@21 就安装好了。

### 2.2 配置 JDK

Mac下让安装的JDK生效及可识别需要如下几步操作:

1. 执行下方命令

```sh
sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
```

2. 检查 `tree /Library Java`

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/Buxp4U.png)

3. 执行 `/usr/libexec/java_home`

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/G3xzO1.png)

至此基本配置完成，查看一下安装好的Java版本。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/BxX6CR.png)



## 三、最后

回归主题，这种办法安装后的优雅性体现为如下两点：

1. 执行如下命令，即可自动卸载。

```sh
brew uninstall openjdk@21
```

2. 在安装了多版本jdk后，可以优雅地快速进行版本切换。

   **以安装jdk8和jdk11为例**

```sh
➜  ~ tree /Library/Java 
/Library/Java
├── Extensions
└── JavaVirtualMachines
    ├── openjdk-11.jdk -> /usr/local/opt/openjdk@11/libexec/openjdk.jdk
    └── openjdk-8.jdk -> /usr/local/opt/openjdk@8/libexec/openjdk.jdk

4 directories, 0 files
```

3. **设置JAVA_HOME**
   `/usr/libexec/java_home`可以指定JDK版本，如下：

```sh
➜  ~ /usr/libexec/java_home -v1.8
/usr/local/Cellar/openjdk@8/1.8.0+282/libexec/openjdk.jdk/Contents/Home
➜  ~ /usr/libexec/java_home -v11
/usr/local/Cellar/openjdk@11/11.0.9/libexec/openjdk.jdk/Contents/Home
```

​	**基于此特性可以采用如下策略：**

```sh
export JAVA_HOME=$(/usr/libexec/java_home -v11)
export JAVA_8_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_11_HOME=$(/usr/libexec/java_home -v11)

alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'
```

至此基于alias就可以实现一个Terminal实例下的JDK版本切换。
