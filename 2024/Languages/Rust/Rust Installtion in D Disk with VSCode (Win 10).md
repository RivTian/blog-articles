
(只记录了必须要内容，日后完善！)

\1. rust的安装与环境变量：

要提前把下面两个环境变量配置好，这样是为了指定安装路径。否则会默认安装在 C 盘下。

```powershell
 CARGO_HOME: D:\Soft\Language\Rust\.cargo  
 RUSTUP_HOME: D:\Soft\Language\Rust\.rustup
```

然后，在这个：[Rust, Get-Start](https://www.rust-lang.org/zh-CN/learn/get-started) 界面上下载 rustup-init.exe。下载完成后直接点击执行，会出现一个**CMD**窗口：仔细阅读上面的内容，如果没有安装 **Microsoft 2019 builder tools**，就打开屏幕上的网址进行下载安装。可以从所给网址直接下载 Microsoft Visual Stdio 2019，或者在这个页面下拉，找到下图所示内容，只下载下图中红框标注的内容即可：

- 2019: [https://aka.ms/vs/16/release/vs_buildtools.exe](https://aka.ms/vs/16/release/vs_buildtools.exe)
    
- 2022: [https://aka.ms/vs/17/release/vs_buildtools.exe](https://aka.ms/vs/17/release/vs_buildtools.exe)
    

![](https://img2023.cnblogs.com/blog/1849509/202312/1849509-20231215090216639-40363553.png)

然后，在下面输入2，进行自定义安装：

![](https://img2023.cnblogs.com/blog/1849509/202312/1849509-20231215090239207-1312056744.png)

按自己的要求设置好之后就开始安装吧！

执行下面的命令看是否安装成功：

```sh
 cargo --version  
 rustc --version
```

执行如下命令安装工具链：可以选择其它版本(如 `nightly-i686-pc-windows-msvc` )

```sh
 rustup toolchain install nightly-x86_64-pc-windows-gnu　
```

安装源码：

```sh
 rustup component add rust-src --toolchain nightly
```

完成之后要设置如下环境变量：

```bash
RUST：D:\Soft\Language\Rust\.rustup\toolchains\nightly-i686-pc-windows-msvc
RUST_SRC_PATH：%RUST%\lib\rustlib\src\rust\src
RUSTBINPATH：%CARGO_PATH%\bin
 
# 下面两个是配置科大源要用到的：
RUSTUP_DIST_SERVER：https://mirrors.ustc.edu.cn/rust-static
RUSTUP_UPDATE_ROOT：https://mirrors.ustc.edu.cn/rust-static/rustup
 
# 并且在path里添加如下路径：
%CARGO_HOME%\bin　　
```

继续安装：

```bash
 cargo +nightly install racer    // 安装 racer  
    
 rustup component add rls-preview --toolchain nightly  
 rustup component add rust-analysis --toolchain nightly
```

---

\2. vscode配置信息:

1. 下载插件：在 VSCode 上搜索插件 rust， 然后把排名的前两个给装上，然后还可以把 **rustfmt** 和 **vscode-rust-syntax** 装上，作用分别是代码格式化和语法高亮。
    
2. File/Preferences/settings/下找到 `setings.json` 文件，向其中添加如下内容：

```json
"rust.mode": "rls",
"rust.cargoHomePath": "%CARGO_HOME%",
"rust.cargoPath":"%RUSTBINPATH%\\cargo.exe",
"rust.racerPath":"%RUSTBINPATH%\\racer.exe",
"rust.rls":"%RUSTBINPATH%\\rls.exe",
"rust.rustfmtPath":"%RUSTBINPATH%\\rustfmt.exe",
"rust.rustup":"%RUSTBINPATH%\\rustup.exe",
"rust.rustLangSrcPath": "%RUST_SRC_PATH%",
"rust.executeCargoCommandInTerminal": true,
```

