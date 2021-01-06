上个月刚安装了 Manjaro ，然后最近在Manjaro下载Github的项目竟然只有几十b/s，这能忍？对于下载Github上的代码是硬需求，没办法直接探索一下突破的方法了。

### 方法一：安装chrome的github加速插件

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106113501.png)

然后就可以直接通过加速链接 下载了，如图所示。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106113542.png)

### 方法二：通过代理的方式，一步搞定，如果你有代理，那么一定是这么玩的。

```text
export ALL_PROXY=socks5://127.0.0.1:1080
```

测试一下TCP成功到达谷歌服务器，就说明咱们终端的TCP已经走代理了。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106113638.jpeg)

这张图片来着知乎大大的实验图。

然后就可以从Github轻松的下载代码了，速度极快的

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106113728.png)

### 方法三：Gitee镜像仓库

码云上有个镜像仓库，虽说也可以在码云里导入github上的仓库，但本质来说，如果这个仓库存在于码云的镜像仓库里，到时候应该是直接从码云里的镜像仓库导入的。

目前码云的镜像仓库里有20000+个项目了（时间截至21.1.6) ，基本一些有名气的都有。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106113851.png)



> 如果没有名气的话，你可以在码云上建立仓库时选择从github上导入仓库，就这么简单。下载时基本上可以跑满你的宽带，就很舒服。和之前的对比，就像滴滴答答的水龙头，和一泻千里的瀑布。用起来感觉太爽太舒服了。话就说到这，感觉再说下去就要被疑车无据了。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106114443.png)

附：导入仓库的方法：

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106114546.png)

![image-20210106114612956](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106114615.png)

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210106114707.png)



