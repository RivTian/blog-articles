#Manjaro 

Yet Another Yogurt: 一个用于 Arch Linux 的工具，用于从 Arch User Repository 中构建和安装软件包。 另见 [pacman](obsidian://open?vault=Obsidian_Notes&file=Tools%2FWSL2%2FArch(Manjaro)%20Linux%20Pacman%20%E5%91%BD%E4%BB%A4%E8%AF%A6%E8%A7%A3).

> `pacman` 的命令基本 yay 都涵盖了，执行时 `pacman` 替换成 `yay` 即可

- 刷新系统包并升级

`yay -Syu`

- 从仓库和 AUR 中交互式搜索和安装软件包：

`yay {{软件包|搜索词}}`

- 同步并更新所有来自仓库和 AUR 的软件包：

`yay`

- 只同步和更新 AUR 软件包：

`yay -Sua`

- 从仓库和 AUR 中安装一个新的软件包。

`yay -S {{软件包}}`

- 从仓库和 AUR 中搜索软件包数据库中的关键词：

`yay -Ss {{关键词}}`

- 显示已安装软件包和系统健康状况的统计数据：

`yay -Ps`

- 删除任何包

`yay -Rns <pkg_name>`

