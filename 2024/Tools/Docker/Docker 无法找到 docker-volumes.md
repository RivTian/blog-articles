* Mac M2 Pro
* Docker 24.0.6

```bash
$ docker volume inspect 14dfdb65fb7075d91b2004c979a3591df54bcc1303ff3ca96a3536f4761a19cc
[
    {
        "CreatedAt": "2023-11-21T12:52:27Z",
        "Driver": "local",
        "Labels": {
            "com.docker.volume.anonymous": ""
        },
        "Mountpoint": "/var/lib/docker/volumes/14dfdb65fb7075d91b2004c979a3591df54bcc1303ff3ca96a3536f4761a19cc/_data",
        "Name": "14dfdb65fb7075d91b2004c979a3591df54bcc1303ff3ca96a3536f4761a19cc",
        "Options": null,
        "Scope": "local"
    }
]
```

嘗試進入這個路徑時，發現它並不存在

```bash
$ ls /var/lib/docker
ls: /var/lib/docker: No such file or directory
```

網路上有許多解決方式是使用下方指令，但仍錯誤。

```bash
$ screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty
```

嘗試之後直接閃退：

```bash
[screen is terminating]
```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/FAwUDb.png)

## 1.解決閃退問題方式

詳細可參考此篇：[Where is /var/lib/docker on Mac/OS X](https://stackoverflow.com/questions/38532483/where-is-var-lib-docker-on-mac-os-x/65645462#65645462)  
mac下docker實際是在vm裡又加了一層，因此需要進入vm 才能進行操作。

* 終端機執行下方指令

```bash
docker run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```

## 2.檢視volumes

解決閃退問題後，會進入VM內，輸入ls，檢視當前路徑下目錄資訊。

```bash
ls /var/lib/docker/volumes/html/_data
```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/xl8zWw.png)

退出：exit

## 補充：

### 進入Mountpoint對應的資料夾 (Linux)

如果使用Linux，可以直接找到Mountpoint對應的目錄，就是和container連接的地方，這裡面的改動和container內是同步的。  
但如果是Mac，用同樣的方式想要進入Mountpoint對應的目錄，會不存在，

Mac需要先創建一個Linux的VM，所以Mountpoint對應的不是Mac裡可以找得到的檔案，而是要到那個VM裡去找，

> 补充完整的命令图

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/ShT5N9.png)

