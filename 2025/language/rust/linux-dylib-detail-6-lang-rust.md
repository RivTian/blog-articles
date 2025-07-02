## rust 工具链

Rust 使用 rustup 命令来管理 rust 编译工具链，在支持的平台可通过如下命令一键安装 Rust 工具链（详见：[官网](https://www.rust-lang.org/learn/get-started)）：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

如上命令后的默认行为如下（x86_64 Linux 系统为例，rust 1.81）：

* rust 工具链以及 rustup 元数据将保存到 `~/.rustup` 目录下。
    * 默认将安装最新的 stable 的 default profile 的 工具链（`x86_64-unknown-linux-gnu`）。
    * 保存到 `~/.rustup/toolchains` 目录下。
* cargo home 目录配置到 `~/.cargo`。
* rust 工具链的 bin 将安装到 `~/.cargo/bin` 目录下，包含 `rustup`、`cargo`、`rustc` 等，值得一提的是：这些 bin 文件本质上时同一个文件，通过硬链接设置为不同的名字，这样做的好处是可以最大程度的共享代码，减少工具链体积，更容易管理。
* 修改当前用户 shell 的 profile 文件，如 `~/.bashrc`，添加 `. "$HOME/.cargo/env"`，将 `~/.cargo/bin` 添加到 `PATH` 环境变量中。

执行如下脚本 `04-lang/02-rust/01-rust-toolchain.sh`：

```bash
#!/usr/bin/env bash


echo '=== 观察 rustc 情况'
echo '--- 查看 rustc 版本'
rustc -V
echo '--- 查看 rustc 的动态库依赖'
ldd $(which rustc)
echo '--- 查看 rustc 的 glibc 依赖情况'
readelf --version-info $(which rustc)
```

输入出下：

```
=== 观察 rustc 情况
--- 查看 rustc 版本
rustc 1.81.0 (eeb90cda1 2024-09-04)
--- 查看 rustc 的动态库依赖
        linux-vdso.so.1 (0x00007ffc6d9ce000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f9c0339e000)
        librt.so.1 => /lib/x86_64-linux-gnu/librt.so.1 (0x00007f9c03399000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f9c03394000)
        libm.so.6 => /lib/x86_64-linux-gnu/libm.so.6 (0x00007f9c032b5000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f9c032b0000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f9c0241f000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f9c033c6000)
--- 查看 rustc 的 glibc 依赖情况

Version symbols section '.gnu.version' contains 330 entries:
 Addr: 0x0000000000003750  Offset: 0x00003750  Link: 4 (.dynsym)
  000:   0 (*本地*)       2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  004:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   4 (GLIBC_2.9)  
  008:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  00c:   5 (GCC_3.0)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   5 (GCC_3.0)    
  010:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  014:   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   5 (GCC_3.0)       0 (*本地*)    
  018:   6 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  01c:   3 (GLIBC_2.2.5)   0 (*本地*)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  020:   7 (GLIBC_2.3.2)   0 (*本地*)       3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  024:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   8 (GLIBC_2.7)     2 (GLIBC_2.2.5)
  028:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  02c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  030:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   7 (GLIBC_2.3.2)
  034:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  038:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   9 (GLIBC_2.12) 
  03c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  040:   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       2 (GLIBC_2.2.5)
  044:   5 (GCC_3.0)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  048:   8 (GLIBC_2.7)     2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  04c:   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       3 (GLIBC_2.2.5)
  050:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       2 (GLIBC_2.2.5)
  054:   2 (GLIBC_2.2.5)   6 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   5 (GCC_3.0)    
  058:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   a (GLIBC_2.17) 
  05c:   2 (GLIBC_2.2.5)   0 (*本地*)       b (GLIBC_2.4)     2 (GLIBC_2.2.5)
  060:   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   5 (GCC_3.0)    
  064:   2 (GLIBC_2.2.5)   c (GLIBC_2.3)     2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  068:   c (GLIBC_2.3)     3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  06c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   d (GCC_3.3)    
  070:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   5 (GCC_3.0)    
  074:   d (GCC_3.3)       3 (GLIBC_2.2.5)   4 (GLIBC_2.9)     3 (GLIBC_2.2.5)
  078:   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  07c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   7 (GLIBC_2.3.2)
  080:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  084:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       2 (GLIBC_2.2.5)
  088:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  08c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  090:   e (GLIBC_2.3.4)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  094:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  098:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  09c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0a0:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       f (GCC_4.2.0)  
  0a4:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0a8:   2 (GLIBC_2.2.5)  10 (GLIBC_2.15)    2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0ac:   3 (GLIBC_2.2.5)   d (GCC_3.3)       2 (GLIBC_2.2.5)   7 (GLIBC_2.3.2)
  0b0:   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)    
  0b4:   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0b8:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   e (GLIBC_2.3.4)   2 (GLIBC_2.2.5)
  0bc:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  0c0:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  0c4:   3 (GLIBC_2.2.5)  11 (GLIBC_2.2.5)   5 (GCC_3.0)       2 (GLIBC_2.2.5)
  0c8:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0cc:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   b (GLIBC_2.4)  
  0d0:   2 (GLIBC_2.2.5)   b (GLIBC_2.4)     3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0d4:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0d8:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   0 (*本地*)       3 (GLIBC_2.2.5)
  0dc:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  0e0:   2 (GLIBC_2.2.5)   5 (GCC_3.0)       2 (GLIBC_2.2.5)   b (GLIBC_2.4)  
  0e4:  12 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0e8:   3 (GLIBC_2.2.5)  13 (GLIBC_2.14)    3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0ec:   3 (GLIBC_2.2.5)   5 (GCC_3.0)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0f0:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  0f4:   2 (GLIBC_2.2.5)   b (GLIBC_2.4)     1 (*全局*)      1 (*全局*)   
  0f8:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  0fc:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  100:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  104:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  108:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  10c:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  110:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  114:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  118:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  11c:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  120:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  124:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  128:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  12c:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  130:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  134:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  138:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  13c:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  140:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  144:   1 (*全局*)      1 (*全局*)      1 (*全局*)      1 (*全局*)   
  148:   1 (*全局*)      1 (*全局*)   

Version needs section '.gnu.version_r' contains 6 entries:
 Addr: 0x00000000000039e8  Offset: 0x000039e8  Link: 5 (.dynstr)
  000000: Version: 1  文件：librt.so.1  计数：1
  0x0010:   Name: GLIBC_2.2.5  标志：无  版本：18
  0x0020: Version: 1  文件：libdl.so.2  计数：1
  0x0030:   Name: GLIBC_2.2.5  标志：无  版本：17
  0x0040: Version: 1  文件：libm.so.6  计数：1
  0x0050:   Name: GLIBC_2.2.5  标志：无  版本：6
  0x0060: Version: 1  文件：libgcc_s.so.1  计数：3
  0x0070:   Name: GCC_4.2.0  标志：无  版本：15
  0x0080:   Name: GCC_3.3  标志：无  版本：13
  0x0090:   Name: GCC_3.0  标志：无  版本：5
  0x00a0: Version: 1  文件：libpthread.so.0  计数：2
  0x00b0:   Name: GLIBC_2.12  标志：无  版本：9
  0x00c0:   Name: GLIBC_2.2.5  标志：无  版本：3
  0x00d0: Version: 1  文件：libc.so.6  计数：10
  0x00e0:   Name: GLIBC_2.14  标志：无  版本：19
  0x00f0:   Name: GLIBC_2.15  标志：无  版本：16
  0x0100:   Name: GLIBC_2.3.4  标志：无  版本：14
  0x0110:   Name: GLIBC_2.3  标志：无  版本：12
  0x0120:   Name: GLIBC_2.4  标志：无  版本：11
  0x0130:   Name: GLIBC_2.17  标志：无  版本：10
  0x0140:   Name: GLIBC_2.7  标志：无  版本：8
  0x0150:   Name: GLIBC_2.3.2  标志：无  版本：7
  0x0160:   Name: GLIBC_2.9  标志：无  版本：4
  0x0170:   Name: GLIBC_2.2.5  标志：无  版本：2
```

可以看出 rust 的 `stable-x86_64-unknown-linux-gnu` (`1.81.0`) 工具链：

* 依赖 2.17 及以上版本的 glibc。
* 依赖 4.2.0 及以上版本的 gcc。

这些信息在官方文档 [rustc - 支持的平台](https://doc.rust-lang.org/nightly/rustc/platform-support.html) 中 `Tier xxx with Host Tools` 部分。

## rust target

上一小节介绍的是 rust 工具链自身的动态链接库，实际上 rust 工具链是支持交叉编译的（例如：在 Linux 平台可以编译出 Windows 平台的产物）。在官方文档 [rustc - 支持的平台](https://doc.rust-lang.org/nightly/rustc/platform-support.html) 中 `Tier xxx` 部分有关于支持的平台的详细介绍。

本文主要探索 rust target 为 `x86_64-unknown-linux-gnu` 以及 `x86_64-unknown-linux-musl` 这两种情况。

`x86_64-unknown-linux-gnu` 已默认安装，`x86_64-unknown-linux-musl` 通过如下命令安装：

```bash
rustup target add x86_64-unknown-linux-musl
```

从如上命令的输出来看， `rustup target add` 命令主要安装的是 `rust-std` 库，安装到了 `~/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/` 路径下。

通过另外通过 `rustc --print target-list` 可以查看所有可用的内置 target （[rustc - 内建的目标](https://doc.rust-lang.org/rustc/targets/built-in.html)）。

这里最重要的是链接器的配置，可以通过如下命令查看（[rustc - 自定义目标](https://doc.rust-lang.org/rustc/targets/custom.html)）。

```bash
# rustup toolchain install nightly
rustc +nightly -Z unstable-options --target=x86_64-unknown-linux-gnu --print target-spec-json | grep linker-flavor
#   "linker-flavor": "gnu-lld-cc",
rustc +nightly -Z unstable-options --target=x86_64-unknown-linux-musl --print target-spec-json | grep linker-flavor
#   "linker-flavor": "gnu-cc",
```

## 示例

### 示例项目

使用 cargo new 命令创建一个示例项目：

`04-lang/02-rust/a01-hello/Cargo.toml`

```toml
[package]
name = "a01-hello"
version = "0.1.0"
edition = "2021"

[dependencies]
```

`04-lang/02-rust/a01-hello/Cargo.lock`

```toml
# This file is automatically @generated by Cargo.
# It is not intended for manual editing.
version = 3

[[package]]
name = "a01-hello"
version = "0.1.0"
```

`04-lang/02-rust/a01-hello/src/main.rs`

```rs
fn main() {
    println!("Hello, world!");
}
```

### 使用默认参数构建

根据 cargo 官方文档（[The Cargo Book - Configuration](https://doc.rust-lang.org/cargo/reference/config.html)）可知，默认情况下 cargo build 的：

* 默认 target 是当前机器的架构，即 rustup 安装的工具链默认的架构，在本示例中，就是 `x86_64-unknown-linux-gnu`。
* 默认 profile 是 dev。
* 默认的链接器为 gcc

验证脚本 `04-lang/02-rust/02-rust-build-hello-default.sh`

```bash
#!/usr/bin/env bash

cd $(dirname $(readlink -f $0))
cd a01-hello
rm -rf target


echo '=== 构建 hello'
echo '--- 构建 dev 产物'
cargo build
# cargo build --profile dev
echo '执行产物'
target/debug/a01-hello
echo 'lld 查看产物'
ldd target/debug/a01-hello
echo 'readelf 查看产物版本'
readelf --version-info target/debug/a01-hello
echo 'readelf 查看产物的 RUNPATH'
readelf -d target/debug/a01-hello | grep RUNPATH

echo '--- 构建 release 产物'
cargo build --release
echo '执行产物'
target/release/a01-hello
echo 'lld 查看产物'
ldd target/release/a01-hello
echo 'readelf 查看产物版本'
readelf --version-info target/release/a01-hello
echo 'readelf 查看产物的 RUNPATH'
readelf -d target/release/a01-hello | grep RUNPATH
echo
```

输出如下

```
=== 构建 hello
--- 构建 dev 产物
   Compiling a01-hello v0.1.0 (/home/rectcircle/omv/00-Important/Workspace/rectcircle/linux-dylib-demo/
04-lang/02-rust/a01-hello)
   Finished `dev` profile [unoptimized + debuginfo] target(s) in 7.22s
执行产物
Hello, world!
lld 查看产物
        linux-vdso.so.1 (0x00007ffe52f8e000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f161a202000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f161a021000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f161a280000)
readelf 查看产物版本

Version symbols section '.gnu.version' contains 67 entries:
 Addr: 0x0000000000000e56  Offset: 0x00000e56  Link: 6 (.dynsym)
  000:   0 (*本地*)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  004:   3 (GLIBC_2.34)    2 (GLIBC_2.2.5)   4 (GCC_3.3)       2 (GLIBC_2.2.5)
  008:   1 (*全局*)      2 (GLIBC_2.2.5)   5 (GLIBC_2.32)    2 (GLIBC_2.2.5)
  00c:   6 (GLIBC_2.18)    7 (GLIBC_2.3.4)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  010:   8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)       8 (GCC_3.0)    
  014:   2 (GLIBC_2.2.5)   9 (GLIBC_2.33)    3 (GLIBC_2.34)    2 (GLIBC_2.2.5)
  018:   2 (GLIBC_2.2.5)   a (GCC_4.2.0)     2 (GLIBC_2.2.5)   8 (GCC_3.0)    
  01c:   3 (GLIBC_2.34)    2 (GLIBC_2.2.5)   b (GLIBC_2.3)     2 (GLIBC_2.2.5)
  020:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.34)    1 (*全局*)   
  024:   c (GLIBC_2.3)     d (GLIBC_2.14)    8 (GCC_3.0)       3 (GLIBC_2.34) 
  028:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  02c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   e (GLIBC_2.16)    f (GLIBC_2.28) 
  030:   8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)       2 (GLIBC_2.2.5)
  034:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  038:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   9 (GLIBC_2.33)    2 (GLIBC_2.2.5)
  03c:   1 (*全局*)      8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)    
  040:   3 (GLIBC_2.34)    8 (GCC_3.0)       2 (GLIBC_2.2.5)

Version needs section '.gnu.version_r' contains 3 entries:
 Addr: 0x0000000000000ee0  Offset: 0x00000ee0  Link: 7 (.dynstr)
  000000: Version: 1  文件：ld-linux-x86-64.so.2  计数：1
  0x0010:   Name: GLIBC_2.3  标志：无  版本：11
  0x0020: Version: 1  文件：libgcc_s.so.1  计数：3
  0x0030:   Name: GCC_4.2.0  标志：无  版本：10
  0x0040:   Name: GCC_3.0  标志：无  版本：8
  0x0050:   Name: GCC_3.3  标志：无  版本：4
  0x0060: Version: 1  文件：libc.so.6  计数：10
  0x0070:   Name: GLIBC_2.28  标志：无  版本：15
  0x0080:   Name: GLIBC_2.16  标志：无  版本：14
  0x0090:   Name: GLIBC_2.14  标志：无  版本：13
  0x00a0:   Name: GLIBC_2.3  标志：无  版本：12
  0x00b0:   Name: GLIBC_2.33  标志：无  版本：9
  0x00c0:   Name: GLIBC_2.3.4  标志：无  版本：7
  0x00d0:   Name: GLIBC_2.18  标志：无  版本：6
  0x00e0:   Name: GLIBC_2.32  标志：无  版本：5
  0x00f0:   Name: GLIBC_2.34  标志：无  版本：3
  0x0100:   Name: GLIBC_2.2.5  标志：无  版本：2
readelf 查看产物的 RUNPATH
--- 构建 release 产物
   Compiling a01-hello v0.1.0 (/home/rectcircle/omv/00-Important/Workspace/rectcircle/linux-dylib-demo/
04-lang/02-rust/a01-hello)   
   Finished `release` profile [optimized] target(s) in 5.68s
执行产物
Hello, world!
lld 查看产物
        linux-vdso.so.1 (0x00007fffd8faa000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007fc0bee97000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc0becb6000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fc0bef15000)
readelf 查看产物版本

Version symbols section '.gnu.version' contains 67 entries:
 Addr: 0x0000000000000e56  Offset: 0x00000e56  Link: 6 (.dynsym)
  000:   0 (*本地*)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  004:   3 (GLIBC_2.34)    2 (GLIBC_2.2.5)   4 (GCC_3.3)       2 (GLIBC_2.2.5)
  008:   1 (*全局*)      2 (GLIBC_2.2.5)   5 (GLIBC_2.32)    2 (GLIBC_2.2.5)
  00c:   6 (GLIBC_2.18)    7 (GLIBC_2.3.4)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  010:   8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)       8 (GCC_3.0)    
  014:   2 (GLIBC_2.2.5)   9 (GLIBC_2.33)    3 (GLIBC_2.34)    2 (GLIBC_2.2.5)
  018:   2 (GLIBC_2.2.5)   a (GCC_4.2.0)     2 (GLIBC_2.2.5)   8 (GCC_3.0)    
  01c:   3 (GLIBC_2.34)    2 (GLIBC_2.2.5)   b (GLIBC_2.3)     2 (GLIBC_2.2.5)
  020:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.34)    1 (*全局*)   
  024:   c (GLIBC_2.3)     d (GLIBC_2.14)    8 (GCC_3.0)       3 (GLIBC_2.34) 
  028:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  02c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   e (GLIBC_2.16)    f (GLIBC_2.28) 
  030:   8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)       2 (GLIBC_2.2.5)
  034:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  038:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   9 (GLIBC_2.33)    2 (GLIBC_2.2.5)
  03c:   1 (*全局*)      8 (GCC_3.0)       2 (GLIBC_2.2.5)   8 (GCC_3.0)    
  040:   3 (GLIBC_2.34)    8 (GCC_3.0)       2 (GLIBC_2.2.5)

Version needs section '.gnu.version_r' contains 3 entries:
 Addr: 0x0000000000000ee0  Offset: 0x00000ee0  Link: 7 (.dynstr)
  000000: Version: 1  文件：ld-linux-x86-64.so.2  计数：1
  0x0010:   Name: GLIBC_2.3  标志：无  版本：11
  0x0020: Version: 1  文件：libgcc_s.so.1  计数：3
  0x0030:   Name: GCC_4.2.0  标志：无  版本：10
  0x0040:   Name: GCC_3.0  标志：无  版本：8
  0x0050:   Name: GCC_3.3  标志：无  版本：4
  0x0060: Version: 1  文件：libc.so.6  计数：10
  0x0070:   Name: GLIBC_2.28  标志：无  版本：15
  0x0080:   Name: GLIBC_2.16  标志：无  版本：14
  0x0090:   Name: GLIBC_2.14  标志：无  版本：13
  0x00a0:   Name: GLIBC_2.3  标志：无  版本：12
  0x00b0:   Name: GLIBC_2.33  标志：无  版本：9
  0x00c0:   Name: GLIBC_2.3.4  标志：无  版本：7
  0x00d0:   Name: GLIBC_2.18  标志：无  版本：6
  0x00e0:   Name: GLIBC_2.32  标志：无  版本：5
  0x00f0:   Name: GLIBC_2.34  标志：无  版本：3
  0x0100:   Name: GLIBC_2.2.5  标志：无  版本：2
readelf 查看产物的 RUNPATH
```

说明： 该可执行文件只能在 glibc 版本大于等于 2.34 的 Linux 系统中运行。

### 使用 musl 构建

若使用 musl 构建，需要做如下事情：

* 使用 rustup 安装 musl 的 `rust-std` 组件，命令为： `rustup target add x86_64-unknown-linux-musl`。
* 下载 musl 交叉编译库，这里可以前往 https://musl.cc 下载预编译好的包，命令如下：

    ```bash
    wget https://musl.cc/x86_64-linux-musl-cross.tgz -O x86_64-linux-musl-cross.tgz
    mkdir -p ~/.local/share/musl
    tar -xzf x86_64-linux-musl-cross.tgz -C ~/.local/share/musl
    ~/.local/share/musl/x86_64-linux-musl-cross/bin/x86_64-linux-musl-ld
    rm -rf x86_64-linux-musl-cross.tgz
    ```

验证脚本 `04-lang/02-rust/03-rust-build-hello-musl.sh`

```bash
#!/usr/bin/env bash

cd $(dirname $(readlink -f $0))
cd a01-hello
rm -rf target


echo '=== 构建 hello target x86_64-unknown-linux-musl'
# CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER=$HOME/.local/share/musl/x86_64-linux-musl-cross/bin/x86_64-linux-musl-ld cargo build --target x86_64-unknown-linux-musl
cargo build --target x86_64-unknown-linux-musl --config "target.x86_64-unknown-linux-musl.linker='$HOME/.local/share/musl/x86_64-linux-musl-cross/bin/x86_64-linux-musl-ld'"
echo '--- 执行产物'
target/x86_64-unknown-linux-musl/debug/a01-hello
echo '--- ldd 查看产物'
ldd target/x86_64-unknown-linux-musl/debug/a01-hello
echo '--- readelf 查看产物版本'
readelf --version-info target/x86_64-unknown-linux-musl/debug/a01-hello
```

输出如下：

```
=== 构建 hello target x86_64-unknown-linux-musl
   Compiling a01-hello v0.1.0 (/home/rectcircle/omv/00-Important/Workspace/rectcircle/linux-dylib-demo/
04-lang/02-rust/a01-hello)                                                                                 Finished `dev` profile [unoptimized + debuginfo] target(s) in 5.82s
--- 执行产物
Hello, world!
--- ldd 查看产物
        statically linked
--- readelf 查看产物版本

No version information found in this file.
```

可以看出使用 musl 构建可以实现 rust 的静态编译。

### 使用 glibc 2.31 构建

上文 《使用默认参数构建》 中，使用默认参数构建，会使用系统默认的 glibc 版本（验证系统为 Debian 12 glibc 版本为 2.36）。

```bash
# 实测 gcc12 无法编译成功，需在 gcc10 环境中编译
# 这里使用 nix 安装 gcc10
nix-env -iA nixpkgs.gcc10
export GLIBC_VERSION=2.31
rm -rf /tmp/glibc-work && mkdir -p /tmp/glibc-work
cd /tmp/glibc-work
wget https://ftp.gnu.org/gnu/glibc/glibc-$GLIBC_VERSION.tar.gz -O glibc-$GLIBC_VERSION.tar.gz
tar -xzf glibc-$GLIBC_VERSION.tar.gz
rm -rf glibc-$GLIBC_VERSION.tar.gz
mkdir build
cd build
mkdir -p $HOME/.local/share/glibc/glibc-$GLIBC_VERSION
../glibc-$GLIBC_VERSION/configure \
  --prefix=$HOME/.local/share/glibc/glibc-$GLIBC_VERSION \
  --host=x86_64-linux-gnu \
  --build=x86_64-linux-gnu \
  --disable-werror \
  CC="gcc -m64" \
  CXX="g++ -m64" \
  CFLAGS="-O2" \
  CXXFLAGS="-O2"
make
make install
rm -rf /tmp/glibc-work
```

也可以前往 [toolchains.bootlin.com](https://toolchains.bootlin.com/) 下载预编译版本（x86-64 只有 2.34 之后的版本）（未验证）。

安装 patchelf 工具，用于修改动态库的 RUNPATH。

```bash
nix-env -iA nixpkgs.patchelf
```

验证脚本 `04-lang/02-rust/04-rust-build-hello-glibc2.31.sh`

```bash
#!/usr/bin/env bash

cd $(dirname $(readlink -f $0))
cd a01-hello
rm -rf target


echo '=== 构建 hello use glibc 2.31'

# cargo build --target x86_64-unknown-linux-gnu --config "target.x86_64-unknown-linux-gnu.rustflags='-L native=$HOME/.local/share/glibc/glibc-2.31/lib'"
cargo build --config "build.rustflags='-L native=$HOME/.local/share/glibc/glibc-2.31/lib'"
echo '--- 执行产物'
target/debug/a01-hello
echo '--- ldd 查看产物'
ldd target/debug/a01-hello
echo '--- readelf 查看产物的 RUNPATH'
readelf -d target/debug/a01-hello | grep RUNPATH
echo '--- readelf 查看产物版本'
readelf --version-info target/debug/a01-hello
echo '--- patchelf 移除 RUNPATH 设置并设置动态链接器'
patchelf --remove-rpath target/debug/a01-hello
patchelf --set-interpreter /lib64/ld-linux-x86-64.so.2 target/debug/a01-hello
echo '--- ldd 查看产物'
ldd target/debug/a01-hello
echo '--- 执行产物'
target/debug/a01-hello
```

输出如下：

```
=== 构建 hello use glibc 2.31
   Compiling a01-hello v0.1.0 (/home/rectcircle/omv/00-Important/Workspace/rectcircle/linux-dylib-demo/04-lang/02-rust/a01-hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 5.72s
--- 执行产物
target/debug/a01-hello: /home/rectcircle/.local/share/glibc/glibc-2.31/lib/libc.so.6: version `GLIBC_2.34' not found (required by /nix/store/6h8sqf7pgrrkgp2rwsh51w915dhxs8z2-gcc-10.5.0-lib/lib/libgcc_s.so.1)
--- ldd 查看产物
target/debug/a01-hello: /home/rectcircle/.local/share/glibc/glibc-2.31/lib/libc.so.6: version `GLIBC_2.34' not found (required by /nix/store/6h8sqf7pgrrkgp2rwsh51w915dhxs8z2-gcc-10.5.0-lib/lib/libgcc_s.so.1)
        linux-vdso.so.1 (0x00007ffed1f50000)
        libgcc_s.so.1 => /nix/store/6h8sqf7pgrrkgp2rwsh51w915dhxs8z2-gcc-10.5.0-lib/lib/libgcc_s.so.1 (0x00007f6e6f072000)
        libpthread.so.0 => /home/rectcircle/.local/share/glibc/glibc-2.31/lib/libpthread.so.0 (0x00007f6e6f051000)
        libdl.so.2 => /home/rectcircle/.local/share/glibc/glibc-2.31/lib/libdl.so.2 (0x00007f6e6f04c000)
        libc.so.6 => /home/rectcircle/.local/share/glibc/glibc-2.31/lib/libc.so.6 (0x00007f6e6ee8f000)
        /nix/store/3dyw8dzj9ab4m8hv5dpyx7zii8d0w6fi-glibc-2.39-52/lib/ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007f6e6f0e5000)
--- readelf 查看产物的 RUNPATH
 0x000000000000001d (RUNPATH)            Library runpath: [/home/rectcircle/.local/share/glibc/glibc-2.31/lib:/nix/store/3dyw8dzj9ab4m8hv5dpyx7zii8d0w6fi-glibc-2.39-52/lib:/nix/store/6h8sqf7pgrrkgp2rwsh51w915dhxs8z2-gcc-10.5.0-lib/lib]
--- readelf 查看产物版本

Version symbols section '.gnu.version' contains 67 entries:
 Addr: 0x00000000000010c4  Offset: 0x000010c4  Link: 6 (.dynsym)
  000:   0 (*local*)       2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  004:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   4 (GCC_3.3)       3 (GLIBC_2.2.5)
  008:   3 (GLIBC_2.2.5)   1 (*global*)      2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  00c:   5 (GLIBC_2.18)    6 (GLIBC_2.3.4)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  010:   7 (GCC_3.0)       3 (GLIBC_2.2.5)   7 (GCC_3.0)       7 (GCC_3.0)    
  014:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  018:   2 (GLIBC_2.2.5)   8 (GCC_4.2.0)     3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  01c:   7 (GCC_3.0)       3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   9 (GLIBC_2.3)  
  020:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  024:   1 (*global*)      a (GLIBC_2.3)     b (GLIBC_2.14)    7 (GCC_3.0)    
  028:   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  02c:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   c (GLIBC_2.16)    d (GLIBC_2.28) 
  030:   3 (GLIBC_2.2.5)   7 (GCC_3.0)       2 (GLIBC_2.2.5)   7 (GCC_3.0)    
  034:   3 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)
  038:   2 (GLIBC_2.2.5)   2 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)   3 (GLIBC_2.2.5)
  03c:   2 (GLIBC_2.2.5)   1 (*global*)      7 (GCC_3.0)       2 (GLIBC_2.2.5)
  040:   7 (GCC_3.0)       7 (GCC_3.0)       2 (GLIBC_2.2.5)

Version needs section '.gnu.version_r' contains 4 entries:
 Addr: 0x0000000000001150  Offset: 0x00001150  Link: 7 (.dynstr)
  000000: Version: 1  File: ld-linux-x86-64.so.2  Cnt: 1
  0x0010:   Name: GLIBC_2.3  Flags: none  Version: 9
  0x0020: Version: 1  File: libgcc_s.so.1  Cnt: 3
  0x0030:   Name: GCC_4.2.0  Flags: none  Version: 8
  0x0040:   Name: GCC_3.0  Flags: none  Version: 7
  0x0050:   Name: GCC_3.3  Flags: none  Version: 4
  0x0060: Version: 1  File: libpthread.so.0  Cnt: 1
  0x0070:   Name: GLIBC_2.2.5  Flags: none  Version: 3
  0x0080: Version: 1  File: libc.so.6  Cnt: 7
  0x0090:   Name: GLIBC_2.28  Flags: none  Version: 13
  0x00a0:   Name: GLIBC_2.16  Flags: none  Version: 12
  0x00b0:   Name: GLIBC_2.14  Flags: none  Version: 11
  0x00c0:   Name: GLIBC_2.3  Flags: none  Version: 10
  0x00d0:   Name: GLIBC_2.3.4  Flags: none  Version: 6
  0x00e0:   Name: GLIBC_2.18  Flags: none  Version: 5
  0x00f0:   Name: GLIBC_2.2.5  Flags: none  Version: 2
--- patchelf 移除 RUNPATH 设置并设置动态链接器
--- ldd 查看产物
        linux-vdso.so.1 (0x00007ffc401f5000)
        libgcc_s.so.1 => /lib/x86_64-linux-gnu/libgcc_s.so.1 (0x00007f0c9d00d000)
        libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f0c9d008000)
        libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f0c9d003000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f0c9ce22000)
        /lib64/ld-linux-x86-64.so.2 (0x00007f0c9d3c2000)
--- 执行产物
Hello, world!
```

分析说明：

* 这里使用 nix 安装了 gcc10，目前 nix 的 gcc10 是使用 glibc 2.39 编译的，因此 `libgcc_s.so.1` 最小 glibc 依赖是 glibc 2.34。
* 然后使用 nix 的 gcc10 编译了 glibc 2.31。
* 然后使用 cargo 编译 rust 项目，此时配置如下：
    * 使用 PATH 环境变量中 nix 的 gcc10 作为链接器。
    * 通过 `"build.rustflags='-L native=$HOME/.local/share/glibc/glibc-2.31/lib'"` 参数指定了 glibc 。
* 因为使用了 nix 的 gcc10 作为链接器，nix 的 gcc 会在编译产物中配置 rpath，动态库都能正确找到。
* 因此 rust 项目的编译产物中动态库依赖情况是：
    * `libgcc_s.so.1` 是 nix 的 gcc10 中的。
    * 其他动态链接库都是 glibc 2.31 中。
* 执行时，因为 `libgcc_s.so.1` 依赖更新版本的 glibc 2.34，但是加载的是 2.31，所以运行不起来。
* 通过 `pathelf` 去除 rpath，此时，可执行文件将会 debian 12 系统路径中找 `libgcc_s.so.1` 和 glibc，此时版本是满足约束的，因此可以正常执行。
* 最后，通过此方式编译出的可执行文件可以在 glibc 版本 >= 2.28 以上的 Linux 平台中运行，而之前使用构建参数构建的可执行文件只能在 glibc 版本大于等于 2.34 的 Linux 系统中运行。

最后，使用 `nix-env --uninstall gcc-wrapper-10.5.0` 清理现场。

## 与动态库有关配置总结

`~.cargo/config.toml`

```toml
[build]
target = "triple"             # 构建目标架构，默认为当前操作系统的架构如 x86_64-unknown-linux-gnu，可以配置为 x86_64-unknown-linux-musl 实现静态链接。也可以通过 cargo build --target 或 `CARGO_BUILD_TARGE` 环境变量指定。
rustflags = ["…", "…"]        # 传递给 rustc 的参数可以是字符串或字符串数组，可以 "-L native=/path/to/lib" 指定动态库查找路径。也可以通过 cargo build --config "build.rustflags='xxx'" 或环境变量 `CARGO_BUILD_RUSTFLAGS`、`CARGO_ENCODED_RUSTFLAGS`、`RUSTFLAGS` 指定。


[target.<triple>]         # 构建目标架构的详细配置，如 x86_64-unknown-linux-musl。
linker = "…"              # 链接器的路径，一般情况下默认为 PATH 中的 gcc，配置 musl 时，可以前往 musl.cc 下载交叉编译工具链，并将 x86_64-linux-musl-ld 的绝对路径配置到这里。也可以通过 cargo build --config "target.x86_64-unknown-linux-musl.linker='xxx'" 或环境变量 `CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_LINKER` 指定。
rustflags = ["…", "…"]    # 用途和 build.rustflags 一样。也可以通过 cargo build --config "target.x86_64-unknown-linux-musl.rustflags='xxx'" 或环境变量 `CARGO_TARGET_X86_64_UNKNOWN_LINUX_MUSL_RUSTFLAGS` 指定。
```

## 参考

* [官网 - Get Started](https://www.rust-lang.org/learn/get-started)
* [rustc - 支持的平台](https://doc.rust-lang.org/nightly/rustc/platform-support.html)
* [rustc - 内建的目标](https://doc.rust-lang.org/rustc/targets/built-in.html)
* [rustc - 自定义目标](https://doc.rust-lang.org/rustc/targets/custom.html)
* [rustup-components-history](https://rust-lang.github.io/rustup-components-history/x86_64-unknown-linux-gnu.html)
* [The Cargo Book - Configuration](https://doc.rust-lang.org/cargo/reference/config.html)
* [The rustc book - Codegen Options - linker](https://doc.rust-lang.org/rustc/codegen-options/index.html#linker)
* [Build glibc from source](https://github.com/jueve/build-glibc)
* [toolchains.bootlin.com](https://toolchains.bootlin.com/)
* [glibc ftp](https://ftp.gnu.org/gnu/glibc/)
* [pathelf](https://github.com/NixOS/patchelf)
* [Rust交叉编译arm64 linux环境设置](https://www.cnblogs.com/turingguo/p/17406999.html)
