#Rust 

在这篇文章中，我们将讨论如何在Rust中实现API限速。当涉及到生产中的服务时，是为了确保不良行为者不会滥用API——这就是API限速的作用所在。

我们将实现“滑动窗口”算法，通过动态周期来检查请求历史，并使用基本的内存hashmap来存储用户IP及其请求时间。我们还将研究如何使用 `tower-governor` 库来实现限速。

## 实现一个简单的滑动窗口限速器

让我们从头开始编写一个 Simple，base on ip 的滑动窗口限速器。

在项目中加入以下依赖：

```toml
[dependencies]  
chrono = { version = "0.4.35", features = ["serde", "clock"] }  
serde = { version = "1.0.197", features = ["derive"] }
```

我们将声明一个新的结构体 `RateLimiter`，它保存IP地址作为键的HashMap，值为 `Vec<DateTime<Utc>>` (Utc时区的时间戳)。

```rust
use std::sync::{Arc, Mutex};  
use std::collections::HashMap;  
use std::net::IpAddr;  
use chrono::{DateTime, Utc};  
  
// 这是用户访问端点的请求限制(每分钟)  
// 如果用户试图超过这个限制，返回一个错误  
const REQUEST_LIMIT: usize = 120;  
  
#[derive(Clone, Default)]  
pub struct RateLimiter {  
    requests: Arc<Mutex<HashMap<IpAddr, Vec<DateTime<Utc>>>>>,  
}
```

首先，我们想要通过使用 `.lock()` 来锁定我们的 $HashMap$，这给了我们写访问权限。
然后，我们需要检查hashmap 是否包含一个键，其中包含我们想要使用 `.entry()` 函数检查的IP地址，
其次，通过保留有效的时间戳来修改它，并根据长度是否在请求限制之下push一个新记录。
最后，检查记录长度是否大于请求限制—如果是，则返回错误；如果不是，则返回 $Ok(())$。

```rust
impl RateLimiter {  
    fn check_if_rate_limited(&self, ip_addr: IpAddr) -> Result<(), String> {  
        // 我们只想保留60秒前的时间戳  
        let throttle_time_limit = Utc::now() - std::time::Duration::from_secs(60);  
  
        let mut requests_hashmap = self.requests.lock().unwrap();  
  
        let mut requests_for_ip = requests_hashmap  
            // 检索记录，并允许我们在适当的地方修改它  
            .entry(ip_addr)  
            // 如果记录为空，则插入带有当前时间戳的vec  
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
  
    // 这里我们请求120次  
    for _ in 1..80 {  
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())  
    }  
  
    // 等待30秒  
    std::thread::sleep(std::time::Duration::from_secs(30));  
  
    // 在这里再做40个请求  
    for _ in 1..40 {  
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())  
    }  
  
    // 再等30秒  
    std::thread::sleep(std::time::Duration::from_secs(30));  
  
    // 现在我们可以再做80个请求  
    for _ in 1..80 {  
        assert!(rate_limiter.check_if_rate_limited(localhost_v4).is_ok())  
    }  
}
```

然而，生产上的限速系统通常比这先进得多。我们将在下面讨论如何利用 `tower-governor` crate 进行速率限制。


## 使用tower-governor实现限速器

提供速率限制的tower服务和层，后端是governo库。`tower-governor` 很大程度上基于 `actix-governor` 所做的工作，可以与Axum, Hyper, Tonic和其他任何基于tower组件库的框架一起使用！

**tower-governor的特点：**

- 速率限制请求基于对端的IP地址，IP地址头或通过自定义设置
    
- 自定义流量限制的标准是按每秒计算
    
- 使用简单
    
- 高可定制性
    
- 高性能
    
- 健壮而灵活的API
    

**它是如何工作的？**

每个调控器中间件都有一个存储配额的配置，配额指定可以从一个IP地址发送多少请求，如果超过配额则阻止进一步请求。

例如，如果配额允许10个请求，客户机可以在中间件开始阻塞之前的短时间内发送10个请求。

一旦使用了至少一个配额元素，配额元素将在指定时间后补充。

例如，如果周期为2秒，并且配额为空，则需要2秒来补充配额的一个元素。这意味着可以平均每两秒钟发送一个请求。

如果配额允许在同一时间段内发送10个请求，则客户端可以再次发送10个请求的突发事件，然后必须等待2秒才能发送进一步的请求，或者在完整配额被补充之前等待20秒，他可以发送另一个突发事件。

下面我们来实现一个例子。

首先，在项目中加入以下依赖项：

```toml
[dependencies]  
tokio = {version = "1.36.0", features = ["full"] }  
tower_governor = "0.3.2"  
axum = "0.7"  
tracing = {version="0.1.37", features=["attributes"]}  
tracing-subscriber = "0.3"  
tower = "0.4.13"
```

然后，在main.rs文件中写入以下代码：

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
   // 配置跟踪  
   // 构造一个订阅者，将格式化的跟踪信息打印到标准输出  
   let subscriber = tracing_subscriber::FmtSubscriber::new();  
   // 使用subscriber处理在此点之后发出的跟踪  
   tracing::subscriber::set_global_default(subscriber).unwrap();  
  
   // 允许每个IP地址最多有五个请求，每两秒钟补充一个  
   // 我们将其装箱是因为Axum 0.6要求所有层都是克隆的，因此我们需要一个静态引用  
   let governor_conf = Box::new(  
       GovernorConfigBuilder::default()  
           .per_second(2)  
           .burst_size(5)  
           .finish()  
           .unwrap(),  
   );  
  
   let governor_limiter = governor_conf.limiter().clone();  
   let interval = Duration::from_secs(60);  
   // 一个单独的后台任务  
   std::thread::spawn(move || {  
       loop {  
           std::thread::sleep(interval);  
           tracing::info!("rate limiting storage size: {}", governor_limiter.len());  
           governor_limiter.retain_recent();  
       }  
   });  
  
   // 构建路由  
   let app = Router::new()  
       // `GET /`   
       .route("/", get(hello))  
       .layer(GovernorLayer {  
           // 我们可以泄漏它，因为它是一次性创建的  
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

运行服务器后，在浏览器中刷新请求，2秒内超过5次会提示以下错误：

![TempFile](https://mmbiz.qpic.cn/mmbiz_jpg/aZSfDXbVYlPNW6Ibnic4vL6ywU01FtZNROnJZvxmkgyILTnVFHkQwnrA065bbicZZQ49PFkwX3rX1Iobokmca3dA/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1)

服务器后台跟踪日志如下：

```sh
INFO tower_governor: rate limiting storage size: 0  
INFO tower_governor: rate limiting storage size: 0  
INFO tower_governor: rate limiting storage size: 1  
INFO tower_governor: rate limiting storage size: 1  
INFO tower_governor: rate limiting storage size: 1  
INFO tower_governor: rate limiting storage size: 0
```