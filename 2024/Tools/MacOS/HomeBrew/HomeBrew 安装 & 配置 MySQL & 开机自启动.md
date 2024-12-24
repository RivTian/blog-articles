

这个是我最新并且一直推崇的方法：
1、安装：`brew install mysql`

2、开启mysql：`mysql.server start`

3、使用mysql的配置脚本：`/usr/local/opt/mysql/bin/mysql_secure_installation //mysql 提供的配置向导`
启动这个脚本后，即可根据如下命令提示进行初始化设置


```bash
14:14:49 with koshkaaaa in ~ took 4.5s
✦6 ➜ mysql_secure_installation // 执行脚步

Securing the MySQL server deployment.
Connecting to MySQL using a blank password.
VALIDATE PASSWORD PLUGIN can be used to test passwordsand improve security. It     checks the strength of password and allows the users to set only those passwords which are secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: k //是否采用mysql密码安全检测插件（这里作为演示选择否，密码检查插件要求密码复杂程度高，大小写字母+数字+字符等）
Please set the password for root here. // 首次使用自带配置脚本，设置root密码

New password:

Re-enter new password:

By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.    
Remove anonymous users? [Y/n] Y //是否删除匿名用户
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y //是否禁止远程登录
 ... Success!

By default, MySQL comes with a database named 'test' that anyone can
access.This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y //删除测试数据库，并登录
 Dropping test database...
 ... Success!
 Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y//重新载入权限表
 ... Success!

All done!  If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!

Cleaning up...
```

4、设置开机启动： `ln -sfv /usr/local/Cellar/mysql/8.0.32/*.plist ~/Library/LaunchAgents`

5、 使用 launchctl 启动 MySQL 服务： `launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist`

6、确认服务是否启动：`mysql -u root -p` 