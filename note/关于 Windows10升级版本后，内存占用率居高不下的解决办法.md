一个月前，打开系统更新，win 10 推送了 最新版本。

然后手贱点了更新。

的确一开始没觉得的有什么明显变化，但最近总觉得机子卡的卡的严重，查看了下内存，占用率居高不下。

经常才打开一两个软件内存就跳到85%以上，但实际去计算一下完全没有这么多。

即时把打开的进程全杀了，也没有用。

然后网上开始找解决方法。研究和折腾了好久，终于找到了解决办法，从注册表入手。

* **准备工作，打开注册表窗口，Win+R 调用Run窗口，并输入 regedit. 之后，打开注册表。**
* 首先，在注册表编辑器窗口，**定位到路径(可直接输入并回车)**：**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem 下**

![注册表示意图](https://gitee.com//riotian/blogimage/raw/master/img/20210320215405.png)

* 其次**，在右侧界面新增(如己存在:**修改)三个注册表项，分别如下:

1：**新增 RefsEnableLargeWorkingSetTrim = 1**
键名:  **RefsEnableLargeWorkingSetTrim**
键值: 1
键值类型: **REG_DWORD  [DWORD(32-bit) Value]** (修改时为十六进制)
设置成  RefsEnableLargeWorkingSetTrim = 1

2：新增 **RefsNumberOfChunksToTrim = 4**
键名:  **RefsNumberOfChunksToTrim**
键值(默认if not set or 0) : 4
值类型: **REG_DWORD  [DWORD(32-bit) Value]** (修改时为十六进制)
设置成 **RefsNumberOfChunksToTrim = 4**

3：新增 **RefsEnableInlineTrim = 1**
键名: **RefsEnableInlineTrim**
键值: 1
值类型: **REG_DWORD  [DWORD(32-bit) Value]** (修改时为十六进制)
设置成 **RefsEnableInlineTrim = 1**

* 最后，修改完成注册表后，**重启电脑**即可。

![任务管理器示意图](https://gitee.com//riotian/blogimage/raw/master/img/20210320215433.png)

参考: https://support.microsoft.com/en-us/help/4016173/fix-heavy-memory-usage-in-refs-on-windows