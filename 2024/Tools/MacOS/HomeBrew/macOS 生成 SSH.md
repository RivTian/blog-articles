```bash
# 检查 SSH 是否存在
ls -a ~/.ssh

# 如果已经存在，则会显示 id_rsa 和 id_rsa.pub
# 查看 .ssh
cd .ssh 
cat id_rsa.pub # 查看公钥 
cat id_rsa     # 查看私钥

# 生成新的 SSH key
# 设置 username 和 email（github 每次 commit 都会记录他们）
git config --global user.name "RivTian"
git config --global user.email "kannajiahe@gmail.com"

# 其中 your_email@example.com 为你在 GitHub 注册时的邮箱
ssh-keygen -t rsa -C "kannajiahe@gmail.com"

# 添加 key 到 SSH
ssh-add ~/.ssh/id_rsa

## 添加 SSH key 到 GitHub 和 coding
# 复制 id_rsa.pub 中的所有内容
code ~/.ssh/id_rsa.pub

# 手动复制以 ssh-rsa 到以 your_email@example.com 结尾的所有内容
# 或者直接输入命令复制 id_rsa.pub 中的所有内容，终端命令：
pbcopy < ~/.ssh/id_rsa.pub

# ## 登录 GitHub 和 coding
#### github：打开个人 Settings–>SSH keys–>Add SSH key

#### coding：打开个人设置–>SSH 公钥 –>新增公钥
# Title 随便写 Key 粘贴之前复制的内容 这样 SSH key 就添加的 GitHub 或者 coding.

## 检测 SSH key
ssh git@github.com
# 成功显示如下：
# Hi your_name! You’ve successfully authenticated, but GitHub does not provide shell access.
```

```bash
## 已经存在 ssh，创建新密钥
### 进入 ssh 目录
### 用 ssh-keygen 命令生成一组新的 id_rsa_new 和 id_rsa_new.pub
ssh-keygen -t rsa -C "new email"

# 需要注意，出现提示输入文件名的时候要输入与默认配置不一样的文件名，比如： id_rsa_new

### 执行 ssh-agent 让 ssh 识别新的私钥
ssh-add ~/.ssh/id_rsa_new
### 添加 SSH 到钥匙串
ssh-add -K KeyPathOrKeyName
```