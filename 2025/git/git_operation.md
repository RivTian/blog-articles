### git 回滚到某次提交

```sh
git log # 查询 commit id

git reset --hard "commit id"

git push origin HEAD --force # 强制提交至远端仓库