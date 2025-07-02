> Rust 版本：1.41.0 (1.39.0 以上)
> 前序文章: [Rust Tokio](./rust-tokio.md)
> Actix 版本：2.0

参考：

* [官方网站](https://actix.rs/)

创建测试项目 `cargo new actix-learn`

## 一、介绍

actix 是 Rust 生态中的 Actor 系统。

actix-web 是 actix actor框架 和 Tokio异步IO系统 之上构建的高级Web框架。

`Cargo.toml` 配置依赖

```toml
[dependencies]
actix-web = "2.0"
```

开发者版本

```toml
[dependencies]
actix-web = { git = "https://github.com/actix/actix-web" }
```

运行例子程序

```bash
git clone https://github.com/actix/examples
cd examples/basics
cargo run
```

## 二、基本

### 1、Hello World

`Cargo.toml` 配置依赖

```toml
[dependencies]
actix-web = "2.0"
actix-rt = "1.0"
```

`src/main.rs`

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// curl http://localhost:8088/
async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

// curl http://localhost:8088/again
async fn index2() -> impl Responder {
    HttpResponse::Ok().body("Hello world again!")
}

use actix_web::get;

// curl http://localhost:8088/hello
// 使用宏解析
#[get("/hello")]
async fn index3() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            .route("/again", web::get().to(index2))
            .service(index3)
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

* `#[actix_rt::main]` 用于生成异步函数运行时，需要引入 `actix-rt = "1.0"` 依赖
* 处理函数应该是一个异步函数，返回一个 实现了 `Responder` 的类型
* `#[get("/hello")]` 可以方便的配置请求宏，参见：https://docs.rs/actix-web-codegen/
* 文件修改自动编译重载，参见：https://actix.rs/docs/autoreload/

### 2、App

`actix_web::App` 是 `actix_web` 的核心，所有的路由、服务、共享数据都围绕 `App` 构建。

在 `actix_web` 中，每个线程持有一个 App 实例。在创建 Server 时，需要传递一个 App 的 工厂函数

#### （1）统一前缀

一个 App 可以通过 [`scope`](https://docs.rs/actix-web/2/actix_web/struct.Scope.html)，为路由添加统一的前缀。

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// curl http://localhost:8088/
// curl http://localhost:8088/app/index.html
async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

use actix_web::get;

// curl http://localhost:8088/hello
// curl http://localhost:8088/app/hello
// 使用宏解析
#[get("/hello")]
async fn index3() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(
                web::scope("/app")
                    .route("/index.html", web::get().to(index))
                    .service(index3)
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

#### （2）共享状态

`actix_web` 提供了 `web::Data` API 用来在程序间共享状态

* 线程级别共享，共享的类型不用实现 线程交换安全，只能用于**只读**，如全局配置。通过 `.data(T)` 初始化
* 进程级别共享，共享的类型需要实现 线程交换安全，可用于读写场景，如计数器。通过 `.app_data(T)` 初始化

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

struct AppState {
    app_name: String,
}

// curl http://localhost:8088/app_state
async fn app_state(data: web::Data<AppState>) -> impl Responder {
    HttpResponse::Ok().body(&data.app_name)
}

use std::sync::Mutex;

struct AppStateWithCounter {
    counter: Mutex<i32>, // <- Mutex is necessary to mutate safely across threads
}

// curl http://localhost:8088/counter
async fn counter(data: web::Data<AppStateWithCounter>) -> String {
    let mut counter = data.counter.lock().unwrap(); // <- get counter's MutexGuard
    *counter += 1; // <- access counter inside MutexGuard

    format!("Request number: {}", counter) // <- response with count
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    let c = web::Data::new(AppStateWithCounter {
        counter: Mutex::new(0),
    });
    HttpServer::new(move || {
        App::new()
            // 由于 HttpServer::new 接收的是 App 工厂函数
            // 所以不同线程的 data 不是同一个实例，所以不是进程级别共享数据，而是线程级别的共享数据
            // 因此只能用于访问只读数据，如全局配置等
            .data(AppState {
                app_name: String::from("Actix-web"),
            })
            .app_data(c.clone())
            .route("/", web::get().to(index))
            .route("/again", web::get().to(index2))
            .service(index3)
            .service(
                web::scope("/app")
                    .route("/index.html", web::get().to(index))
                    .service(index3)
            )
            .route("/app_state", web::get().to(app_state))
            .route("/counter", web::get().to(counter))
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

#### （3）应用级别守卫

参见： https://docs.rs/actix-web/2/actix_web/guard/trait.Guard.html

#### （4）配置

actix_web 提供了 `configure` 用来传递一个配置函数，这样就可以将实现拆分到不同的模块中实现。

* 该配置函数 传递一个参数 [`ServiceConfig`](https://docs.rs/actix-web/2/actix_web/web/struct.ServiceConfig.html)，该参数可以配置自己的 `data`, `routes`, 和 `services`。

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};

// this function could be located in different module
// curl http://localhost:8088/app3/test
fn scoped_config(cfg: &mut web::ServiceConfig) {
    cfg.service(
        web::resource("/test")
            .route(web::get().to(|| HttpResponse::Ok().body("test")))
            .route(web::head().to(|| HttpResponse::MethodNotAllowed())),
    );
}

// this function could be located in different module
// curl http://localhost:8088/app2
fn config(cfg: &mut web::ServiceConfig) {
    cfg.service(
        web::resource("/app2")
            .route(web::get().to(|| HttpResponse::Ok().body("app2")))
            .route(web::head().to(|| HttpResponse::MethodNotAllowed())),
    );
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    let c = web::Data::new(AppStateWithCounter {
        counter: Mutex::new(0),
    });
    HttpServer::new(move || {
        App::new()
            .configure(config)
            .service(
                web::scope("/app3")
                    .configure(scoped_config)
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

### 3、Server

[HttpServer](https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html) 负责 处理 HTTP 请求

`HttpServer` 的构造函数 以 `App` 工厂作为参数（必须实现`Send` + `Sync`），然后使用 `bind` 绑定端口，`run` 函数将启动 Http Server

默认情况下，HttpServer 以多线程方式 启动 Server，线程为等于当先系统的核心数。可以通过如下方式，指定线程数。

```rust
use actix_web::{web, App, HttpResponse, HttpServer};

#[actix_rt::main]
async fn main() {
    HttpServer::new(|| {
        App::new().route("/", web::get().to(|| HttpResponse::Ok()))
    })
    .workers(4); // <- Start 4 workers
}
```

由于 actix_web 是异步的，所以要防止阻塞的发生，阻塞将大大降低系统的吞吐量。所以，所有 IO 操作都需要使用对应的异步版本来实现。

默认情况下，actix_web 当收到 `SIGTERM` 信号时，将优雅关机，关机时间超时默认30s，可以通过` HttpServer::shutdown_timeout()` 配置

以下内容，参考 https://actix.rs/docs/server/

* SSL
* Keep-Alive

### 4、请求处理器

处理器是一个异步函数：`async fn(p: impl FromRequest) -> impl Responder`，其中参数p的数目支持`0~10`个

关于 `FromRequest` 参见下一节 提取器

actix-web 为某些标准类型（例如`＆'static str`、`String`等）提供 `Responder` 实现。

以下是关于不同 `Responder` 的实验

添加如下依赖

```toml
serde = "1.0"
serde_json = "1.0"
futures = "0.3"
```

`src/main.rs`

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder, Error, HttpRequest, Either};

// curl http://localhost:8088/responder/str
async fn responder_str() -> &'static str {
    "responder_str"
}

// curl http://localhost:8088/responder/string
async fn responder_string() -> String {
    "responder_string".to_owned()
}

// curl http://localhost:8088/responder/impl_responder
async fn responder_impl_responder() -> impl Responder{
    web::Bytes::from_static(b"responder_string")
}

use serde::Serialize;
use futures::future::{ready, Ready};

// 自定义 Response
#[derive(Serialize)]
struct ResponseWrapper<T> {
    code: i32,
    msg: String,
    data: Option<T>,
}

// Responder
impl <T> Responder for ResponseWrapper<T> where T: Serialize {
    type Error = Error;
    type Future = Ready<Result<HttpResponse, Error>>;

    fn respond_to(self, _req: &HttpRequest) -> Self::Future {
        let body = serde_json::to_string(&self).unwrap();

        // Create response and set content type
        ready(Ok(HttpResponse::Ok()
            .content_type("application/json")
            .body(body)))
    }
}

// curl http://localhost:8088/responder/custom_responder
async fn responder_custom_responder() -> impl Responder {
    ResponseWrapper {
        code: 0,
        msg: "success".to_string(),
        data: Some("custom_responder".to_string()) }
}

use futures::stream::once;
use futures::future::ok;

// curl http://localhost:8088/responder/stream
async fn responder_stream_responder() -> HttpResponse {
    let body = once(ok::<_, Error>(web::Bytes::from_static(b"test")));

    HttpResponse::Ok()
        .content_type("application/json")
        .streaming(body)
}

type RegisterResult = Either<HttpResponse, Result<&'static str, Error>>;

// curl http://localhost:8088/responder/either
async fn responder_either_responder() -> RegisterResult {
    Either::A(HttpResponse::BadRequest().body("Bad data"))
    // Either::B(Ok("Hello!"))
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {

    let c = web::Data::new(AppStateWithCounter {
        counter: Mutex::new(0),
    });
    HttpServer::new(move || {
        App::new()
            .service(
                web::scope("/responder")
                    .route("/str", web::get().to(responder_str))
                    .route("/string", web::get().to(responder_string))
                    .route("/impl_responder", web::get().to(responder_impl_responder))
                    .route("/custom_responder", web::get().to(responder_custom_responder))
                    .route("/stream", web::get().to(responder_stream_responder))
                    .route("/either", web::get().to(responder_either_responder))
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}

```

### 5、提取器（Extractor）

提取器，即实现了 `FromRequest` 的类型，其可以作为 请求处理器 的参数

`actix-web` 提供了如下常用的提取器：

* `Path` 路径参数提取
* `Query`
* `Json`
* `Form`
* 其他
    * `Data` - 用于访问应用程序状态
    * `HttpRequest` - 核心请求体，包含所有请求信息
    * `String` - 将请求体转换为一个字符串
    * `bytes::Bytes` - 将请求体转换为一个字节流
    * `Payload` - 请求体的核心

除了这些专用提取器外，从1元组到10元组（元组的每个类型必须是 `FromRequest`）也实现了 `FromRequest` （代码参见 `extract.rs`），配合 `handler::Factory` 和 `handler::ExtractResponse` （代码参见 `handler.rs`），再配合 `actix_service::Service` 就实现了路由转发。

因此，本质上，在 `actix_web` 内部，就是通过 元组 和 宏实现的，如何实现通过元组调用一个多参数的函数，参见如下例子

```rust

trait CallFnWithTuple<T, R> {
    fn call_with_tuple(&self, param: T) -> R;
}

impl <Func, A, R> CallFnWithTuple<(A,), R> for Func where Func: Fn(A,) -> R {
    fn call_with_tuple(&self, param: (A,)) -> R {
        (self)(param.0,)
     }

}

impl <Func, A, B, R> CallFnWithTuple<(A, B,), R> for Func where Func: Fn(A, B,) -> R {
    fn call_with_tuple(&self, param: (A, B,)) -> R {
        (self)(param.0, param.1)
     }

}

fn proxy<T, R>(f: impl CallFnWithTuple<T, R>, p: T) -> R {
    f.call_with_tuple(p)
}

fn test_1(a: i32) -> i32 {
    a + 1
}
fn test_2(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    println!("{}", proxy(test_1, (1,)));
    println!("{}", proxy(test_2, (1,2)));
    println!("{}", test_2.call_with_tuple((1,2)));
}
```

`FromRequest` 定义如下

```rust
/// Request 提取器
///
/// 实现了该特质的类型可以作为 请求处理器 的参数使用
pub trait FromRequest: Sized {
    /// 发送错误时，返回的类型
    type Error: Into<Error>;

    /// 将 Self 转换为一个 Future
    type Future: Future<Output = Result<Self, Self::Error>>;

    /// 该提取器的配置
    type Config: Default + 'static;

    /// 将 request 转换为 Self
    fn from_request(req: &HttpRequest, payload: &mut Payload) -> Self::Future;

    /// 将 request 转换为 Self
    ///
    /// This method uses `Payload::None` as payload stream.
    fn extract(req: &HttpRequest) -> Self::Future {
        Self::from_request(req, &mut Payload::None)
    }

    /// 创建一个Config实例
    fn configure<F>(f: F) -> Self::Config
    where
        F: FnOnce(Self::Config) -> Self::Config,
    {
        f(Self::Config::default())
    }
}
```

下面就是actix_web内置的提取器的示例

```rust
use actix_web::{error, web, App, FromRequest, HttpResponse, HttpServer, Responder, Error, HttpRequest, Either};

use serde::Deserialize;

// 提取器 extractors
#[derive(Deserialize, Debug)]
struct QueryInfo {
    username: String,
}

// curl http://localhost:8088/extractor/multiple/p1/p2?username=xiaoming
async fn extractor_multiple(p: web::Path<(String, String)>, q: web::Query<QueryInfo>) -> String {
    format!("p={:?}, q={:?}", p, q)
}

#[derive(Deserialize, Debug)]
struct PathInfo {
    user_id: u32,
    friend: String,
}

// curl http://localhost:8088/extractor/path/123/friend_name
async fn extractor_path(p: web::Path<PathInfo>) -> String {
    format!("path-param={:?}", p)
}

// curl http://localhost:8088/extractor/manual_path/123/friend_name
async fn extractor_manual_path(req: HttpRequest) -> String {
    let friend: String =
        req.match_info().get("friend").unwrap().parse().unwrap();
    let user_id: i32 = req.match_info().query("user_id").parse().unwrap();

    format!("user_id={}, friend={}", user_id, friend)
}

// curl http://localhost:8088/extractor/query?username=xiaoming
async fn extractor_query(info: web::Query<QueryInfo>) -> String {
    format!("{:?}", info)
}

#[derive(Deserialize, Debug)]
struct JsonInfo {
    username: String,
}

// curl -i -H 'Content-Type: application/json' -d '{"username": "xiaoming"}' -X POST http://localhost:8088/extractor/json
// curl -i -H 'Content-Type: application/json' -d '{"username": 1}' -X POST http://localhost:8088/extractor/json
async fn extractor_json(info: web::Json<JsonInfo>) -> String {
    format!("{:?}", info)
}

#[derive(Deserialize, Debug)]
struct FormData {
    username: String,
}

/// 使用serde提取表单数据
/// 仅当内容类型为*x-www-form-urlencoded*时，才会调用此处理程序
/// 并且请求的内容可以反序列化为FormData结构
// curl -i -H 'Content-Type: application/x-www-form-urlencoded' -d 'username=xiaoming' -X POST http://localhost:8088/extractor/form
async fn extractor_form(form: web::Form<FormData>) -> String {
    format!("{:?}", form)
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(move || {
        App::new()
            // 配置 Json Extractor
            .app_data(web::Json::<JsonInfo>::configure(|cfg| {
                    cfg.limit(4096).error_handler(|err, _req| {
                        error::InternalError::from_response(
                            err,
                            HttpResponse::Conflict().finish(),
                        )
                        .into()
                    })
                }))
            .service(
                web::scope("/extractor")
                    .route("/multiple/{p1}/{p2}", web::get().to(extractor_multiple))
                    .route("/path/{user_id}/{friend}", web::get().to(extractor_path))
                    .route("/manual_path/{user_id}/{friend}", web::get().to(extractor_manual_path))
                    .route("/query", web::get().to(extractor_query))
                    .route("/json", web::post().to(extractor_json))
                    .route("/form", web::post().to(extractor_form))
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

## 三、进阶

### 1、错误处理

Actix-web 使用过 `actix_web::error::Error` 类型和 `actix_web::error::ResponseError` 特质来处理错误。

在请求处理函数 `async fn(p: impl FromRequest) -> impl Responder` 中，你只需要返回 `Result<impl Responder, impl Into<Error>>` 的 `Err` 即可 进入 内置的错误处理程序。因为，`actix-web` 为 `Result<impl Responder, impl Into<Error>>` 提供了实现

```rust
impl<T, E> Responder for Result<T, E>
where
    T: Responder,
    E: Into<Error>,
{
```

又因为 `Error` 类型 通过泛型实现了 `From<T: actix_web::error::ResponseError>` 特质（编译器隐式为  `T: actix_web::error::ResponseError` 类型 实现`Into<Error>` 特质），所以 只需要实现 `actix_web::error::ResponseError` 特质的类型（即自动实现了 `Into<Error>`），就可以可作为 `Err` 在请求处理函数中返回。

```rust
impl<T: ResponseError + 'static> From<T> for Error {
    fn from(err: T) -> Error {
        Error {
            cause: Box::new(err),
        }
    }
}
```

因此：

* `actix_web::error::Error` 是内部真实的错误类型（对用户类型进行包装）
* `actix_web::error::ResponseError` 是用户接口

`actix-web` 为常用的错误类型提供了 `ResponseError` 实现，参见：https://docs.rs/actix-web/2/actix_web/error/trait.ResponseError.html#foreign-impls

`actix_web::error::ResponseError` 特质声明如下（提供了默认实现）：

* 必须实现 `fmt::Debug + fmt::Display`
* 默认返回 500 状态码
* 将 `fmt::Display` 写回到相应体中

```rust
pub trait ResponseError: fmt::Debug + fmt::Display {
    /// 状态码：500
    fn status_code(&self) -> StatusCode {
        StatusCode::INTERNAL_SERVER_ERROR
    }

    /// Create response for error
    ///
    /// Internal server error is generated by default.
    fn error_response(&self) -> Response {
        let mut resp = Response::new(self.status_code());
        let mut buf = BytesMut::new();
        let _ = write!(Writer(&mut buf), "{}", self);
        resp.headers_mut().insert(
            header::CONTENT_TYPE,
            header::HeaderValue::from_static("text/plain; charset=utf-8"),
        );
        resp.set_body(Body::from(buf))
    }

    #[doc(hidden)]
    fn __private_get_type_id__(&self) -> TypeId
    where
        Self: 'static,
    {
        TypeId::of::<Self>()
    }
}

```

错误一般分为 内部错误 和 外部错误

* 内部错误应该展示详细细节
* 外部错误应该隐藏细节

错误日志：参见 https://actix.rs/docs/errors/

实验代码

```rust
use actix_web::{error, web, http, App, FromRequest, HttpResponse, HttpServer, Responder, Error, HttpRequest, Either, Result};

use failure::Fail;

#[derive(Fail, Debug)]
#[fail(display = "my error")]
struct MyError {
    name: &'static str,
}

impl error::ResponseError for MyError {}

// curl -i http://localhost:8088/error/custom
async fn error_custom() -> Result<&'static str, MyError> {
    Err(MyError { name: "test" })
}

#[derive(Fail, Debug)]
#[allow(dead_code)]
enum MyErrorEnum {
    #[fail(display = "internal error")]
    InternalError,
    #[fail(display = "bad request")]
    BadClientData,
    #[fail(display = "timeout")]
    Timeout,
}

use actix_http::ResponseBuilder;

impl error::ResponseError for MyErrorEnum {
    fn error_response(&self) -> HttpResponse {
        ResponseBuilder::new(self.status_code())
            .set_header(http::header::CONTENT_TYPE, "text/html; charset=utf-8")
            .body(self.to_string())
    }

    fn status_code(&self) -> http::StatusCode {
        match *self {
            MyErrorEnum::InternalError => http::StatusCode::INTERNAL_SERVER_ERROR,
            MyErrorEnum::BadClientData => http::StatusCode::BAD_REQUEST,
            MyErrorEnum::Timeout => http::StatusCode::GATEWAY_TIMEOUT,
        }
    }
}
// curl -i http://localhost:8088/error/enum
async fn error_enum() -> Result<&'static str, MyErrorEnum> {
    Err(MyErrorEnum::BadClientData)
}

// curl -i http://localhost:8088/error/helper
async fn error_helper() -> Result<&'static str> {
    let result: Result<&'static str, MyError> = Err(MyError { name: "test error" });

    Ok(result.map_err(|e| error::ErrorBadRequest(e.name))?)
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(move || {
        App::new()
            .service(
                web::scope("/error")
                    .route("/custom", web::get().to(error_custom))
                    .route("/enum", web::get().to(error_enum))
                    .route("/helper", web::get().to(error_helper))
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

### 2、URL 分发

`actix-web` 匹配用户给定 `url 模式` 进行匹配 将请求分发到 请求处理器 即 `async fn(p: impl FromRequest) -> impl Responder`，以下简称Handler

#### （1）路由配置

* (1) `App::new().route("/path", Route)` 其中 `Route` 构建方式如下
    * 常用的方法
        * `web::get().to(Handler)`
        * `web::post().to(Handler)`
        * `web::put().to(Handler)`
        * `web::patch().to(Handler)`
        * `web::delete().to(Handler)`
    * 更详细的配置 `web::route().guard(actix_web::guard::Get()).to(Handler)` 等价于 `web::get().to(Handler)`
* (2) `App::new().service(HttpServiceFactory)` 其中 `HttpServiceFactory` 构建方式如下
    * `web::resource("/path").guard(Guard).name("res_name").route(Route)` 其中 `Route` 构建方式  参见 (1)
    * `web::scope("/scope").route("/path", Route)`  其中 `Route` 构建方式 参见 (1)
    * `web::scope("/scope").service(HttpServiceFactory)`  参见 (2)

#### （2）路由匹配

* 根据路由配置生成类似如下的树形结构结构
    * `scope -> scope`
    * `scope -> path -> guard -> handler`
* `actix-web` 从 根节点 进行匹配，
    * 找到匹配的路径后将调用其 `handler`
    * 没有找到匹配将返回失配的返回 （默认为404）

#### （3）path语法

* 如果 path 不以 `/` 开头，则自动添加一个 `/`，以下两种写法等价
    * `path`
    * `/path`
* path 支持 通过 `{path_name}` 提取路径参数
    * 提取的参数 可以通过 `HttpRequest.match_info()` 或者 `Path` 在 处理函数中使用
    * 提取出的参数会进行URL解码
    * 实现方式为将 `{path_name}` 转换为 `[^{}/]+` 这则表达式匹配
        * 因此 `{path_name}` 对应的部分至少应该有一个字符
    * 另外 `{path_name}` 支持 正则表达式 匹配，语法为 `{path_name:reg_exp}`

#### （4）生成URL

`HttpRequest.url_for("res_name", IntoIterator<Item = AsRef<str>>)`

* 内部资源
    * 如果这样配置 `web::resource("/test/{a}/{b}/{c}").name("foo")`
    * 且 `HttpRequest.url_for("foo", &["1", "2", "3"])`
    * 则 返回 `$protocol:://$host/test/1/2/3`
* 外部资源
    * 如果这样配置 `App::new().external_resource("youtube", "https://youtube.com/watch/{video_id}")`
    * 且 `let url = req.url_for("youtube", &["oHg5SJYRHA0"]).unwrap();`
    * 则 `assert_eq!(url.as_str(), "https://youtube.com/watch/oHg5SJYRHA0");`

#### （5）路径正则化

`App::new().wrap(actix_web::middleware::NormalizePath)` 用于合并多个 `/` 成一个（2.0不支持在最后添加`/`）

#### （6）Guard

支持通过请求头返回一个true或者false，来决定是否匹配，定义如下

```rust
pub trait Guard {
    fn check(&self, request: &RequestHead) -> bool;
}
```

Guard 还提供 `guard::Not` 等谓词来 连接多个 `guard`

#### （7）修改默认的 Not Found 返回

```rust
App::new()
    .service(web::resource("/").route(web::get().to(index)))
    .default_service(
        web::route()
            .guard(guard::Not(guard::Get()))
            .to(|| HttpResponse::MethodNotAllowed()),
    )
```

### 2、请求

Actix-web自动解压缩body。支持以下编解码器：

* Brotli
* Chunked
* Compress
* Gzip
* Deflate
* Identity
* Trailers
* EncodingExt

会根据 `Content-Encoding` 头选择相应的解码器，但是 Actix-web 不支持 多编码 例如 `Content-Encoding: br, gzip`

#### （1）JSON 请求

针对 Json 请求，支持如下几种读取方式

* 使用 `Json<T>` 提取器
* 手动加载 `web::Payload` 到内存，手动反序列化成对象
    * `web::Payload` 是反序列化后的字节流迭代器对象

```rust
use actix_web::{error, web, App, Error, HttpResponse};
use bytes::BytesMut;
use futures::StreamExt;
use serde::{Deserialize, Serialize};
use serde_json;

#[derive(Serialize, Deserialize)]
struct MyObj {
    name: String,
    number: i32,
}

const MAX_SIZE: usize = 262_144; // max payload size is 256k

async fn index_manual(mut payload: web::Payload) -> Result<HttpResponse, Error> {
    // payload is a stream of Bytes objects
    let mut body = BytesMut::new();
    while let Some(chunk) = payload.next().await {
        let chunk = chunk?;
        // limit max size of in-memory payload
        if (body.len() + chunk.len()) > MAX_SIZE {
            return Err(error::ErrorBadRequest("overflow"));
        }
        body.extend_from_slice(&chunk);
    }

    // body is loaded, now we can deserialize serde-json
    let obj = serde_json::from_slice::<MyObj>(&body)?;
    Ok(HttpResponse::Ok().json(obj)) // <- send response
}
```

#### （2）Multipart body

通过 [actix-multipart](https://crates.io/crates/actix-multipart) 提供多部分流支持 （例如 `multipart/form-data`）

#### （3）Urlencoded body

也就是 `application/x-www-form-urlencoded` 可以通过 `Form<T>` 解析

#### （4）Streaming request

通过 `web::Payload` 读取。例如打印请求体

```rust
use actix_web::{web, Error, HttpResponse};
use futures::StreamExt;

async fn index(mut body: web::Payload) -> Result<HttpResponse, Error> {
    let mut bytes = web::BytesMut::new();
    while let Some(item) = body.next().await {
        bytes.extend_from_slice(&item?);
    }

    println!("Chunk: {:?}", bytes);
    Ok(HttpResponse::Ok().finish())
}
```

### 3、响应 Response

#### （1）请求处理器返回响应的方式

* 创建 `Response` 方式，使用 `actix_web::HttpResponse::Ok()` 等系列快捷方法，该方法将返回一个 `actix_http::ResponseBuilder` 类型，可以方便的指定
    * `.content_type()`
    * `.header()`
    * `.body()`、`.finish()`、`json` 将返回 `HttpResponse`
* 返回实现 `Responder` 的类型

#### （2）编码

使用 `Compress middleware` 中间件 `https://docs.rs/actix-web/2/actix_web/middleware/struct.Compress.html`，可以支持如下响应体编码：

* `Brotli`
* `Gzip`
* `Deflate`
* `Identity`

使用方式

* 基本配置 `App::new().wrap(middleware::Compress::default())`，相当于 `ContentEncoding::Auto`，即相依根据请求Header的 `Accept-Encoding` 头确定（自动协商）
* 手动指定某个请求处理器的编码 `HttpResponse::Ok().encoding(ContentEncoding::Br)`
* 全局使用某个编码 ` App::new().wrap(middleware::Compress::new(ContentEncoding::Br))`
* 手动禁用某个请求处理器的压缩编码（比如返回内容本事就已经是压缩的，需要使用该配置，否则将压缩两次） `HttpResponse::Ok().encoding(ContentEncoding::Identity)` 例子：

```rust
use actix_web::{
    http::ContentEncoding, middleware, dev::BodyEncoding, HttpResponse,
};

static HELLO_WORLD: &[u8] = &[
    0x1f, 0x8b, 0x08, 0x00, 0xa2, 0x30, 0x10, 0x5c, 0x00, 0x03, 0xcb, 0x48, 0xcd, 0xc9,
    0xc9, 0x57, 0x28, 0xcf, 0x2f, 0xca, 0x49, 0xe1, 0x02, 0x00, 0x2d, 0x3b, 0x08, 0xaf,
    0x0c, 0x00, 0x00, 0x00,
];

async fn index() -> HttpResponse {
    HttpResponse::Ok()
        .encoding(ContentEncoding::Identity)
        .header("content-encoding", "gzip")
        .body(HELLO_WORLD)
}
```

#### （3）JSON响应

返回 `Json<T:Serialize>` 对象即可

### 4、测试

#### （1）单元测试

`actix_web::test::TestRequest` 用于方便创建 `HttpRequest`

* 常见的静态方法
    * `::with_header("content-type", "text/plain")`
    * `::with_hdr(h: Header)`
    * `::with_uri(path)`
    * `::get()`
    * `::post()`
    * `::put()` 等

```rust

#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::test;

    // 单元测试
    #[actix_rt::test]
    async fn test_index_ok(){
        // 构建测试的TestRequest
        let req = test::TestRequest::get()
            .header("content-type", "text/plain")
            // .get()
            .to_http_request();

        println!("{:?}", req);

        // 执行测试函数
        let resp = index().await.respond_to(&req).await.ok().unwrap();

        // 断言
        assert_eq!(resp.status(), http::StatusCode::OK);
    }
}
```

#### （2）集成测试

```rust
use actix_web::{error, web, http, App, FromRequest, HttpResponse, HttpServer, Responder, Error, HttpRequest, Either, Result};

use serde::{Serialize, Deserialize};
use futures::future::{ready, Ready};

// 自定义 Response
#[derive(Serialize, Deserialize)]
struct ResponseWrapper<T> {
    code: i32,
    msg: String,
    data: Option<T>,
}

// Responder
impl <T> Responder for ResponseWrapper<T> where T: Serialize {
    type Error = Error;
    type Future = Ready<Result<HttpResponse, Error>>;

    fn respond_to(self, _req: &HttpRequest) -> Self::Future {
        let body = serde_json::to_string(&self).unwrap();

        // Create response and set content type
        ready(Ok(HttpResponse::Ok()
            .content_type("application/json")
            .body(body)))
    }
}

// curl http://localhost:8088/responder/custom_responder
async fn responder_custom_responder() -> impl Responder {
    ResponseWrapper {
        code: 0,
        msg: "success".to_string(),
        data: Some("custom_responder".to_string()) }
}


#[cfg(test)]
mod tests {
    use super::*;
    use actix_web::test;

    // 集成测试
    #[actix_rt::test]
    async fn test_index_get() {
        let mut app = test::init_service(App::new().route("/", web::get().to(index))).await;
        let req = test::TestRequest::with_header("content-type", "text/plain").to_request();
        let resp = test::call_service(&mut app, req).await;
        assert!(resp.status().is_success());
    }

    #[actix_rt::test]
    async fn test_index_post() {
        let mut app = test::init_service(App::new().route("/", web::get().to(index))).await;
        let req = test::TestRequest::post().uri("/").to_request();
        let resp = test::call_service(&mut app, req).await;
        assert!(resp.status().is_client_error());
    }

    #[actix_rt::test]
    async fn test_json_response() {
        let mut app = test::init_service(
            App::new()
                .data(AppState { app_name: "app_name".to_string() })
                .route("/responder/custom_responder", web::get().to(responder_custom_responder)),
        ).await;
        let req = test::TestRequest::get().uri("/responder/custom_responder").to_request();
        let resp: ResponseWrapper<String> = test::read_response_json(&mut app, req).await;

        assert_eq!(resp.code, 0);
    }
}
```

#### （3）返回流测试

参见：https://actix.rs/docs/testing/

```rust
use std::task::Poll;
use bytes::Bytes;
use futures::stream::poll_fn;

use actix_web::http::{ContentEncoding, StatusCode};
use actix_web::{web, http, App, Error, HttpRequest, HttpResponse};

async fn sse(_req: HttpRequest) -> HttpResponse {
    let mut counter: usize = 5;

    // yields `data: N` where N in [5; 1]
    let server_events = poll_fn(move |_cx| -> Poll<Option<Result<Bytes, Error>>> {
        if counter == 0 {
            return Poll::Ready(None);
        }
        let payload = format!("data: {}\n\n", counter);
        counter -= 1;
        Poll::Ready(Some(Ok(Bytes::from(payload))))
    });

    HttpResponse::build(StatusCode::OK)
        .set_header(http::header::CONTENT_TYPE, "text/event-stream")
        .set_header(
            http::header::CONTENT_ENCODING,
            ContentEncoding::Identity.as_str(),
        )
        .streaming(server_events)
}

pub fn main() {
    App::new().route("/", web::get().to(sse));
}

#[cfg(test)]
mod tests {
    use super::*;
    use actix_rt;

    use futures_util::stream::StreamExt;
    use futures_util::stream::TryStreamExt;

    use actix_web::{test, web, App};

    #[actix_rt::test]
    async fn test_stream() {
        let mut app = test::init_service(App::new().route("/", web::get().to(sse))).await;
        let req = test::TestRequest::get().to_request();

        let mut resp = test::call_service(&mut app, req).await;
        assert!(resp.status().is_success());

        // first chunk
        let (bytes, mut resp) = resp.take_body().into_future().await;
        assert_eq!(bytes.unwrap().unwrap(), Bytes::from_static(b"data: 5\n\n"));

        // second chunk
        let (bytes, mut resp) = resp.take_body().into_future().await;
        assert_eq!(bytes.unwrap().unwrap(), Bytes::from_static(b"data: 4\n\n"));

        // remaining part
        let bytes = test::load_stream(resp.take_body().into_stream()).await;
        assert_eq!(bytes.unwrap(), Bytes::from_static(b"data: 3\n\ndata: 2\n\ndata: 1\n\n"));
    }
}
```

### 5、中间件

Actix-web 的中间件允许我们向请求/响应中添加其他行为。中间件可以挂接到传入的请求流程中，使我们能够修改请求以及暂停请求处理以尽早返回响应。

通常，中间件涉及以下操作：

* 请求之前附加逻辑
* 响应之后附加逻辑
* 修改Application State
* 访问外部服务（如Redis、日志记录、会话等）

中间件可以注册到 `App`、`Scope` 和 `Resource`。并以与注册相反的顺序执行。通常，一个中间件的实现需要实现 `Service` 特质 和 `Transform` 特质。这两个特质都有默认的方法实现（就是什么都不做）

#### （1）自定义实现中间件

工厂特质 `Transform` 用于创建一个 `Service` 类型，相当于如下函数

* `async fn new_transform<NextReq, NextRes, NextErr, Req, Res, Err, InitErr>(next_service: Service<NextReq, NextRes, NextErr>) -> Result<Service<Req, Res, Err>, InitErr>`

`Service` 类型（服务/中间件），相当于如下两个函数

* 一个异步处理函数 `async fn<Req, Res, Err>(req: Req) -> Result<Res, Err>`
* 一个异步就绪判断 `async fn poll_ready<Err>() -> Result<(), Err>`

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

use actix_service::{Service, Transform};
use actix_web::{dev::ServiceRequest, dev::ServiceResponse, Error};
use futures::future::{ok, Ready};
use std::future::Future;

async fn index() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

// 中间件处理分为两个步骤。
// 1. 中间件初始化，使用链中的下一个服务作为参数调用中间件工厂。
// 2. 中间件的call方法被普通请求调用。
pub struct SayHi;

// 中间件工厂需要实现 `Transform` 来自 `actix-service` crate
// Transform 特质相当于如下函数声明（忽略错误）
// type Service = async fn<Req, Res, Err>(req: Req) -> Result<Res, Err>
// async fn new_transform<NextReq, NextRes, NextErr, Req, Res, Err, InitErr>(next_service: Service<NextReq, NextRes, NextErr>) -> Result<Service<Req, Res, Err>, InitErr>

// `S` - 下一个 service 的 类型
// `B` - response body 的类型
impl<S, B> Transform<S> for SayHi
where
    S: Service<Request = ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    // 当前 Service 的请求
    type Request = ServiceRequest;
    // 当前 Service 的响应
    type Response = ServiceResponse<B>;
    // 当前 Service 的错误类型
    type Error = Error;
    // 创建 当前 Service 时可能出现的错误
    type InitError = ();
    // 当前 Transform 的类型
    type Transform = SayHiMiddleware<S>;
    // 异步的包装
    type Future = Ready<Result<Self::Transform, Self::InitError>>;

    // 工厂方法
    fn new_transform(&self, service: S) -> Self::Future {
        ok(SayHiMiddleware { service })
    }
}

pub struct SayHiMiddleware<S> {
    service: S,
}

// Service 中间件/服务，基本等价于
// 一个异步处理函数：async fn<Req, Res, Err>(req: Req) -> Result<Res, Err>
// 一个异步就绪判断：async fn poll_ready<Err>() -> Result<(), Err>
// 服务是 Actix-web 的核心抽象
impl<S, B> Service for SayHiMiddleware<S>
where
    S: Service<Request = ServiceRequest, Response = ServiceResponse<B>, Error = Error>,
    S::Future: 'static,
    B: 'static,
{
    type Request = ServiceRequest;
    type Response = ServiceResponse<B>;
    type Error = Error;
    type Future = Pin<Box<dyn Future<Output = Result<Self::Response, Self::Error>>>>;

    // 一个异步函数
    // 确定当前 Service 是否可以处理请求，不可以处理时，返回 Pending
    fn poll_ready(&mut self, cx: &mut Context<'_>) -> Poll<Result<(), Self::Error>> {
        self.service.poll_ready(cx)
    }

    // 处理函数，不应该调用 poll_ready。允许
    // actix可能在不调用poll_ready的情况下调用call，因此实现上必须要考虑这一点
    fn call(&mut self, req: ServiceRequest) -> Self::Future {
        println!("Hi from start. You requested: {}", req.path());

        let fut = self.service.call(req);

        Box::pin(async move {
            let res = fut.await?;

            println!("Hi from response");
            Ok(res)
        })
    }
}


#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(move || {
        App::new()
            .wrap(SayHi{})
            .route("/", web::get().to(index))
    }
}
```

#### （2）使用 `wrap_fn` 注册简单的中间件

```rust
use actix_service::Service;
use actix_web::{web, App};
use futures::future::FutureExt;

#[actix_rt::main]
async fn main() {
    let app = App::new()
        .wrap_fn(|req, srv| {
            println!("Hi from start. You requested: {}", req.path());
            srv.call(req).map(|res| {
                println!("Hi from response");
                res
            })
        })
        .wrap_fn(|req, srv| {
            println!("Hi from start. You requested: {}", req.path());
            let fut = srv.call(req);
            async {
                let res = fut.await;
                println!("Hi from response");
                res
            }
        })
        .route(
            "/index.html",
            web::get().to(|| async {
                "Hello, middleware!"
            }),
        );
}
```

#### （3）内置中间件——Logging

依赖

```rust
env_logger = "0.7"
```

基本使用

```rust
use actix_web::middleware::Logger;
use env_logger;

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{App, HttpServer};

    std::env::set_var("RUST_LOG", "actix_web=info");
    env_logger::init();

    HttpServer::new(|| {
        App::new()
            .wrap(Logger::default())
            .wrap(Logger::new("%a %{User-Agent}i"))
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

默认格式为：

```
%a %t "%r" %s %b "%{Referer}i" "%{User-Agent}i" %T
```

格式标志的含义

%% 百分号
%a IP地址 (如果是反向代理则是代理的IP地址)
%t 请求开始处理的时间
%P 为请求提供服务的孩子的进程ID
%r 请求的第一行，也就是 类是这种 "GET /app_state HTTP/1.1"
%s 响应吗
%b 响应大小（以字节为单位），包括HTTP标头
%T 服务请求所用的时间，以秒为单位，浮动分数为.06f格式
%D 服务请求所花费的时间（以毫秒为单位）
%{FOO}i `request.headers[‘FOO’]`
%{FOO}o `response.headers[‘FOO’]`
%{FOO}e `os.environ[‘FOO’]`

#### （4）内置中间件——Default headers

配置默认响应头

```rust
use actix_web::{http, middleware, HttpResponse};

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{web, App, HttpServer};

    HttpServer::new(|| {
        App::new()
            .wrap(middleware::DefaultHeaders::new().header("X-Version", "0.2"))
            .service(
                web::resource("/test")
                    .route(web::get().to(|| HttpResponse::Ok()))
                    .route(
                        web::method(http::Method::HEAD)
                            .to(|| HttpResponse::MethodNotAllowed()),
                    ),
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

#### （5）User sessions 中间件

Actix-web提供了会话管理的通用解决方案。actix-session中间件可以与不同的后端类型一起使用，以将会话数据存储在不同的后端中。

默认情况下仅实现基于 cookie 后端 的 Session 方案（数据存储在指定 Cookie 中），其他后端类型可以自定义实现

[基于 Cookie 后端](https://docs.rs/actix-session/0.3.0/actix_session/struct.CookieSession.html) 的后端实现为 `CookieSessionBackend`。该方式可以存储小于4000字节的数据

依赖库 `actix-session = "0.3"`

```rust
use actix_session::{CookieSession, Session};
use actix_web::{web, App, Error, HttpResponse, HttpServer};

async fn index(session: Session) -> Result<HttpResponse, Error> {
    // access session data
    if let Some(count) = session.get::<i32>("counter")? {
        session.set("counter", count + 1)?;
    } else {
        session.set("counter", 1)?;
    }

    Ok(HttpResponse::Ok().body(format!(
        "Count is {:?}!",
        session.get::<i32>("counter")?.unwrap()
    )))
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .wrap(
                CookieSession::signed(&[0; 32]) // <- create cookie based session middleware
                    .secure(false),
            )
            .service(web::resource("/").to(index))
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

#### （6）内置中间件——错误处理

`ErrorHandlers` 允许你为特定错误码添加处理程序

```rust
use actix_web::middleware::errhandlers::{ErrorHandlerResponse, ErrorHandlers};
use actix_web::{dev, http, HttpResponse, Result};

fn render_500<B>(mut res: dev::ServiceResponse<B>) -> Result<ErrorHandlerResponse<B>> {
    res.response_mut().headers_mut().insert(
        http::header::CONTENT_TYPE,
        http::HeaderValue::from_static("Error"),
    );
    Ok(ErrorHandlerResponse::Response(res))
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{web, App, HttpServer};

    HttpServer::new(|| {
        App::new()
            .wrap(
                ErrorHandlers::new()
                    .handler(http::StatusCode::INTERNAL_SERVER_ERROR, render_500),
            )
            .service(
                web::resource("/test")
                    .route(web::get().to(|| HttpResponse::Ok()))
                    .route(web::head().to(|| HttpResponse::MethodNotAllowed())),
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

### 5、静态文件

#### （1）单个文件

使用路由创建一个处理函数，返回一个 `NamedFile::open` 类型即可

```rust
use actix_files::NamedFile;
use actix_web::{HttpRequest, Result};
use std::path::PathBuf;

async fn index(req: HttpRequest) -> Result<NamedFile> {
    let path: PathBuf = req.match_info().query("filename").parse().unwrap();
    Ok(NamedFile::open(path)?)
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    use actix_web::{web, App, HttpServer};

    HttpServer::new(|| App::new().route("/{filename:.*}", web::get().to(index)))
        .bind("127.0.0.1:8088")?
        .run()
        .await
}
```

#### （2）目录

```rust
use actix_files as fs;
use actix_web::{App, HttpServer};

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().service(fs::Files::new("/static", ".").show_files_listing())
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

其他内容 参见 [API文档](https://docs.rs/actix-files/0.2.1/actix_files/)

## 四、协议

### 1、WebSocket

> 解析入门：https://www.cnblogs.com/chyingp/p/websocket-deep-in.html

Actix-web 通过 `actix-web-actors` crate 支持 WebSocket。可以使用 `web::Payload` 将请求的有效负载转换为 `ws::Message` 流，然后使用流组合器来处理实际消息，但是处理与http actor的websocket通信更简单。

```rust
use actix::{Actor, StreamHandler};
use actix_web::{web, App, Error, HttpRequest, HttpResponse, HttpServer};
use actix_web_actors::ws;

/// 定义Http Actor
struct MyWs;

impl Actor for MyWs {
    type Context = ws::WebsocketContext<Self>;
}

/// 一个 ws::Message 消息的 处理器
impl StreamHandler<Result<ws::Message, ws::ProtocolError>> for MyWs {
    fn handle(
        &mut self,
        msg: Result<ws::Message, ws::ProtocolError>,
        ctx: &mut Self::Context,
    ) {
        match msg {
            Ok(ws::Message::Ping(msg)) => ctx.pong(&msg),
            Ok(ws::Message::Text(text)) => ctx.text(text),
            Ok(ws::Message::Binary(bin)) => ctx.binary(bin),
            _ => (),
        }
    }
}

// ws://localhost:8088/ws/echo
/*
curl --include \
     --no-buffer \
     --header "Connection: Upgrade" \
     --header "Upgrade: websocket" \
     --header "Host: echo.websocket.org" \
     --header "Origin: https://echo.websocket.org" \
     --header "Sec-WebSocket-Key: NVwjmQUcWCenfWu98asDmg==" \
     --header "Sec-WebSocket-Version: 13" \
     http://localhost:8088/ws/echo
*/
async fn ws_echo(req: HttpRequest, stream: web::Payload) -> Result<HttpResponse, Error> {
    let resp = ws::start(MyWs {}, &req, stream);
    println!("{:?}", resp);
    resp
}


#[actix_rt::main]
async fn main() -> std::io::Result<()> {

    HttpServer::new(move || {

        App::new()
            .service(
                web::scope("/ws")
                    .route("/echo", web::get().to(ws_echo))
            )
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

例子

* [echo](https://github.com/actix/examples/tree/master/websocket/)
* [websocket-chat](https://github.com/actix/examples/tree/master/websocket-chat/)

### 2、HTTP2.0

HTTP 2.0 参见 [HTTP 协议连接技术演进](/posts/http-connection-evolution/)

https://actix.rs/docs/http2/

## 五、自动加载服务器

安装相关二进制工具

```bash
cargo install systemfd cargo-watch
```

添加依赖

```rust
[dependencies]
listenfd = "0.3
```

修改代码

```rust
use actix_web::{web, App, HttpRequest, HttpServer, Responder};
use listenfd::ListenFd;

async fn index(_req: HttpRequest) -> impl Responder {
    "Hello World!"
}

#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    let mut listenfd = ListenFd::from_env();
    let mut server = HttpServer::new(|| App::new().route("/", web::get().to(index)));

    server = if let Some(l) = listenfd.take_tcp_listener(0).unwrap() {
        server.listen(l)?
    } else {
        server.bind("127.0.0.1:3000")?
    };

    server.run().await
}
```

启动

```bash
systemfd --no-pid -s http::8088 -- cargo watch -x run
```

## 六、数据库

> 本部分前序文章： [rust-diesel](/posts/rust-diesel/)

核心API：

* `actix_web::block` 用于包装阻塞函数

以 `diesel` 为例

添加依赖 `Cargo.toml`

```toml
# 数据库
diesel = { version = "1.4", features = ["mysql", "r2d2", "chrono"] }
dotenv = "0.15.0"
# 时间日期
chrono = "0.4"
```

配置数据库连接 `.env`

```env
DATABASE_URL=mysql://root:123456@localhost/actix_learn
```

初始化测试表

```bash
diesel setup
diesel migration generate init
```

编辑数据表初始化文件 `migrations/2020-03-04-090155_init/up.sql`

```sql
-- Your SQL goes here
CREATE TABLE posts (
  id BIGINT PRIMARY KEY auto_increment comment 'ID',
  user_id BIGINT not null,
  title VARCHAR(256) NOT NULL comment '标题',
  body TEXT NOT NULL comment '内容',
  published BOOLEAN NOT NULL DEFAULT 0 comment '是否发布'
);

CREATE TABLE users (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  name TEXT NOT NULL,
  hair_color TEXT,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

`migrations/2020-03-04-090155_init/down.sql`

```sql
-- This file should undo anything in `up.sql`
DROP TABLE posts
DROP TABLE users;
```

执行数据库变更

```bash
diesel migration run
```

编写代码

`src/lib.rs`

```rust
#[macro_use]
extern crate diesel;
extern crate dotenv;

use diesel::prelude::*;
use dotenv::dotenv;
use diesel::r2d2;
use std::env;

pub mod schema;
pub mod model;

pub type PoolConnection = r2d2::Pool<r2d2::ConnectionManager<MysqlConnection>>;

pub fn new_connection_pool() ->  PoolConnection {
    dotenv().ok();

    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    let manager = r2d2::ConnectionManager::<MysqlConnection>::new(database_url);
    let pool = r2d2::Pool::builder()
        .max_size(15)
        .build(manager)
        .expect("Failed to create pool.");
    pool
}
```

`src/main.rs`

```rust
use actix_learn::*;
use diesel::prelude::*;

// curl http://localhost:8088/block/user/create
async fn create_user(pool: web::Data<PoolConnection>) -> String {
    let conn = pool.get().expect("couldn't get db connection from pool");

    let r = web::block(move ||  {
        use schema::users;
        let user = model::UserForInsert {
            name: "name".to_string(),
            hair_color: Some("blank".to_string()),
        };
        diesel::insert_into(users::table)
            .values(&user)
            .execute(&conn)
    }).await;
    if let Err(e) = r {
        String::from(format!("{:?}", e))
    } else {
        String::from("create_success")
    }
}
```

## 七、设计图

### 1、HTTP Server初始化

```rust
#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::to(|| HttpResponse::Ok()))
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

如上代码的 时序图 如下

![actix_web server 初始化](https://actix.rs/img/diagrams/http_server.svg)

### 2、连接生命周期

Server 开始侦听所有套接字后，Accept和Worker是两个主要循环，负责处理传入的客户端连接。

总体 时序图 如下

![总体时序图](https://actix.rs/img/diagrams/connection_overview.svg)

Accept 循环细节 如下

![connection_accept](https://actix.rs/img/diagrams/connection_accept.svg)

Worker 循环细节 如下

![connection_worker](https://actix.rs/img/diagrams/connection_worker.svg)

Request 循环

![connection_request](https://actix.rs/img/diagrams/connection_request.svg)

## 八、Actor 系统（Actix）

> 参考：
>
> * [Actix 文档](https://actix.rs/book/actix/)
> * [例子](https://github.com/actix/actix/tree/master/examples/)

### 1、Getting Started

添加依赖 `Cargo.toml`

```toml
[dependencies]
actix = "0.9"
futures = "0.3"
```

`src/bin/actor_ping.rs`

```rust
extern crate actix;
use actix::prelude::*;

// 测试 Actor 对象
struct MyActor {
    count: usize,
}

// 所有的 Actor 必须实现 Actor 特质
impl Actor for MyActor {
    // 每个 Actor 都有一个执行上下文
    type Context = Context<Self>;
}

// Actor 接收的参数
struct Ping(usize);

// Actor 接收的参数必须实现 Message 对象
impl Message for Ping {
    // 定义 该参数的 返回值类型
    type Result = usize;
}

// Actor 的具体处理函数
impl Handler<Ping> for MyActor {
    // 返回值
    type Result = usize;

    fn handle(&mut self, msg: Ping, _ctx: &mut Context<Self>) -> Self::Result {
        self.count += msg.0;

        self.count
    }
}

#[actix_rt::main]
async fn main() {
    // 启动一个Actor，返回一个 Addr<MyActor>
    let addr = MyActor { count: 10 }.start();

    // 发送消息并获取 Future 结果
    let res = addr.send(Ping(10)).await;

    // handle() returns tokio handle
    // 返回一个结果
    println!("RESULT: {}", res.unwrap() == 20);

    // stop system and exit
    System::current().stop();
}
```

运行：`cargo run --bin actor_ping`

### 2、Actor

Actix 是一个 基于 [Actor 模型](https://en.wikipedia.org/wiki/Actor_model) 的 Rust 库

该模型允许将应用程序编写为一组通过消息进行通信的独立执行但相互协作的 Actor 。Actor 是封装状态和行为并在actix库提供的Actor系统中运行的对象。

Actor在特定的执行上下文 `Context<A>` 中运行。上下文对象仅在执行期间可用。每个参与者都有一个单独的执行上下文。执行上下文还控制 Actor 的生命周期。

Actor 仅通过交换消息进行通信。发送方可以选择等待响应。Actor 不是直接引用的，而是通过地址引用的。

任何类型的都可以是一个 Actor，它只需要实现 Actor 特质即可。

为了能够处理特定的消息，Actor 必须为此消息提供 `Handler<M>` 实现。所有消息都是静态类型的。消息可以异步方式处理。Actor可以产生其他actor或将Future或stream添加到执行上下文。Actor特征提供了几种方法，可以控制Actor的生命周期。

#### （1）Actor 生命周期

**Started**

Actor 以 Started 状态开始。在此状态下，将调用 Actor 的 `started()` 方法。Actor 特质提供了此方法的默认实现。actor上下文在此状态下可用，并且 actor 可以启动更多 actor 或注册异步流或进行任何其他所需的配置。

**Running**

Actor 在 `started()` 方法被调用后处于的状态，Actor可以一致保持运行状态。

**Stopping**

在以下情况下，Actor的执行状态变为停止状态：

* `Context::stop` 被自己调用
* Actor 的所有地址都会被丢弃。即没有其他演员引用它
* 在上下文中没有注册任何事件对象。

通过创建新地址或添加事件对象，并返回 `Running::Continue` actor可以从停止状态恢复到运行状态。

如果一个 Actor 由于调用 `Context::stop()` 而将状态更改为停止，则上下文会立即停止处理传入消息并调用 `Actor::stopping()` 。如果参与者没有恢复到运行状态，则会丢弃所有未处理的消息。默认情况下，此方法返回 `Running::Stop` ，以确认停止操作。

**Stopped**

如果参与者在 `Stopping` 状态期间未修改执行上下文，则参与者状态将变为 `Stopped`。该状态被认为是最终状态，此时 Actor 将被 Drop

#### （2）Message

Actor 通过发送消息与其他 Actor 进行通信。在 actix 中的所有消息都是有类型的。消息需要实现 `Message` 特质。`Message::Result` 定义返回类型。让我们定义一个简单的Ping消息-接受此消息的actor需要返回 `io::Result<bool>`。

```rust
use std::io;
use actix::prelude::*;

struct Ping;

impl Message for Ping {
    type Result = Result<bool, io::Error>;
}
```

#### （3）产生一个Actor

如何启动 Actor 取决于其上下文。通过 Actor 特性的启动和创建方法可以生成新的异步actor。它提供了多种创建参与者的方法。有关详细信息，参见下文。

#### （4）用MessageResponse作为Actor的返回值

```rust
pub trait MessageResponse<A: Actor, M: Message> {
    fn handle<R: ResponseChannel<M>>(self, ctx: &mut A::Context, tx: Option<R>);
}
```

`src/bin/actor_ping2.rs`

```rust
extern crate actix;
use actix::prelude::*;
use actix::dev::{MessageResponse, ResponseChannel};

// 测试 Actor 对象
struct MyActor {
    count: usize,
}

// 所有的 Actor 必须实现 Actor 特质
impl Actor for MyActor {
    // 每个 Actor 都有一个执行上下文
    type Context = Context<Self>;
}

// Actor 接收的参数
struct Ping(usize);

// Actor 接收的参数必须实现 Message 对象
impl Message for Ping {
    // 定义 该参数的 返回值类型
    type Result = Pong;
}

// 这是返回值
#[derive(Debug)]
struct Pong(bool);

// 返回值必须实现 MessageResponse 特质
impl<A, M> MessageResponse<A, M> for Pong
where
    A: Actor,
    M: Message<Result = Pong>,
{
    // 将 返回值 发送出去的逻辑
    fn handle<R: ResponseChannel<M>>(self, _: &mut A::Context, tx: Option<R>) {
        if let Some(tx) = tx {
            tx.send(self);
        }
    }
}

// Actor 的具体处理函数
impl Handler<Ping> for MyActor {
    // 返回值
    type Result = Pong;

    fn handle(&mut self, msg: Ping, _ctx: &mut Context<Self>) -> Self::Result {
        self.count += msg.0;

        // self.count
        Pong(true)
    }
}

#[actix_rt::main]
async fn main() {
    // 启动一个Actor，返回一个 Addr<MyActor>
    let addr = MyActor { count: 10 }.start();

    // 发送消息并获取 Future 结果
    let res = addr.send(Ping(10)).await;

    // handle() returns tokio handle
    // 返回一个结果
    println!("RESULT: {:?}", res);

    // stop system and exit
    System::current().stop();
}
```

运行：`cargo run --bin actor_ping2`

### 3、地址

Actor 仅通过交换消息进行通信。发送方可以选择等待响应。无法仅通过其地址直接引用Actor。

有几种获取 Actor 地址的方法。Actor 特征提供了两种启动 Actor 的辅助方法。两者都返回 `started` 角色的地址。

* `MyActor.start();`
* `let addr = ctx.address();`

#### （1）消息

要将消息发送给参与者，需要使用Addr对象。Addr提供了几种发送消息的方法。

* `Addr::do_send(M) -> ()` 此方法将忽略邮件发送中的任何错误。如果邮箱已满，则邮件仍在排队，绕过限制。如果参与者的邮箱已关闭，则该消息将被静默丢弃。此方法不会返回结果，因此，如果邮箱已关闭并且发生故障，则不会有任何指示。
* `Addr::try_send(M) -> Result<(), SendError> ` 此方法尝试立即发送消息。如果邮箱已满或关闭（角色已死），则此方法返回 `SendError`。
* `Addr::send(M) -> Future<Result<M::Result, MailboxError>>` 该消息返回一个将来的对象，该对象解析为消息处理过程的结果。如果丢弃返回的Future对象，则该消息将被取消

#### （2）收件人（Recipient）

一种特殊的 `Addr`，是 `Addr` 的一种 `Clone`。一般作为 Actor 结构体的成员，用于 Actor 的组合

参见：https://github.com/actix/actix/blob/master/examples/ring.rs

### 4、Content

Actor 维护一个内部执行上下文或状态。这样，参与者可以确定自己的地址，更改邮箱限制或停止执行。

* 设置邮箱容量 `ctx.set_mailbox_capacity(1);`
* 获取自身地址 `ctx.address()`
* 停止 Actor `ctx.stop()`

#### （1）Mailbox

所有消息都首先发送到 Actor 的邮箱，然后 Actor 的执行上下文调用特定的消息处理程序。邮箱通常是有界的。该能力特定于上下文实现。对于Context类型，默认情况下，容量设置为16条消息，可以使用 `Context::set_mailbox_capacity()` 增加容量。

```rust
struct MyActor;
impl Actor for MyActor {
    type Context = Context<Self>;

    fn started(&mut self, ctx: &mut Self::Context) {
        ctx.set_mailbox_capacity(1);
    }
}

let addr = MyActor.start();
```

对 `Addr::do_send(M)` 或完全绕过邮箱的 `AsyncContext::notify(M)` 和 `AsyncContext::notify_later(M, Duration)` 无效。

#### （2）获取Actor地址

Actor 可以从上下文中查看自己的地址。

```rust

struct MyActor;

struct WhoAmI;

impl Message for WhoAmI {
    type Result = Result<actix::Addr<MyActor>, ()>;
}

impl Actor for MyActor {
    type Context = Context<Self>;
}

impl Handler<WhoAmI> for MyActor {
    type Result = Result<actix::Addr<MyActor>, ()>;

    fn handle(&mut self, msg: WhoAmI, ctx: &mut Context<Self>) -> Self::Result {
        Ok(ctx.address())
    }
}

let who_addr = addr.do_send(WhoAmI {} );
```

#### （3）停止Actor

在参与者执行上下文中，您可以选择阻止参与者处理任何将来的邮箱消息。这可能是对错误情况的响应，或者是程序关闭的一部分。为此，请调用 `Context::stop()`。

```rust
impl Handler<Ping> for MyActor {
    type Result = usize;

    fn handle(&mut self, msg: Ping, ctx: &mut Context<Self>) -> Self::Result {
        self.count += msg.0;

        if self.count > 5 {
            println!("Shutting down ping receiver.");
            ctx.stop()
        }

        self.count
    }
}
```

### 5、Arbiter

即 Actor 异步执行器，运行在 tokio （`rt-core`单线程）之上。

`src/bin/actor_arbiter.rs`

```rust
extern crate actix;
extern crate futures;
use actix::prelude::*;
use futures::FutureExt;
use futures::TryFutureExt;

struct SumActor {}

impl Actor for SumActor {
    type Context = Context<Self>;
}

struct Value(usize, usize);

impl Message for Value {
    type Result = usize;
}

impl Handler<Value> for SumActor {
    type Result = usize;

    fn handle(&mut self, msg: Value, _ctx: &mut Context<Self>) -> Self::Result {
        msg.0 + msg.1
    }
}

struct DisplayActor {}

impl Actor for DisplayActor {
    type Context = Context<Self>;
}

struct Display(usize);

impl Message for Display {
    type Result = ();
}

impl Handler<Display> for DisplayActor {
    type Result = ();

    fn handle(&mut self, msg: Display, _ctx: &mut Context<Self>) -> Self::Result {
        println!("Got {:?}", msg.0);
    }
}

fn main() {
    let system = System::new("single-arbiter-example");

    // 创建 Addr
    let sum_addr = SumActor {}.start();
    let dis_addr = DisplayActor {}.start();

    // 定义一个执行流的Future
    // 起初发送 `Value(6, 7)` 给 `SumActor`
    // `Addr::send` 返回 `Request` 类型，该类型实现了 `Future`
    // Future::Output = Result<usize, MailboxError>
    let execution = sum_addr
        .send(Value(6, 7))
        // `.map_err` 转换 `Future<usize, MailboxError>` 为 `Future<usize, ()>`
        //   如果有错误将打印错误信息
        // 实现来自于 use futures::TryFutureExt;
        .map_err(|e| {
            eprintln!("Encountered mailbox error: {:?}", e);
        })
        // 假设发送成功，并成功返回，and_then将得到执行，其中参数为上一个Future的Result<T, E> 的 T
        // 实现来自于 use futures::TryFutureExt;
        .and_then(move |res| {
            // `res` 是 `SumActor` 参数为 `Value(6, 7)` 的返回值，类型为 `usize`

            // res 发送给 DisplayActor 展示
            dis_addr.send(Display(res)).map_err(|_| ())
        })
        .map(move |_| {
            // 当 DisplayActor 返回后停止，将关闭所有 Actor
            System::current().stop();
        });

    // 提交 Future 到 Arbiter/event 循环中
    Arbiter::spawn(execution);

    system.run().unwrap();
}
```

### 6、SyncArbiter

```rust
use actix::prelude::*;

struct MySyncActor;

impl Actor for MySyncActor {
    type Context = SyncContext<Self>;
}


fn main() {

    // 同步执行器
    // 创建运行在指定线程中的Actor
    let addr = SyncArbiter::start(2, || MySyncActor);
}
```

Sync Actor没有邮箱限制，但您仍应正常使用do_send，try_send并发送以解决其他可能的错误或同步与异步行为。
