>  开始学习 Jmeter

## 环境准备

Mac 安装 Jmeter 可以从官网下载，也可以使用 Brew 直接安装

1. 官网：[Jmeter Download](https://jmeter.apache.org/download_jmeter.cgi)

2. `brew install jmeter`

   `jmeter -v` 查看版本

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230416115633212.png)

Jmeter 的最新版本是 $5.5$，官网备注的 JDK 要求 JDK8+



## 启动 Jmeter

终端下，直接输入 `jmeter` 命令，启动 **Jmeter**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230416115844026.png)

启动之后我们可以看到终端打印的信息：不要使用GUI模式进行负载测试，GUI只用于创建脚本以及用来debug，执行测试时建议使用非GUI模式运行。后面紧接着的就是命令行模式的命令提示。

Jmeter 启动成功。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230416115931967.png)



### Jmeter 使用

