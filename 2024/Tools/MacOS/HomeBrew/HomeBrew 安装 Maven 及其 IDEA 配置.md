

## Brew 安装 Maven

```bash
brew search maven # 使用搜索工具去搜索maven包
brew info maven   #使用info查看maven包当前的信息情况，包括版本依赖描述等
brew install maven
```

安装过程很顺利，这时候brew就已经帮我们做好了环境变量了。很多教程在这一步后会手动去生命maven的目录情况，我看了几个教程里面的设置完全没有道理，根本链接的不是brew的安装位置。

> 但是对于 JDK，我使用的是在 IDEA 中安装的 "Azul Systems, Inc." - "Zulu 8.68.0.21"
>
> 所以会发生 HomeBrew 安装的 Maven 中路径与 Java 路径不匹配的情况
>
> 即在安装maven 后输入 `mvn -v` 不能正确显示



### JDK 不匹配解决方案

使用 HomeBrew  安装的 JDK 不用设置 JAVA_HOME 也可以使用，是因为 Java 可执行命令在 `/usr/bin/java` 下有导致使用了 JAVA_HOME 的 Maven 找不到 JAVA_HOME 没有设置

```bash
/usr/libexec/java_home -V
Matching Java Virtual Machines (1):
    1.8.0_362 (x86_64) "Azul Systems, Inc." - "Zulu 8.68.0.21" /Users/koshkaaaa/Library/Java/JavaVirtualMachines/azul-1.8.0_362/Contents/Home

# 将输出的路径添加到 ~/.zshrc 中
JAVA_HOME="/Users/koshkaaaa/Library/Java/JavaVirtualMachines/azul-1.8.0_362/Contents/Home"
export JAVA_HOME
CLASS_PATH="$JAVA_HOME/lib"
PATH=".$PATH:$JAVA_HOME/bin"

# 更新 zshrc
source ~/.zshrc
```

此时使用 `mvn -v` 问题得到解决

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/imgimage-20230406152237858.png)

## 配置 Maven

可以看到 brew 自动帮我装好了x86版本的maven。如果是m1的芯片，会帮你装m1版本，路径位置可能不同，但设置没啥区别，注意换成自己路径即可。我们接下来看maven的配置文件位置。

上面maven信息输出中有***Maven home: /usr/local/Cellar/maven/3.9.1/libexec***,冒号后面就是brew帮我们下载的安装目录。直接执行***cd Maven home: /usr/local/Cellar/maven/3.9.1/libexec***，进入目录。

```bash
cd /usr/local/Cellar/maven/3.9.1/libexec
ls
bin  boot conf lib
cd conf 
ls 
logging        settings.xml   toolchains.xml
# 看到了熟悉的 setting 文件
```

在xml文件中，我们要关注两个地方，我在下面列了出来***localRepository***和***mirrors***。第一个localRepository是你本地仓库所在的位置，你的包都会下载到这里，默认在你用户目录的.m2目录下，我觉得挺直观的，就没修改了，这里可以换成你想存放的地址。第二个mirror是你的包下载地址，因为有墙，所以建议增加阿里云仓库配置，来加速下载。具体配置如下，可以自己在xml中寻找。

```xml
<mirrors>
  <!-- mirror
   | Specifies a repository mirror site to use instead of a given repository. The repository that
   | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
   | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
   |
  <mirror>
    <id>mirrorId</id>
    <mirrorOf>repositoryId</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://my.repository.com/repo/path</url>
  </mirror>
   -->
  <mirror>
    <id>maven-default-http-blocker</id>
    <mirrorOf>external:http:*</mirrorOf>
    <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
    <url>http://0.0.0.0/</url>
    <blocked>true</blocked>
  </mirror>
  <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
  </mirror>

</mirrors>
```

到这里，我们的maven已经安装完毕。仓库位置在我们本地用户目录下的.m2中，maven安装位置为之前的maven home。我们开始为IDEA配置本地maven环境。



## IDEA 的 Maven 配置

我们打开idea的偏好设置，搜索maven，出现以下的配置界面。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230406153708577.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230406154230485.png)

我们将安装目录和配置目录替换成配置maven时确定的路径，不好通过访达选文件就直接把路径复制填上去，应用保存即可。