---
title: "Git基础使用方法"
date: 2020-11-06T11:58:08+08:00
draft: false
tags:
  - Git
gitinfo: true
---

**概述**

Git是一个免费的开源分布式版本控制系统，设计用于处理从小型到大型的所有项目，速度和效率都很高。

## Git命令行

参考:

- [Git基本应用](https://www.cnblogs.com/nopnog/p/7763008.html)
- [官方Git使用文档](https://www.git-scm.com/book/zh/v2/起步-命令行)

### 提交代码

#### 1. 配置用户

```
git config --global user.email "xxx@mgmail.com"
git config --global user.name "xxx"
```

#### 2. 查看文件状态

```
git status
```

#### 3. 添加到缓存

```
git add test.cpp
或
git add .
```

#### 4. 提交到本地仓库

```
git commit -m "[feature] ...."
```

#### 5. 推送到远端

```
git push
```

### 拉取指定分支

```
git clone -b develop git@.....git
```

### 拉取子模块

```
git submodule sync --recursive
git submodule update --init --recursive
```

### 切换分支

#### 1. 查看当前分支

```
git branch
```

#### 2. 创建分支

```
git checkout -b develop
或
git branch develop
或
git checkout develop
```

#### 3. 切换分支

```
git checkout master // 切换至本地master分支
git checkout -b release/0.0.6 origin/release/0.0.6    // 拉取远端的分支并切换
```

#### 4. 合并分支

```
git merge develop
```

#### 5. 删除分支

```
git branch -d develop
```

### 丢弃某个文件的更改

```
git checkout -- readme.md
```

> - 若 [readme.md](http://readme.md/) 修改后未放到暂存区, 则撤销至和版本库一模一样的状态
> - 若 [readme.md](http://readme.md/) 已添加至暂存区, 又作了修改, 则撤销至之前添加到暂存区的状态.

### 将本地非空工程与远端创建的新仓库关联

```
git remote add origin git@github.com:账号/learngit.git
```

之后可以将本地工程文件提交至远端

```
git push -u origin master
```

### 查看提交记录

```
git log
git log --pretty=oneline // 输出格式比较简洁
```

## 骚操作

### 利用分支重置提交代码

- 用途: 去除无效提交信息至主干分支
- 具体操作流程:
  1. 开发时, 从主干分支中开出功能分支
  2. 开发完毕准备合并入主干分支时, 从最新的功能分支上新建一个新的分支
  3. 然后将该新分支重置到当初从主干分支分出来的节点, 一次性提交功能分支的所有修改
  4. 合并重置分支至主干分支
  5. 删除功能分支
  6. 删除重置分支

### 将另一个远端仓库推送到自己的远端仓库

以下图文教程来自：[王竹兴学长](#)

- 新增远程仓库

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225551.jpeg)

  随便取个名称, 并填充其**git**路径：

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225559.jpeg)

- 拉取另一个远端的**master**到本地**master**

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225608.jpeg)

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225613.jpeg)

- 解决 **git pull** 失败 , 提示：`fatal: refusing to merge unrelated histories`

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225619.jpeg)

  这个问题是因为 两个 根本不相干的 git 库， 一个是本地库， 一个是远端库， 然后本地要去推送到远端， 远端觉得这个本地库跟自己不相干， 所以告知无法合并

  解决办法：把两段不相干的 分支进行强行合并, 再**push**到远端即可。

  ```
  git pull origin master --allow-unrelated-histories
  ```

  - 步骤1

  ![](https://gitee.com//riotian/blogimage/raw/master/img/20201104225625.jpeg)

  - 步骤二