
#Rust 

Doc.rs: [Here](https://docs.rs/reqwest/latest/reqwest/)

---

`reqwest` 是 Rust 中一个非常流行和强大的 HTTP 客户端库，它提供了一种简单的方式来发送 HTTP 请求并处理响应。`reqwest` 支持阻塞和非阻塞（异步）请求，使其适合于各种不同的应用场景。在这篇博文中，我们将详细介绍如何使用 `reqwest` 发送各种 HTTP 请求，并处理返回的响应。

### 开始之前

在开始编写代码之前，你需要在你的 Rust 项目中添加 `reqwest` 依赖。打开你的 `Cargo.toml`文件，并添加以下内容：

```text
[dependencies]
reqwest = "0.11.24"
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0.197", features = ["derive"] }
serde_json = "1.0.114"
```

这里我们还添加了其他几个依赖：

- `tokio`: 在后面的示例中，我们将使用 `reqwest` 的异步功能。
- `serde`: 用于数据解析，在示例中，我们会演示 json 数据的解析。 
- `serde_json`: 便于使用 `json!` 宏快速构建 json 数据。

### 发送 GET 请求

发送一个 GET 请求是最基本的 HTTP 操作。以下是如何使用 `reqwest` 发送 GET 请求并设置请求头的示例：

```rust
use reqwest::header;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error>{
    let params = [("key1", "value1"), ("key2", "values")];
    let client = reqwest::Client::new();
    let body = client.get("http://httpbin.org/get")
            // set query params
        .form(&params)
            // set request headers
        .header(header::USER_AGENT, "My Rust Program")
        .header(header::CONTENT_TYPE, "application/json")
        .await?
        .text()
        .await?;
    println!("body = {:?}", body);
    Ok(())
}
```

在这个例子中，我们使用 `reqwest::get` 函数发送一个 GET 请求到 "[https://httpbin.org/get](https://link.zhihu.com/?target=https%3A//httpbin.org/get)"，并通过 `text` 方法获取响应的文本内容。

### 发送 POST - text 请求

```rust
use reqwest::Client;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let res = client.post("http://httpbin.org/post")
        .body("the exact body that is sent")
        .send()
        .await?
        .text()
        .await?;

    println!("body: {:?}", res);
    Ok(())
}
```

### 发送 POST - form 请求

```rust
#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let params = [("key1", "value1"), ("key2", "values")];
    let client = reqwest::Client::new();
    let res = client.post("http://httpbin.org/post")
        .form(&params)
        .send()
        .await?
        .text()
        .await?;

    println!("body: {:?}", res);
    Ok(())
}
```

### 发送 POST - json 请求

发送 POST 请求通常用于向服务器提交数据。以下是如何使用 `reqwest` 发送包含 JSON 数据的 POST 请求的示例：

```rust
use reqwest;
use serde_json::json;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();
    let res = client.post("https://httpbin.org/post")
        .json(&json!({"key": "value"}))
        .send()
        .await?;

    let body = res.text().await?;
    println!("Body:\n{}", body);

    Ok(())
}
```

这里我们使用 `Client::post` 方法创建一个 POST 请求，并通过 `json` 方法设置 JSON 负载。然后，我们调用 `send` 方法发送请求。

### 处理 JSON 响应

```rust
use reqwest;
use serde::Deserialize;

#[derive(Deserialize)]
struct Ip {
    origin: String,
}

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let ip: Ip = reqwest::get("https://httpbin.org/ip")
        .await?
        .json()
        .await?;

    println!("IP: {}", ip.origin);
    Ok(())
}
```

在这个示例中，我们定义了一个 `Ip` 结构体来表示 JSON 响应，然后使用 `json` 方法将响应反序列化为 `Ip` 类型。

### 总结

`reqwest` 库为 Rust 提供了一个功能丰富而灵活的 HTTP 客户端，适用于各种网络编程任务。无论是简单的数据获取还是复杂的 API 交互，`reqwest` 都能帮助你以简洁的 Rust 代码完成任务。希望这篇博文能帮助你开始使用 `reqwest` 来开发网络相关的 Rust 应用！

部分涉及代理操作的教程：[Here](https://zhuanlan.zhihu.com/p/572150411)
