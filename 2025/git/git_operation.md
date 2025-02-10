## git 回滚到某次提交

```sh
git log # 查询 commit id

git reset --hard "commit id"

git push origin HEAD --force # 强制提交至远端仓库
```

## git submodule

> [参考链接 🔗](git_submodule_wiki.md)

```sh
# 子模块的添加
# url  子模块的路径, 一般为 git 链接
# path 子模块存储的目录路径
git submodule add <url> <path>

git diff --cached # 查看修改内容可以看到增加了子模块，并且新文件下为子模块的提交hash摘要

git commit提交即完成子模块的添加
```