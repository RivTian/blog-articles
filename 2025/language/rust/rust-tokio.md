> Rust 版本：1.41.0 (1.39.0 以上)
> 前序文章: [Rust异步编程](./Rust异步编程.md)
> Tokio 版本：0.2

参考：

* [官方网站](https://tokio.rs/)
* [中文官方网站](https://tokio-zh.github.io/)（有点过时）
* [API 文档](https://docs.rs/tokio/0.2.11/tokio/)

创建测试项目 `cargo new tokio-learn`

## 一、简介

Tokio是一个事件驱动的非阻塞I / O平台，用于使用Rust编程语言编写异步应用程序。在较高的层面上，它提供了一些主要组件：

* 基于多线程，工作窃取的任务调度程序。
* 由操作系统的事件队列（epoll，kqueue，IOCP等）支持的反应器。
* 异步TCP和UDP套接字。

这些组件提供构建异步应用程序所需的运行时组件。

参见：https://tokio-zh.github.io/document/

个人理解：

* 因为 Rust 的 异步实现 是基于非回调的 轮询 实现的，且标准库中并不提供 执行器（轮询器）
* Tokio 提供了 rust 异步机制所需要的 运行时环境（Executor），并依托于该异步环境，提供了大量开箱即用的异步库
* Tokio 基本上是 实际意义上的官方异步执行器

## 二、Getting Started

### 1、网络调试工具安装

安装 `socat` 和 `telnet`

* [`socat`](http://www.dest-unreach.org/socat/) 即 Socket CAT，一个多功能的网络工具。可以模拟 一个 Socket 服务器。
    * Mac 安装 `brew install socat`
* `telnet` 一种远程连接客户端
    * Mac 安装 `brew install telnet`

基本使用

* 创建一个 tcp socket 服务端 监听在 6142 `socat TCP-LISTEN:6142,fork stdout`
* 连接到 6142 `telnet localhost 6142`，然后随意输入内容

### 2、Hello World

添加依赖 `Cargo.toml`

```toml
[dependencies]
tokio = { version = "0.2", features = ["full"] }
```

`src/main.rs`

```rust
use tokio::net::TcpStream;
use tokio::prelude::*;

#[tokio::main] // 该宏创建了一个tokio运行时环境
async fn main() {
    // === tcp 客户端 ===
    // 创建一个Tcp连接
    let mut stream: TcpStream = TcpStream::connect("127.0.0.1:6142").await.unwrap();
    println!("created stream");

    // 向Tcp连接中写入数据
    let result = stream.write(b"hello world\n").await;
    println!("wrote to stream; success={:?}", result.is_ok());

    // 关闭Tcp连接
    if let Ok(()) = stream.shutdown(std::net::Shutdown::Write) {
        println!("close stream success");
    }
}
```

测试

* 启动 `socat TCP-LISTEN:6142,fork stdout` 服务端
* 启动测试代码 `cargo run`

### 3、依赖

```toml
[dependencies]
tokio = { version = "0.2", features = ["full"] }
```

一般情况下，当编写 针对普通用户的 App 时，使用全部特性即可 也就是 `features = ["full"]`

当开发一个针对开发者的库时，需要选择需要使用的特性，这样能显著减少编译时间。

#### （1）宏和核心运行时

当需要使用 `#[tokio::main]` 时，需要引入 `macros` 特性，如果我们是服务端程序，需要启用多线程，则建议启用多线程运行时特性 `rt-threaded`

```toml
tokio = { version = "0.2", features = ["macros", "rt-threaded"] }
```

当在低性能的客户端上使用时，建议使用 `rt-core`，单线程调度器

```toml
tokio = { version = "0.2", features = ["macros", "rt-core"] }
```

#### （2）Tcp 流连接

在 Hello World 例子中

* 使用了 `tokio::net::TcpStream`。因此需要启用 `net` 特性
* 使用了 `stream.write(b"hello world\n").await;`。所以需要启用 `io-util` 特性

#### （3）更多

因此 Hello World 的依赖可以改成

```toml
tokio = { version = "0.2", features = ["macros", "rt-threaded", "net", "io-util"] }
```

参见： https://docs.rs/tokio/0.2.11/tokio/#feature-flags

### 4、例子：Echo服务器

添加依赖 `Cargo.toml`

```toml
[dependencies]
tokio = { version = "0.2", features = ["full"] }
futures = "0.3"
```

`src/main.rs`

```rust
use tokio::net::TcpStream;
use tokio::prelude::*;  // 引入预定义的实现

use tokio::net::TcpListener;
use futures::stream::StreamExt;


#[tokio::main] // 该宏创建了一个tokio运行时环境
async fn main() {
    // === tcp 服务端 Echo 服务 ===
    let addr = "127.0.0.1:6143";
    let mut listener:TcpListener = TcpListener::bind(addr).await.unwrap();

    println!("Server running on {}", addr);

    // 以下实现通过 Stream 方式实现
    async move {
        // 将 TcpListener 转换为一个 Stream
        let mut incoming = listener.incoming();
        // 等待 stream 获取到数据（即客户端的连接）
        // 依赖 tokio 的 stream feature 和 futures = "0.3"
        while let Some(conn) = incoming.next().await {
            match conn {
                Err(e) => eprintln!("accept failed = {:?}", e),
                Ok(mut sock) => {
                    // 当收到一个Tcp连接时，提交一个 Future 到 tokio 运行时
                    tokio::spawn(async move {
                        // 获取到读写句柄
                        let (mut reader, mut writer) = sock.split();

                        // 将接收到的数据写回，完成Echo
                        // 使用 tokio::io::copy 方法，同样该方法是异步的
                        match tokio::io::copy(&mut reader, &mut writer).await {
                            Ok(amt) => {
                                println!("wrote {} bytes", amt);
                            }
                            Err(err) => {
                                eprintln!("IO error {:?}", err);
                            }
                        }
                    });
                }
            }
        }
    }.await;

    // // 直观的实现实现
    // loop {
    //     // Asynchronously wait for an inbound socket.
    //     // 异步等待 客户端 连接
    //     let (mut socket, _): (TcpStream, _) = listener.accept().await.unwrap();

    //     // 获取到客户端连接后，提交一个异步任务到 tokio 运行时，用来处理运行客户端连接
    //     tokio::spawn(async move {
    //         // buffer
    //         let mut buf = [0; 1024];

    //         // In a loop, read data from the socket and write the data back.
    //         // 循环等待用户输入
    //         loop {
    //             // 异步等待用户输入数据到buffer中
    //             let n = socket
    //                 .read(&mut buf)
    //                 .await
    //                 .expect("failed to read data from socket");

    //             if n == 0 {
    //                 return;
    //             }

    //             // 写会到客户端
    //             socket
    //                 .write_all(&buf[0..n])
    //                 .await
    //                 .expect("failed to write data to socket");
    //         }
    //     });
    // }
}
```

测试

* 启动测试代码 `cargo run`
* `telnet localhost 6143` 随意输入

## 三、Runtime

Rust Future需要一些东西来轮询它的完成，Tokio就是这样的运行时

运行时负责重复调用Future上的poll，直到返回其值。在实践中可以通过几种不同的方式进行。例如，basic_scheduler配置将阻止当前线程并处理所有产生的任务。threaded_scheduler配置使用窃取工作的线程池，并在多个线程之间分配负载。threaded_scheduler是应用程序的默认值，basic_scheduler是测试的默认值。

需要注意的是：

* async 代码块中 的 所有 Future 必须被 `await` 或者 通过 `tokio::spawn` 提交，否则 运行时将不知道要轮训那些 Future
* `await` 和 `tokio::spawn` 的区别在于
    * `await` 表示要 `Future` 在 当前 `Task` 中被轮训
    * `tokio::spawn` 表示要 `Future` 作为新的任务被轮训
    * 一般使用 `tokio::spawn` 该 `Future` 相对独立于当前任务，可以理解成一个轻量级线程
    * 如果在多线程的执行环境中，`tokio::spawn` 将可以更充分的利用 CPU 资源

## 四、常用API

### 1、`tokio::task`

https://docs.rs/tokio/0.2.11/tokio/task/index.html

* `spawn` 提交一个异步任务
* `spawn_blocking` 提交一个阻塞任务到专有线程池，防止阻塞轮训线程
* `block_in_place` （仅 `rt_threaded` 时可用）在当前线程中执行阻塞任务，将其他任务移动到其他工作线程
* `yield_now` 让出CPU，让其他任务先执行一轮
