
#Docker

> **Reference**
> 
> - **[https://www.willh.cn/articles/2022/07/13/1657676401964.html](https://www.willh.cn/articles/2022/07/13/1657676401964.html)**
>     

Docker默认安装在C盘： "C:\Program Files\Docker" 文件夹下。

本文将Docker安装在D:\Program Files\Docker文件夹下

1、用管理员身份打开 Powershell 窗口，然后运行如下命令：

```shell
cmd /c mklink /j "C:\Program Files\Docker" "D:\Program Files\Docker"
```

以上代码输出如下：

 为 C:\Program Files\Docker <<===>> D:\Program Files\Docker 创建的联接

注意需要保证D:\Program Files\Docker存在（如无，自己创建对映名字文件夹，否则在安装Docker时会报错）。

> 前提 C 盘的 Docker 目录也不存在，即尚未安装 Docker

2、官网下载Docker安装包，执行安装

成功将Docker安装在D:\Program Files\Docker文件夹下。

### **Docker下载的image仍保存在C盘怎么办？**

打开Docker➡点击设置➡Resources➡更改imge存储路径到D盘➡Apply保存更改

![](file://D:\Coding\GitLab\personal-docs\Docs\Tools\asserts\image-20231121111520557.png?lastModify=1706162816)

---

## Linux 环境下 Docker Image 下载位置修改

通过修改 Docker 文件路径的方式， 达到修改镜像默认下载路径的目的。

一、查看 Docker 安装位置。 默认 Docekr 的安装位置是在 `/var/lib/docker` 。

 # 默认存放位置  
 $ sudo docker info | grep "Docker Root Dir"

二、停止 Docker 服务

 # 重启 Docker  

```sh
systemctl restart docker  
```
 ​  
 # 启动 Docker   

```sh
systemctl start docker  
```
 ​  
 # 停止 Docker  

```sh
systemctl stop docker
```

三、新建软链接。 Docker 默认位置安装是 `/var/lib/docker`， 系统在配置文件中设置了这个路径，会去这个路径寻找 Docker。我们把整个 Docker 文件移动到其他位置， 然后通过软链接的方式在源路径下创建一个关联到新路径的软链接。

 # 移动原有的内容  

```sh
mv /var/lib/docker /data/docker  
```
 ​  
 # 进行链接  

```sh
ln -sf /data/docker /var/lib/docker
```