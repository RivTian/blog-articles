## 使用Hugo+Github Actions实现自动化部署博客

> *很早就想搭建一个自己的博客了，因为某些原因一直为付诸行动。直到近期自己对建博的需求愈发强烈，在尝试过了各类流行的方案后，选择Hugo+Github Action来实现静态博客的自动化部署。这里也将整理过的流程记录下来，也作为新博客的第一篇文章。*

# 一、环境准备

安装[Git](https://git-scm.com/)、[Hugo](https://gohugo.io/)，注册[Github](https://github.com/)账号，网上教程很多就不再赘述，我使用MacOS系统，Windows系统用户可自行调整。

注：请自行将下文中的`My_blog`、`RivTian`替换为自己的仓库名、站点名、文件夹名、用户名二、初始化Hugo项目

# 二、初始化Hugo项目

## 1.新建站点

本地安装好各类环境后，终端通过`hugo new site My_blog`新建站点，生成hugo站点的默认目录，目录树如下：

```
My_blog  
├─ archetypes     
│  └─ default.md  
├─ content        
├─ data           
├─ layouts        
├─ static         
├─ themes         
└─ config.toml    
```

## 2.安装主题

Hugo默认不带主题，可以在[官方主题仓库](https://themes.gohugo.io/)选择喜欢的主题，本次我使用了的[memE主题](https://io-oi.me/tech/documentation-of-hugo-theme-meme/)，在主题页面下载好主题文件后，将主题文件复制到`themes`文件夹下，并重命名为`meme`，并将主题的示例文件`exampleSite`目录下的示例文件复制到项目根目录下替换原有文件：

# 三、Git仓库配置

在Github上创建名为`My_blog`的公开仓库，创建时无需进行初始化

依次在终端中执行以下命令：

1. `git init`初始化仓库
2. `git remote add origin https://github.com/RivTian/My_blog.git`连接远端仓库
3. `git checkout -b develop`切换至develop分支（[不使用master分支的原因？](https://weibo.com/ttarticle/p/show?id=2309404516196870390640)）
4. `git add .`暂存全部文件
5. `git commit -m "first commit"`提交
6. `git push -u origin develop`推送到远端仓库

执行完毕后去Github上看下文件是否都已上传，顺手在Github上新建一个`gh-pages`分支

# 四、自动化部署

在Github中的[创建一个Token](https://github.com/settings/tokens)(记得在页面关闭前保存下来，页面一旦关闭就再也看不到了),回到`My_blog`项目仓库中在`Settings/Secrets`创建一个`Secrets`，标题命名为`personal_token`，内容填入刚刚创建的`Token`。

本地`My_blog`目录中新建`.github/workflows/deploy.yml`，内容填入以下代码：

```
name: Deploy Hugo

on:
  push:
    branches:
      - develop

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1  # v2 does not have submodules option now
        # with:
          # submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          # extended: true

      - name: Build
        run: hugo

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.personal_token }} # 这里的 ACTIONS_DEPLOY_KEY 则是上面设置 Private Key的变量名
          PUBLISH_BRANCH: gh-pages
          PUBLISH_DIR: ./public
          commit_message: ${{ github.event.head_commit.message }}
```

依次在终端中执行以下命令：

1. `git add .`暂存全部文件
2. `git commit -m "Github Action自动化部署"`提交
3. `git push -u origin develop`推送到远端仓库

等待Github仓库中Actions`Deploy Hugo`执行完毕后，看下`gh-pages`分支是否已生成HTML文件。

# 五、其他补充内容

## 1.绑定域名

详细介绍：[Click Here](GitHub Pages 绑定域名.md)

在本地`static`目录下新建`CNAME`文件，内容填入绑定的域名`www.mlosun.com`，同时也需要将域名解析至`mlosun.github.io.`

后依次在终端中执行：

1. `git add .`暂存全部文件
2. `git commit -m "绑定域名"`提交
3. `git push -u origin develop`推送到远端仓库

## 2.修改主题配置

可以参考这位前辈的文章：[Here](https://ztygcs.github.io/posts/meme%E4%B8%BB%E9%A2%98%E4%BC%98%E5%8C%96/)

## 3.访问测试

尝试访问[https://www.riotian.com/](https://www.riotian.com/)，看是否能顺利打开，页面显示是否正常等.

# 六、参考资料

- [Hugo + Github Actions 实现自动化部署](https://immmmm.com/hugo-github-actions/)
- [用 Hugo 配合 Github Actions 自动构建我的博客](https://www.nashome.cn/posts/hugo-github-actions/)
- [Deploy Hugo on Github Pages](https://piggy.site/posts/deploy-hugo-on-github-pages/)