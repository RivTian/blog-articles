

通过使用 Homebrew，可以大大降低在 Mac OS X 上设置和配置开发环境的成本。

让我们安装 Redis。

```bash
$ brew install redis
```

安装后，我们将看到一些有关配置注意事项的通知。 离开它并继续关注本文中的一些任务。

#### 开机自启动 Redis

```bash
$ ln -sfv  /usr/local/Cellar/redis/7.0.10/*.plist  ~/Library/LaunchAgents
```

#### 通过“launchctl”启动 Redis 服务器

```bash
$ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

#### 使用配置文件启动 Redis 服务器

```bash
$ redis-server /usr/local/etc/redis.conf
# 似乎会报错
```

#### 在计算机启动时自动启动时停止Redis

```bash
$ launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

#### Redis 配置文件的位置

```bash
/usr/local/etc/redis.conf
```

#### 卸载 Redis 及其文件

```bash
$ brew uninstall redis
$ rm ~/Library/LaunchAgents/homebrew.mxcl.redis.plist
```

#### 获取 Redis 包信息

```bash
$ brew info redis
```

#### 测试 Redis 服务器是否正在运行

```bash
$ redis-cli ping
```

如果它回复“PONG”，那就说明我们安装成功，并且服务可以正常运行了！