#Tools

最近在捣鼓 Nvim 的配置，然后偶然发现了Wakatime这个可以记录你Coding时间的工具，在这里介绍下~

## 使用 WakeTime 记录你的 Coding 数据

### 前言

[WakaTime](https://wakatime.com/) 是一款可以记录你的编码时间的工具，目前支持绝大部分主流的 IDE 以及 Chrome 浏览器。

### 安装Wakatime

1. [注册](https://wakatime.com/signup) WakaTime 账号；
2. 在[官网](https://wakatime.com/plugins)找到对应的 IDE 插件，按照步骤安装 WakaTime 插件；
3. 在[个人设置](https://wakatime.com/settings/account)页面复制 Secret API Key ，填入对应的 WakaTime 插件中；
4. 过一段时间后，你就可以在 WakaTime 网站上看到你的编码情况；

### 接入waka-box

[waka-box](https://github.com/matchai/waka-box)提供了你每周的代码状态，并且更新为类似于下面的内容：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg@main/uPic/WlXC6w.jpg)


下面是接入步骤：

1. 在Github创建一个Gist：[Github Gist](https://gist.github.com/)，标题与内容可随意，成功运行后会被修改，保存 `Gist-url` 上一串 `key`，如：`https://gist.github.com/matchai/6d5f84419863089a167387da62dd7081` 中 `6d5f84419863089a167387da62dd7081`；
2. 创建带有 `gist` 功能 `Github Token` 并保存下来 [Token](https://github.com/settings/tokens/new)
3. 登入 `WakaTime` 页面，没有账号的，请[自行创建](https://wakatime.com/signup)
4. 进入 `WakaTime` [配置页面](https://wakatime.com/settings/profile)，勾选 `Display coding activity publicly` 与 `Display languages, editors, operating systems publicly`
5. 查看 `WakaTime` 账号 [api-key](https://wakatime.com/settings/api-key)，并保存好

项目设置

1. `fork` 项目 - [github.com/matchai/wak…](https://github.com/matchai/waka-box)
2. 在 `fork` 项目下 `Settings > Secrets`，新增 `GH_TOKEN` 与 `WAKATIME_API_KEY`，Value 分别对应上面生成的 `Github Token` 与 `WakaTime api-key`
3. 打开 `.github/workflows/schedule.yml` 文件，修改 `GIST_ID`为自己的；