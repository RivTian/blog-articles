#WSL2 
## 启用毛玻璃特性

> 微软还是太狗了，这么好看的毛玻璃效果藏着掖着，今天有幸看到，就有了本篇踩坑记录


官方文档：https://learn.microsoft.com/zh-cn/windows/terminal/custom-terminal-gallery/frosted-glass-theme

### 标签页毛玻璃

设置 --> 外观 --> 开启 Use acrylic material in the tab row (requires relaunch)

如果没有左下角打开JSON配置文件

在最外层{}内加一行

```json
{
    ...
    "useAcrylicInTabRow": true
}
```

![](https://qiu-blog.oss-cn-hangzhou.aliyuncs.com/Article/1679137804431419871.png)

### 终端毛玻璃

设置 --> 默认值 --> 其他设置 --> 外观 --> 下拉找到透明度 --> 启用亚克力不透明度调到70%

![img](https://qiu-blog.oss-cn-hangzhou.aliyuncs.com/Article/1679137518846721308.png)