
#WSL2 

一、安装 OpenSSH

```sh
yay -S openssh
# 查看 sshd status
systemctl status sshd.service
# 如果未开启的话，需要启动sshd命令，执行命令
sudo systemctl start sshd
# 如果想要将sshd命令开机启动的话，同样需要root权限执行
sudo systemctl enable sshd.service
# 查看 IP 地址, ssh 远程连接即可
ip addr
ssh <username>@ip
```

二、开启启动