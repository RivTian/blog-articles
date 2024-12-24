#Rust 

在这篇文章中，我们将讨论如何在 Rust 中实现 API 限速。当涉及到生产中的服务时，是为了确保不良行为者不会滥用 API——这就是 API 限速的作用所在。

我们将实现 “滑动窗口” 算法，通过动态周期来检查请求历史，并使用基本的内存 hashmap 来存储用户 IP 及其请求时间。我们还将研究如何使用 tower-governor 库来实现限速。

 实现一个简单的滑动窗口限速器 

让我们从头开始编写一个简单的基于 IP 的滑动窗口限速器。

 在项目中加入以下依赖项：

```toml
[dependencies]
chrono = { version = "0.4.34", features = ["serde", "clock"] }
serde = { version = "1.0.196", features = ["derive"] }
```

我们将声明一个新的结构体 RateLimiter，它保存 IP 地址作为键的 HashMap，值为 Vec\<DateTime>(Utc 时区的时间戳)。

```rust
use std::sync::{Arc, Mutex};
use std::collections::HashMap;
use std::net::IpAddr;
use chrono::{DateTime, Utc};

// 這是用戶訪問端點的請求限制(每分鐘)
// 如果用戶試圖超過這個限制，返回一個錯誤
const REQUEST_LIMIT: usize = 120;

#[derive(Clone, Default)]
pub struct RateLimiter {
    requests: Arc<Mutex<HashMap<IpAddr, Vec<DateTime<Utc>>>>>,
}
```

首先，我们想要通过使用.lock() 来锁定我们的 HashMap，这给了我们写访问权限。然后，我们需要检查hashmap 是否包含一个键，其中包含我们想要使用.entry() 函数检查的IP 地址，然后通过保留有效的时间戳来修改它，并根据长度是否在请求限制之下push 一个新记录。然后检查记录长度是否大于请求限制—如果是，则返回错误；如果不是，则返回 Ok(())。

```rust
impl RateLimiter {
    fn check_if_rate_limited(&self, ip_addr: IpAddr) -> Result<(), String> {
        // 我們只想保留60秒前的時間戳
        let throttle_time_limit = Utc::now() - std::time::Duration::from_secs(60);

        let mut requests_hashmap = self.requests.lock().unwrap();

        let mut requests_for_ip = requests_hashmap
            // 檢索記錄，並允許我們在適當的地方修改它
            .entry(ip_addr)
            // 如果記錄爲空，則插入帶有當前時間戳的vec
            .or_default();

        requests_for_ip.retain(|x| x.to_utc() > throttle_time_limit);
        requests_for_ip.push(Utc::now());

        if requests_for_ip.len() > REQUEST_LIMIT {
            return Err("IP is rate limited :(".to_string());
        }

        Ok(())
    }
}
```

测试如下：

```rust
fn main() {
    let rate_limiter = RateLimiter::default();

    let localhost_v4 = IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1));

    // 這裏我們請求120次
    for _ in 1..80 {
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())
    }

    // 等待30秒
    std::thread::sleep(std::time::Duration::from_secs(30));

    // 在這裏再做40個請求
    for _ in 1..40 {
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())
    }

    // 再等30秒
    std::thread::sleep(std::time::Duration::from_secs(30));

    // 現在我們可以再做80個請求
    for _ in 1..80 {
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())
    }
}
```

然而，生产上的限速系统通常比这先进得多。我们将在下面讨论如何利用 tower-governor crate 进行速率限制。

 使用 tower-governor 实现限速器 

提供速率限制的 tower 服务和层，后端是 governo 库。 tower-governor 很大程度上基于 actix-governor 所做的工作，可以与 Axum, Hyper, Tonic 和其他任何基于 tower 组件库的框架一起使用！

 tower-governor 的特点：

- 速率限制请求基于对端的 IP 地址，IP 地址头或通过自定义设置 
    
-  自定义流量限制的标准是按每秒计算 
    
-  使用简单 
    
-  高可定制性 
    
- 高性能
    
-  健壮而灵活的 API 
    

它是如何工作的？

每个调控器中间件都有一个存储配额的配置，配额指定可以从一个 IP 地址发送多少请求，如果超过配额则阻止进一步请求。

例如，如果配额允许 10 个请求，客户机可以在中间件开始阻塞之前的短时间内发送 10 个请求。

一旦使用了至少一个配额元素，配额元素将在指定时间后补充。

例如，如果周期为 2 秒，并且配额为空，则需要 2 秒来补充配额的一个元素。这意味着可以平均每两秒钟发送一个请求。

如果配额允许在同一时间段内发送10 个请求，则客户端可以再次发送10 个请求的突发事件，然后必须等待2 秒才能发送进一步的请求，或者在完整配额被补充之前等待20 秒，他可以发送另一个突发事件。

 下面我们来实现一个例子。

 首先，在项目中加入以下依赖项：

建议版本一致

```toml
[dependencies]
tokio = {version = "1.36.0", features = ["full"] }
tower_governor = "0.3.2"
axum = "0.7" 
tracing = {version="0.1.37", features=["attributes"]}
tracing-subscriber = "0.3"
tower = "0.4.13"
```

然后，在 main.rs 文件中写入以下代码：

```rust
use axum::{routing::get, Router};
use std::net::SocketAddr;
use std::time::Duration;
use tokio::net::TcpListener;
use tower_governor::{governor::GovernorConfigBuilder, GovernorLayer};

async fn hello() -> &'static str {
   "Hello world"
}

#[tokio::main]
async fn main() {
   // 配置跟蹤
   // 構造一個訂閱者，將格式化的跟蹤信息打印到標準輸出
   let subscriber = tracing_subscriber::FmtSubscriber::new();
   // 使用subscriber處理在此點之後發出的跟蹤
   tracing::subscriber::set_global_default(subscriber).unwrap();

   // 允許每個IP地址最多有五個請求，每兩秒鐘補充一個
   // 我們將其裝箱是因爲Axum 0.6要求所有層都是克隆的，因此我們需要一個靜態引用
   let governor_conf = Box::new(
       GovernorConfigBuilder::default()
           .per_second(2)
           .burst_size(5)
           .finish()
           .unwrap(),
   );

   let governor_limiter = governor_conf.limiter().clone();
   let interval = Duration::from_secs(60);
   // 一個單獨的後臺任務
   std::thread::spawn(move || {
       loop {
           std::thread::sleep(interval);
           tracing::info!("rate limiting storage size: {}", governor_limiter.len());
           governor_limiter.retain_recent();
       }
   });

   // 構建路由
   let app = Router::new()
       // `GET /` 
       .route("/", get(hello))
       .layer(GovernorLayer {
           // 我們可以泄漏它，因爲它是一次性創建的
           config: Box::leak(governor_conf),
       });

   let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
   tracing::debug!("listening on {}", addr);
   let listener = TcpListener::bind(addr).await.unwrap();
   axum::serve(listener, app.into_make_service_with_connect_info::<SocketAddr>())
       .await
       .unwrap();
}
```

运行服务器后，在浏览器中刷新请求，2 秒内超过 5 次会提示以下错误：

![](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/Xrj4Bv.png)


服务器后台跟踪日志如下：

```sh
2024-05-09T05:41:10.065323Z  INFO limit_api2: rate limiting storage size: 1
2024-05-09T05:41:20.067175Z  INFO limit_api2: rate limiting storage size: 1
2024-05-09T05:41:30.071745Z  INFO limit_api2: rate limiting storage size: 1
2024-05-09T05:41:40.073187Z  INFO limit_api2: rate limiting storage size: 1
2024-05-09T05:41:50.077540Z  INFO limit_api2: rate limiting storage size: 0
2024-05-09T05:42:00.077448Z  INFO limit_api2: rate limiting storage size: 0
```