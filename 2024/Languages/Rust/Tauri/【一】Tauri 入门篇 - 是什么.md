#Rust  #Tauri

> 文章里包含大量的个人探索学习，无法保证其是否为最佳实践，我会尽量客观地将相关资料及学习过程记录下来。虽然 Tauri 经发布 v1.0 版本，但是到目前为止，国内资料仍少的可怜，我个人想基于 Tauri 开发一款工具集（各种小功能）。在这个过程中，我遇到了一些，或者说是一系列的问题，我就想把它们以专栏系列的形式记录下来（好记性不如烂笔头），希望可以帮助到更多和我一样对新技术充满热情的人。如果这些文章对你有所帮助，可以将文章转发给更多有需要的人。大家的支持也会给我更大的写作动力，感恩。点击“阅读原文”可以跳转到仓库（star 收藏不迷路）。

> Tauri$^{[1]}$ 使用 Web 前端构建更小、更快、更安全的桌面应用程序。

## Tauri 是什么？

Tauri 是一个工具包，可以帮助开发者为主要桌面平台制作应用程序（如 mac，windows，linux 等）。几乎支持现有的任何前端框架（如 react, vue, vite 等），其核心是使用 Rust 编写的。Rust 负责系统通信，应用构建，上层应用开发只需要专注于 webview 中的网页编写（可以使用自己喜欢的任何前端语言）。

### 安全第一

- Tauri 允许您选择要发布的 API 端点，是否希望在你的应用程序中内置一个本地主机（localhost）服务器，甚至可以在运行时随机化功能句柄
    
- 默认情况下 Tauri 只提供二进制文件，而非 Asar$^{[2]}$ 文件

#### Asar

> 提高在 Windows 等平台上读取文件的性能，而不是通过复制其所有源文件来发布您的应用程序。

Asar 用于将 Electron 应用程序文件连接到一个大文件。为了缓解 Windows 上长路径名问题，稍微加快 require 并隐藏源代码不被粗略检查（Cursory Inspection），可以选择将应用程序打包成 asar 归档文件，而对源代码几乎没有改动。

### 多语言支持

目前，Tauri 使用 Rust 作为后端 —— 但在不远的将来，其他后端如 Go、Nim、Python、Csharp 等也将成为可能。目前官方维护的是 Rust 与 webview 组织的绑定，因为 API 可以用任何带有 C 语言互操作的语言来实现，所以选择自己喜欢的语言作为后端只是一个 PR 的问题。

### 开放源代码

如果没有一个社区，这一切都没有任何意义。今天的软件社区是令人惊奇的地方，人们在那里互相帮助，创造出令人敬畏的东西。

### Tauri vs Electron

Electron 的一个主要好处是可以针对单个浏览器和运行时版本进行开发，而不必处理所有小而耗时的兼容性问题。Tauri 相比于 Electron 放弃了一些兼容性，换来了内存以及应用体积的更小化，在安全方面也做了一些事情。从 Tauri 的架构图来看它实现的是 Webview2 和系统之间的一个胶水层，目前后端是 Rust 实现，将来也可以替换成别的语言。前端现在就可以使用自己喜欢的框架来编写（vue，react 等）。而且从 README platforms$^{[3]}$ 可以看到，Tauri 始于桌面，但并不止步于此。后期可以跨平台到 iOS/iPadOS，Android 等。称其为下一代跨端框架似乎也不过分。

![640](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/640.png)

## Tauri 架构

Tauri 是一个多语言通用工具包，其超强的组合性允许工程师制作各种应用程序。它使用 Rust 工具和 Webview 中呈现的 HTML 的组合来构建桌面应用程序。后端使用 Rust 和系统进行交互（如系统信息，系统通知，系统文件等），包装成 Tauri 插件后会暴露出 JS API 供前端使用，通过 Webview 进行消息传递来控制系统。所以不支持的功能，开发人员可以通过编写 Rust 来扩展默认 API。

Tauri 应用程序可以有自定义菜单$^{[4]}$和托盘类型的界面$^{[5]}$。它们可以按预期由用户的操作系统进行更新和管理[6]。它们非常小[7]，因为它们使用操作系统的 webview。它们不提供运行时，因为最终的二进制文件是从 Rust 编译的。这使得 Tauri 应用程序的逆向不是一项简单的任务[8]。

![](https://user-images.githubusercontent.com/16164244/173187920-f7aa807a-4105-49bc-bd49-043cb3db109f.png)

- **核心（Core）**
    - [tauri](https://github.com/tauri-apps/tauri/tree/dev/core/tauri) - 主条板箱（major crate）把所有东西（运行时、宏、实用程序和 API ）都集中在一起构成最终产品。在编译时读取 [tauri.conf.json](https://tauri.studio/v1/api/config) 文件，以引入功能并进行应用程序的实际配置（包括项目中的 Cargo.toml 文件）。在运行时处理脚本注入（用于 polyfills/原型修改），托管系统交互的 API 及管理更新。
    - [tauri-runtime](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-runtime) - Tauri 本身和较低级别的 webview 库之间的粘合层。
    - [tauri-macros](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-macros) - 通过利用 `tauri-codegen` crate 为上下文、处理程序和命令创建宏。
    - [tauri-utils](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-utils) - 在许多地方重用的通用代码，并提供有用的实用程序，例如解析配置文件、检测平台三元组（CPU 系列/型号的名称、供应商和操作系统名称）、注入 CSP 和管理资产。
    - [tauri-build](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-build) - 在构建时应用宏，为 `cargo` 装配所需的一些特殊功能。
    - [tauri-codegen](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-codegen) - 嵌入、散列和压缩资产，包括应用程序和系统托盘的图标。在编译时解析 `tauri.conf.json` 并生成配置结构。
    - [tauri-runtime-wry](https://github.com/tauri-apps/tauri/tree/dev/core/tauri-runtime-wry) - 专门为 WRY 开辟了直接的系统级交互，如打印、显示器检测和其他与窗口相关的任务。
- **上游条板箱（Upstream Crates）** - Tauri-Apps 组织维护 Tauri 的两个“上游”板条箱，即用于创建和管理应用程序窗口的 TAO，以及用于与窗口内的 Webview 交互的 WRY。
    - [TAO](https://github.com/tauri-apps/tao) - Rust 中的跨平台应用程序窗口创建库，支持 Windows、macOS、Linux、iOS 和 Android 等所有主要平台。它是用 Rust 编写的，是一个 [winit](https://github.com/rust-windowing/winit) 的分支，根据需要进行了扩展，例如菜单栏和系统托盘。
    - [WRY](https://github.com/tauri-apps/wry) - WRY 是 Rust 中的跨平台 WebView 渲染库，支持 Windows、macOS 和 Linux 等所有主要桌面平台。 Tauri 使用 WRY 作为抽象层，负责确定使用哪个 webview（以及如何进行交互）。


### Tauri 不是什么?

- Tauri 不是一个轻量级的内核包装器。相反，它直接使用 `WRY` 和 `TAO` 来完成对操作系统进行系统调用的繁重工作。
- Tauri 不是虚拟机或虚拟化环境。相反，它是一个允许制作 Webview OS 应用程序的应用程序工具包。