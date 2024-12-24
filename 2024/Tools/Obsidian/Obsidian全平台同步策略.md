
> Obsidian是一个纯本地化的基于Markdown语法的编辑器，为了让自己杂乱的设备在使用Obsidian的时候无缝切换，我就实践了一个全平台同步的可行方案。

### 前期资料收集

1.  [Obsidian Sync](https://sspai.com/link?target=https%3A%2F%2Fobsidian.md%2Fsync)
2.  [坚果云官网](https://sspai.com/link?target=https%3A%2F%2Fwww.jianguoyun.com%2F%23%2F)

### 引言

首先需要提一下，Obsidian是有个官方的[云同步方案](https://sspai.com/link?target=https%3A%2F%2Fobsidian.md%2Fsync)，不过96美元/年的定价实在有点Hold不住。本文中提到的方案对于绝大多数人来说应该是能做到零成本的，所以无需有花费上的顾及。

众所周知，iCloud与Window之间是天生的相性不符，而对Android则是直接不支持的。考虑到iOS设备目前第三方只支持iCloud存储，所以首先就排除了全OneDrive方案。

梳理下来问题就很简单了：首先回了顾及iOS设备，苹果全家桶必须要使用iCloud备份；而考虑到iCloud在Windows上糟糕的适配，在Windows上基于iCloud进行同步也是相当不靠谱的。所以问题的中心就放在了**如何将iCloud上的数据同步到第三方云盘**，用以非苹果设备的同步。

### 硬件需求

一台**MacOS电脑** - 在iOS和非苹果设备同步时需要保持开机并联网状态

### 软件需求

**坚果云** - **MacOS电脑**和其它需要同步的桌面平台（**Windows**/**Linux**）  
**FolderSync** - **Android设备**同步需要安装此软件

### 配置流程

#### MacOS电脑端

1.  安装Obsidian客户端
2.  在iCloud云盘新建`Obsidian`文件夹，并将现有的**Obsidian库**拖拽至该文件夹内，手动更新Obsidian客户端内的库路径索引
3.  注册并安装坚果云客户端
4.  在iCloud云盘的`Obsidian`文件夹右击，选中坚果云选项中的**仅为我同步**配置同步信息
5.  待库同步结束后便可以在非苹果设备下进行同步操作

#### Windows / Linux 电脑端

1.  安装Obsidian客户端
2.  安装坚果云客户端
3.  将**Obsidian默认库**选定为坚果云中刚才同步的文件夹
4.  可以将坚果云中该文件夹设置为始终保持在本设备上，方便后续使用
5.  配置结束

#### Android端

1.  安装Obsidian客户端
2.  安装[FolderSync](https://sspai.com/link?target=https%3A%2F%2Fplay.google.com%2Fstore%2Fapps%2Fdetails%3Fid%3Ddk.tacit.android.foldersync.lite)客户端，这里免费版就够用了不需要选择付费版
3.  在账户选项中新建**WebDAV账户**
    -   服务器地址： https://dav.jianguoyun.com/dav/
    -   账户：（坚果云账户）
    -   密码：（应用密码） - 在坚果云Web端的`账户信息/安全选项/第三方应用管理`中**生成授权**
4.  在FolderSync中将**坚果云Obsidian文件夹**与**Android端Obsidian文件夹**建立双向文件夹同步
5.  等同步完成后便可正常使用了

#### iOS端

1.  安装Obsidian客户端，等待iCloud文件下载到本地即可

### 总结

Obsidian的文档文件空间占用极小，**合理利用图床的话总文档大小不会超过一张图片的体积**。即使选择本地方案，iCloud的5GB空间和坚果云的每月1GB上传/3GB下载也是绰绰有余的。

另外，iCloud在苹果设备中的可靠性和坚果云在国内的网络速度都是很可靠的，只要不是真正的多端无时差同时操作同一个文件，文件版本冲突的情况是极少会出现的。