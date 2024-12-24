#Win #镜像 #ISO

> 近日需要使用Windows Server的ISO镜像，担心第三方整理的镜像有污染，就去官方找了一下，没想到真的找到了直链，现在分享一下。

## 简单说明

1. 带有`Essentials`后缀的相当于官方精简版，功能比较少但是性能诉求也更小，不过Essentials版本并不支持无GUI的Core模式安装，因此如果要大规模部署还是用其他版本的Core模式性能更好。
2. Windows Server 2008 R2的官方支持已于2020年1月14日结束，如果不是特别需求建议不要使用。建议不要因为配置较低而选择Windows Server 2008 R2，因为即使是Windows Server 2019 Essentials也能够支持到最低512MB内存，而更低内存配置则强烈建议安装Alpine、Ubuntu Server、CentOS、Debian等Linux发行版。

## 一些安装的提示

### Windows Server 2008 R2

默认试用期为10天，可以重置5次。按Win+R打开“运行”，输入`slmgr.vbs –rearm`以重置试用期。重置5次后失效，需要继续使用只能重装（已经买不到正版了），建议虚拟机使用时利用好快照功能。

### Windows Server 2012/2012 R2/2016/2019

安装时跳过输入密钥部分即可，默认带有180天试用期。

## Windows Server 2012 R2/2016/2019 Essentials

Essentials版本安装的时候必须输入密钥才能继续，以下为官方提供的带有180天试用期的密钥：

- **Windows Server 2012 R2 Essentials** R9N79-23MWD-MBP9B-KHF8Q-C36WX
    
- **Windows Server 2016 Essentials** NCPR7-K6YH2-BRXYM-QMPPQ-3PF6X
    
- **Windows Server 2019 Essentials** NJ3X8-YTJRF-3R9J9-D78MF-4YBP4
    

## 下载地址

- [Windows Server 2008 Standard](https://download.microsoft.com/download/D/D/B/DDB17DC1-A879-44DD-BD11-C0991D292AD7/6001.18000.080118-1840_x86fre_Server_en-us-KRMSFRE_EN_DVD.iso)
    
- [Windows Server 2008 R2](https://download.microsoft.com/download/F/3/8/F384E78B-8F1D-42A6-A308-63E45060E823/7601.17514.101119-1850_x64fre_server_eval_zh-cn-GRMSXEVAL_CN_DVD.iso)
    
- [Windows Server 2012](https://download.microsoft.com/download/2/8/7/287E63DD-3A27-4469-9094-5F7B7C4BF828/9200.16384.WIN8_RTM.120725-1247_X64FRE_SERVER_EVAL_ZH-CN-HRM_SSS_X64FREE_ZH-CN_DV5.ISO)
    
- [Windows Server 2012 R2 Essentials](https://download.microsoft.com/download/8/1/6/816441D4-3F4E-412C-847A-2463146FAF25/9600.16384.WINBLUE_RTM.130821-1623_X64FRE_SERVER_SOLUTION_ZH-CN-IRM_SSSO_X64FRE_ZH-CN_DV5.ISO)
    
- [Windows Server 2012 R2](https://download.microsoft.com/download/D/6/7/D675380B-0028-46B3-B47F-A0646E859F76/9600.17050.WINBLUE_REFRESH.140317-1640_X64FRE_SERVER_EVAL_ZH-CN-IR3_SSS_X64FREE_ZH-CN_DV9.ISO)
    
- [Windows Server 2016 Essentials](https://download.microsoft.com/download/5/E/9/5E9219C2-57D9-43D7-95A5-5915B2C372BB/14393.0.161119-1705.RS1_REFRESH_SERVERESSENTIALS_OEM_X64FRE_ZH-CN.ISO)
    
- [Windows Server 2016](https://download.microsoft.com/download/B/5/F/B5F1A996-B590-45FD-BA99-DE7E745A0882/14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_ZH-CN.ISO)
    
- [Windows Server 2019 Essentials](https://software-download.microsoft.com/download/pr/17763.737.190906-2324.rs5_release_svc_refresh_SERVERESSENTIALS_OEM_x64FRE_zh-cn_1.iso)
    
- [Windows Server 2019](https://software-download.microsoft.com/download/pr/17763.737.190906-2324.rs5_release_svc_refresh_SERVER_EVAL_x64FRE_zh-cn_1.iso)