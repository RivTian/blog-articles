## 域名选购

域名注册商有很多，国内的万网，国外的 GoDaddy 等等。区别在于国内域名注册后需要备案，因为政策因素也可能随时被停用，相对的，国外注册域名在交流和沟通方面不如国内方便，而因为没有国内的政策限制，域名注册商通常会给予用户域名的完全控制权与转移权，在安全性方面可能比国内稍差。

本站域名申请使用国外域名平台 Namesilo，是目前价格较便宜的域名平台，支持微信、支付宝、Paypal、Visa 等多种付款方式，提供免费的域名隐私保护，性价比较高，用户评价也不错。

---

## NS 修改

国外的域名使用默认的域名服务器（NS）解析可能较慢，网上很多人推荐转到 DNSPod，也就是说指派 DNSPod 进行域名的解析工作。

### 1、注册 DNSPod 账号

在 DNSPod 官网注册账号，在域名解析页面选择添加域名，添加已购买的域名。

![image-20201024130720565](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20201024130722.png)

完成后点击查看，DNSPod 提供两条默认的 NS 记录：

```
f1g1ns1.dnspod.net
f1g1ns2.dnspod.net
```

### 2、修改 Namesilo 中 NS 记录

登录 Namesilo，选择 Manage My Domains，进入域名管理页面。勾选你的域名，点击选项栏中的 Change Nameservers。

[![图3](https://tding.top/archives/b48e2719/3.png)图 3](https://tding.top/archives/b48e2719/3.png)

将 NS1，NS2 改为 DNSPod 提供的两条 NS 记录，删除第三条 NS 记录，点击提交。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20201024131029.png)

NS 的修改需要一段时间，一般最长 48 小时生效，个人情况来看，10 分钟左右即可完成更改。

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20201024131129.png)

### 3、DNSPod 解析服务

> 自 2018 年 5 月 1 日，Github 支持自定义域名的 HTTPS 请求了。
>
> 配置也相当简单，只需要更新 DNS 配置里的 A 记录，将其指向以下 4 个 IP 地址中的至少一个。
>
> - 185.199.108.153
> - 185.199.109.153
> - 185.199.110.153
> - 185.199.111.153
>
> HTTPS 让你的网站和网站访客更安全，并且 Github 提供的这些 IP 地址自动将你的站点加入了 CDN，提高了访问速度。你还可以在 GiHub Pages 仓库的设置里勾选 ‘Enforce HTTPS’，这样所有访问你站点的请求都会走 HTTPS。

下面我们添加三个数据

```
@ A 185.199.108.153
@ A 185.199.109.153
www CNAME username.github.io
```

![](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20201024131207.png)

其中 `185.199.108.153` 和 `185.199.109.153` 都是 GitHub 的地址。

上述设置的解释：

- 设置 A 记录的意思是，当我输入 `tding.top` 这个域名的时候，访问的是 `185.199.108.153` 这个地址；
- 设置 CNAME 的意思是，当我访问 `dta0502.github.io` 这个地址的时候，会跳转到 `tding.top`，之后的过程就和 A 记录相同了，即访问 `185.199.108.153`。

## 添加 CNAME 文件

在本地`static`目录下新建`CNAME`文件，内容填入绑定的域名`www.riotian.com`，同时也需要将域名解析至`RivTian.github.io.`

后依次在终端中执行：

1. `git add .`暂存全部文件
2. `git commit -m "绑定域名"`提交
3. `git push -u origin develop`推送到远端仓库

## Github Pages 对自定义域上 Https

然后我们在 Github Pages 项目中 Settings 选项卡 Github Pages 选项：在 Custom domain 添加你的自定义域名。

刷新页面 如果能勾选 Enforce HTTPS 即完成。

![image-20201024131311814](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20201024131314.png)

现在，我们就可以通过 https 访问自定义域了。

## 参考

- [Hexo 基础教程 (二)：个人域名绑定](https://indexmoon.com/articles/1318032967/)
- [github 怎么绑定自己的域名？](https://www.zhihu.com/question/31377141/answer/103056861)
- [为 Github 上的 Hexo 博客绑定个性域名](http://cps.ninja/2016/10/09/customize-your-blog-domain/)
- [自定义域故障排除](https://help.github.com/cn/articles/troubleshooting-custom-domains)