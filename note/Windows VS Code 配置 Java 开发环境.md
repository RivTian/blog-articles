> **前篇：** **[Windows VS Code 配置 C/C++ 开发环境](https://www.cnblogs.com/RioTian/p/13812319.html)**

### 准备

- Windows 【这个相信大家都有 笑: )】
- [VS Code](https://code.visualstudio.com/)
- [JDK](https://www.oracle.com/java/technologies/javase-downloads.html) 建议 JDK8以上（不包含JDK8，[关于 Windows环境下JDK安装与环境配置教程](https://www.cnblogs.com/liuhongfeng/p/4177568.html)）

提前安装好 VS Code 和 配置好 JDK环境（一般安装好JDK后应该重启电脑）。

### 安装 Java Extension Pack 插件

进入 VS Code，键入 `Ctlr + Shift + X` 打开扩展库输入 **Java Extension Pack**，一般是显示在第一个然后安装

![](https://gitee.com//riotian/blogimage/raw/master/img/20210131221440.png)

又或者点击链接安装：https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack

![](https://gitee.com//riotian/blogimage/raw/master/img/20210131221525.png)

> **Java Extension Pack**是集成6个扩展的一个插件，所以安装一个即可。

### 选择 JDK 编译版本

在VS Code界面键入：`Ctrl+Shift+P`，然后输入：`Java: Configure Java Runtime` 进入页面确认 JDK是否导入能正常使用。

如果提前安装好 JDK 以后这里会显示当前 JDK版本号。

![](https://gitee.com//riotian/blogimage/raw/master/img/20210131221634.png)

### 编写并测试 Java 程序

`Ctlr + Shift + E` 打开资源编辑器新建一个 `Java`文件：`Hello.java` 。 

输入：

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

![image-20210131221900114](https://gitee.com//riotian/blogimage/raw/master/img/20210131221900.png)

`F5` 即可编译运行。

> 关于 Code Runner 的运行会找不到 类主体的问题目前我还没找到好的方法解决，之后会补上这部分，如果您有好的方法请务必指导我一下。谢谢！