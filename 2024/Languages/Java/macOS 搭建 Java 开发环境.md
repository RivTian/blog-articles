#Java #Maven 

## 一、关于 JDK 的选择

[OpenJDK 和 OracleJDK 哪个jdk更好更稳定，正式项目用哪个呢？](https://www.zhihu.com/question/353325963)

对我来说，个人项目使用 Zulu JDK，企业项目使用标准的 Oracle JDK 为准

## 二、Maven 和 Gradle 选择

这一项，根据国内社区活跃状态，可以无脑选择 Maven

## 三、开始配置环境

todo！
```sh
brew install --cask oracle-jdk

# Config path
# Java 
export JAVA_HOME21="/Library/Java/JavaVirtualMachines/jdk-21.jdk/Contents/Home"
export PATH=$PATH:$JAVA_HOME21/bin

# 验证版本
java -version
javac --version

brew install maven
mvn -v
```

> [mvn -v 不显示的问题](https://www.cnblogs.com/RioTian/p/17293008.html)


###  什么是/Usr/Libexec/Java_home

在MacOS X 10.5或更高版本上，我们可以使用`/usr/libexec/java_home`返回默认JDK的位置。

```sh
❯ /usr/libexec/java_home
/Users/koshkaaa/Library/Java/JavaVirtualMachines/azul-21.0.2/Contents/Home
```

另外，查找所有已安装的jdk。

```sh
❯ /usr/libexec/java_home -V                                         
Matching Java Virtual Machines (1):
    21.0.2 (arm64) "Azul Systems, Inc." - "Zulu 21.32.17" /Users/koshkaaa/Library/Java/JavaVirtualMachines/azul-21.0.2/Contents/Home
/Users/koshkaaa/Library/Java/JavaVirtualMachines/azul-21.0.2/Contents/Home
```

另外，运行指定的JDK命令。

```sh
❯ /usr/libexec/java_home -v1.8             
/Users/koshkaaa/Library/Java/JavaVirtualMachines/azul-21.0.2/Contents/Home
```