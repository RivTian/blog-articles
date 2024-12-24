
#Git #SSH

Git Clone 支持 HTTP 和 SSH 两种方式下载源码。

使用 Git 的 SSH 方式下载时，若未配置过 SSH Key 则会提示，没有操作权限、拒绝访问。

下面就介绍一下如何配置 Git 的 SSH Key，以便我们可以用 Git 方式下载源码。

```sh
# 检查一下用户名和邮箱是否配置
git config --global  --list 

# 如未配置，则执行以下命令进行配置
git config --global  user.name "Koshkaaa"
git config --global user.email "pengtianjun@sansi.com"

# 然后执行以下命令生成秘钥
ssh-keygen -t rsa -C "这里换上你的邮箱"

# 执行命令后需要进行3次或4次确认：
# 1. 确认秘钥的保存路径（如果不需要改路径则直接回车）；
# 2. 如果上一步置顶的保存路径下已经有秘钥文件，则需要确认是否覆盖（如果之前的秘钥不再需要则直接回车覆盖，如需要则手动拷贝到其他目录后再覆盖）；
# 3. 创建密码（如果不需要密码则直接回车）；
# 4. 确认密码；

# 在指定的保存路径下会生成2个名为id_rsa和id_rsa.pub的文件：
# 等会需要使用到

# 在如 Github 等仓库中选择 SSH 选项
# 之前生成的是 SSH秘钥，所以下面选择New SSH key（笔者这里已经配置了一个key，如果是未配置秘钥的用户，这里应该是空的）：

# 然后用文本工具打开之前生成的id_rsa.pub文件，把内容拷贝到key下面的输入框，并为这个key定义一个名称（通常用来区分不同主机），然后保存

# 此时使用 Git 则可以正常下载了。
```