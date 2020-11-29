众所周知，ping命令是个非常实用的网络命令；有时，我们会发现在电脑中无法使用ping命令，一般来说，是由于电脑的环境变量出了问题，本文将介绍如何解决这个问题。

**1.一般出现ping命令无法使用的情况如图：**

![](https://gitee.com//riotian/blogimage/raw/master/img/20201123203012.png)

**2.我遇到的ping命令无法使用的情况，基本都是因为“环境变量”导致的，查看环境变量path，发现没有配置“C:\Windows\System32”这一项：**

**3.接下来直接在“系统变量”中找到“Path”这一项添加即可：**

追加

```
C:\Windows\System32
```

**4.检验是否生效**

![](https://gitee.com//riotian/blogimage/raw/master/img/20201123203303.png)

**至此，ping命令又可以正常使用了：**