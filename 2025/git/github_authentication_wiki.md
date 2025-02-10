![](https://iphysresearch.github.io/blog/post/programing/2021-08-13-token-authentication-requirements-for-git-operations/featured_hu6c83029614b08710250e87a3d4b30811_99776_67d129ec1399f2437fc927705f661cd9.webp)

这两天我发现用 GitHub 的时候，push 不了代码了，不停地出如下的问题：

```sh
remote: Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.
remote: Please see https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/ for more information.
fatal: 无法访问 '<git repo url>'：The requested URL returned error: 403
```

其实官方早在去年年底的[官方博文](https://github.blog/security/application-security/token-authentication-requirements-for-git-operations/)就给出了上面链接的说明，同时最近的[官方博文](https://github.blog/changelog/2021-08-12-git-password-authentication-is-shutting-down/)中也有着详细的解释：

> As [previously announced](https://github.blog/security/application-security/token-authentication-requirements-for-git-operations/), starting on August 13, 2021, at 09:00 PST, we will no longer accept account passwords when authenticating Git operations on GitHub.com. Instead, token-based authentication (for example, personal access, OAuth, SSH Key, or GitHub App installation token) will be required for all authenticated Git operations.
> Please refer to this [blog post](https://github.blog/security/application-security/token-authentication-requirements-for-git-operations/#what-you-need-to-do-today) for instructions on what you need to do to continue using git operations securely.

简单来说，就是从 2021 年 8 月 13 号开始，在 GitHub.com 上任何授权 Git 的行为都不再支持密码验证了，请用使用基于 token 的授权方式来替代，比如说：personal access, OAuth, SSH Key, or GitHub App installation token。

（以下为正文）

--- 

下面就是基于我的 macOS 系统完成的配置并解决文体，仅供参考。

> FYI：如果你想自己学着根据官方材料配置的话，可以参考如下两个链接就足够了。
>
> * [Creating a personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
> 
> * [Updating credentials from the macOS Keychain](https://docs.github.com/en/get-started/getting-started-with-git/updating-credentials-from-the-macos-keychain)


总共两步：

第一步，先到 GitHub 个人设置里设置 token。

- 打开自己的 GitHub主页，点击自己的头像找到 Settings 并进入，在左边目录栏找到 Developer settings - Personal access tokens，点击 Generate new token，按照步骤申请即可，过程简单。Scopes（范围）那里建议全选。

![](https://vip1.loli.io/2021/08/16/UJAjyMbfY2lZ1wx.png)

![](https://vip1.loli.io/2021/08/16/1vXBb5PLGJyfCcU.png)

Token 申请成功后，一定记得要复制保存下来这个 token 字符串，因为这是你最后一次看到它的机会，忘记了的话 GitHub.com 是不会给你查找的，你只能重新申请新的。

![](https://vip2.loli.io/2021/08/16/i3gKBjRhQlr5fAd.png)

第二步，在 macOS 的钥匙串访问里修改 GitHub 的密码为刚刚获得的 token。

用 Spotlight 或者 Alfred 等找到钥匙串访问 (Keychain Access)，并打开。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_250210051142_image.png)

右上方搜索 **github.com**，然后种类类别里找到 **互联网密码(internet password)**，双击这个条目打开以后，在密码处填入第一步中复制保存下来的 token，最后存储更改即可。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_250210051503_image.png)