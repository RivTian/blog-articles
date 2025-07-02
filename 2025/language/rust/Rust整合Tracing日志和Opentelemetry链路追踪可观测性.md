## racing是什么

`tracing`不仅可以用于`Web`日志还可以用于`lib`库的日志，它也不仅仅是一个日志库，它还支持以下功能

- 日志
- 函数和代码块执行时间测量
- 分布式系统中的链路追踪耗时分析
- 记录函数调用参数和返回值
- 追踪代码执行路径和逻辑流程
- 监控系统运行状态和资源使用情况
- 业务流程可视化和分析

## tracing核心概念

- Spans

  ：跨度，跨度表示程序在特定上下文中执行的

  时间段

  ，当程序进入某一个服务中的上下文执行任务时进入跨度，停止执行时退出跨度；线程当前执行的范围称为当前线程的跨度

  - 代码指向范围追踪：追踪代码何时开始执行、何时结束执行、状态变化
  - 收集关联信息：日志详细记录（业务具体步骤、中间状态）、耗时信息
  - 结构化观测数据：通过`span`可以构建出整个业务流程的调用链路

- **Event**：事件，某一个时间点发生了什么事

- **Collector**：收集器，当`span`开始/结束或`Event`发生时，他们的记录会被`Collector`收集，`tracing-subscriber`就是一个`Collector`

## tracing相关的crate

- [tracing-appender](https://crates.io/crates/tracing-appender)：提供了一个执行非阻塞写入的订阅者
- [tracing-futures](https://crates.io/crates/tracing-futures)：提供与 async/await 的兼容性
- [tracing-subscriber](https://crates.io/crates/tracing-subscriber)：提供了一些辅助函数来构建订阅者
- [tracing-bunyan-formatter](https://crates.io/crates/tracing-bunyan-formatter)：将Bunyan格式的日志转换为JSON
- [tracing-log](https://crates.io/crates/tracing-log)：日志处理库，可以将日志转发给订阅者
- [tracing-opentelemetry](https://crates.io/crates/tracing-opentelemetry)：提供了一个订阅者，用于将跟踪发送到与 OpenTelemetry 兼容的分布式跟踪系统
- [opentelemetry_otlp](https://docs.rs/opentelemetry-otlp/latest/opentelemetry_otlp/)：导出日志、指标和跟踪 格式添加到 OpenTelemetry 收集器或其他兼容的后端
- [tracing-timing](https://crates.io/crates/tracing-timing) ：在tracing之上实现事件间计时指标。 它提供了一个订阅者，用于记录事件对之间经过的时间并生成直方图

## tracing 分布式链路追踪日志

通常，我们会将应用分为多个服务部署到多台服务器上，一旦其中一台服务器发生问题，查看日志非常麻烦，通过tracing将所有日志集中管理，无需SSH登录到每个节点去查看日志

日志应该包含哪些信息：

```sh
 时间 严重级别 请求ID 用户ID 应用ID ...
```

## Web应用使用tracing

添加`crate`

```toml
 [dependencies]
# 在函数定义前添加 #[tracing::instrument]，当函数被调用时，tracing 库会自动记录函数的进入、退出情况以及执行时长等信息，并且可以携带函数的参数等作为额外的上下文信息
tracing = { version = "0.1.40", features = ["attributes", "instrument"] }
# 将 Rust 的错误类型与 Tracing 的 span 相关联的库
# 当出现错误时，通常希望能将错误的发生与当时的执行上下文（也就是 tracing 所记录的 span 相关信息）结合起来，以便更好地理解错误产生的背景和原因
tracing-error = "0.2.0"
# 配置和管理 tracing 所产生的日志和追踪数据的收集、格式化以及输出等操作
tracing-subscriber = { version = "0.3", default-features = true, features = [
    "std",
    "env-filter",
    "registry",
    "local-time",
    "fmt",
] }
# 处理日志数据的写入目标和写入方式
tracing-appender = "0.2.3"
# 用于将 Rust 标准库中的 log 宏记录的日志与 tracing 库的日志记录机制进行集成，使得使用 log 宏编写的旧有日志记录代码能够无缝地与基于 tracing 的新日志系统协同工作，方便在项目迁移或者同时使用两种日志记录方式的场景下进行统一管理和输出
tracing-log = "0.2.0"
# 用于将 tracing 库产生的日志按照 Bunyan 格式进行格式化的库
tracing-bunyan-formatter = "0.3.9"
# 读取env添加到到环境变量
dotenvy = "0.15.7"
```

### 日志管理

1、`src`目录同级创建`.env`文件记录日志级别

```env
# 这里设置info级别
RUST_LOG=info
```

或在`.env`文件指定级别`RUST_LOG=trace cargo run`

> `ERROR`、`WARN`、`DEBUG`、`INFO`、`TRACE`或者1-5，级别从低到高

2、`src/config/tracing.rs`初始化日志和订阅者，这里使用格式化层为日志添加了人类易读的颜色，并设置了每日日志（每天在`src`同级生成一个`logs`文件夹，在日志文件末尾以日期为后缀）

```rust
use std::error::Error;
use tracing::{ subscriber, Subscriber };
use tracing_appender::{ non_blocking::WorkerGuard, rolling::{ RollingFileAppender, Rotation } };
use tracing_log::LogTracer;
use tracing_subscriber::{ fmt::{ self, MakeWriter }, layer::SubscriberExt, EnvFilter, Registry };
use tracing_bunyan_formatter::{ BunyanFormattingLayer, JsonStorageLayer };
// 创建订阅者
fn create_subscriber<W>(
    name: &str,
    env_filter: EnvFilter,
    writer: W
) -> impl Subscriber + Sync + Send
    where W: for<'a> MakeWriter<'a> + Send + Sync + 'static
{
    // 创建格式化层
    let fmt_layer = fmt::Layer
        ::default()
        // 显示日志来源的目标信息
        .with_target(true)
        // 显示线程 ID
        .with_thread_ids(true)
        // 线程名称信息
        .with_thread_names(true)
        // 启用 ANSI 转义码来支持彩色输出
        .with_ansi(true)
        .compact();
    // 注册订阅者
    Registry::default()
        .with(env_filter)
        .with(fmt_layer)
        // 以 JSON 格式进行处理
        .with(JsonStorageLayer)
        // 以文本格式进行输出到控制台
        .with(BunyanFormattingLayer::new(name.into(), std::io::stdout))
        // 以文本格式进行输出到文件
        .with(BunyanFormattingLayer::new(name.into(), writer))
}
// 初始化订阅者
pub fn init_subscriber<S>(subscriber: S) -> Result<(), Box<dyn Error>>
    where S: Subscriber + Send + Sync + 'static
{
    // 初始化日志
    LogTracer::init()?;
    // 设置全局默认订阅者
    subscriber::set_global_default(subscriber)?;
    Ok(())
}
// 初始化并返回文件句柄
pub fn init() -> Result<WorkerGuard, Box<dyn Error>> {
    // 构建每日日志，前缀为app.log
    let file_appender = RollingFileAppender::new(Rotation::DAILY, "logs", "app.log");
    // 构建非阻塞日志
    let (file_appender, file_appender_guard) = tracing_appender::non_blocking(file_appender);
    // 初始化订阅者，从环境变量设置日志级别
    init_subscriber(create_subscriber("app", EnvFilter::from_default_env(), file_appender))?;
    Ok(file_appender_guard)
}
```

3、使用

```rust
use axum::{ response::Html, routing::get, Router };
use tracing_log::log::info;
mod config{
    pub mod tracing;
}
#[tokio::main]
async fn main() {
    // 加载.env 环境配置文件，成功返回包含的值，失败返回None
    // 没有环境变量无法设置日志级别，如果发现info!没有出书就是这个原因
    dotenvy::dotenv().ok();
    // 注册tracing
    let file_appender_guard = config::tracing::init();
    // 创建路由
    let app = Router::new().route("/", get(handler));
    info!("App Router Create Successfully");
    // 创建
    let listener = tokio::net::TcpListener::bind("127.0.0.1:40000").await.unwrap();
    info!("listening on {}", listener.local_addr().unwrap());
    axum::serve(listener, app).await.unwrap();
    // 在日志文件关闭之前不退出，确保日志被完全记录
    drop(file_appender_guard);
}

async fn handler() -> Html<&'static str> {
    Html("<h1>Hello, World!</h1>")
}
```

4、控制台日志效果

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/bf0bc49f4fdc4b048c8121c937edd2dc~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=Uxcde4iKlryguV26tyetVsaT7Us%3D)

文件日志效果

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4159dda2fe784ff7a0e1e498ed87f48d~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=o9L3QdiLstIGaGwg%2FcBc2XCRIhc%3D)

## 日志信息

```json
{
"v":日志格式版本,"name":"应用或服务名称","msg":"日志消息主主体信息","level":日志级别,"hostname":"主机名","pid":进程ID,"time":"日志记录时间（按照UTC记录）",
"target":"日志来源（代码模块）","line":日志来源在代码中的行号,"file":日志来源的源文件名,
"log.module_path":"日志关联的模块路径","log.line":代码行号,"log.target":"日志来源模块","log.file":"记录日志的源文件路径"
}
```

## 使用链路追踪

### 创建示例项目

`src/main.rs`

```rust
use tracing::info;
use axum::{ routing::{ get, patch }, Router };
// 注意这里的嵌套关系
mod config {
    pub mod tracing;
}
mod handler {
    pub mod todo_handler;
}
mod dto;
use handler::todo_handler::{ todos_create, todos_delete, todos_index, todos_update, Db };
#[tokio::main]
async fn main() {
    // 加载.env 环境配置文件，成功返回包含的值，失败返回None
    // 没有环境变量无法设置日志级别，如果发现info!没有出书就是这个原因
    dotenvy::dotenv().ok();
    // 注册tracing
    let file_appender_guard = config::tracing::init();
    // 模拟数据库
    let db = Db::default();
    // 创建路由
    let app = Router::new()
        .route("/todos", get(todos_index).post(todos_create))
        .route("/todos/:id", patch(todos_update).delete(todos_delete))
        .with_state(db);
    info!("App Router Create Successfully");
    // 创建服务监听器
    let listener = tokio::net::TcpListener::bind("127.0.0.1:8080").await.unwrap();
    info!("listening on {}", listener.local_addr().unwrap());
    axum::serve(listener, app).await.unwrap();
    // 在日志文件关闭之前不退出，确保日志被完全记录
    drop(file_appender_guard);
}

```

`src\config\tracing.rs` 这里实现了Todo的增删改查

```rust
use std::error::Error;
use tracing::{ subscriber, Subscriber };
use tracing_appender::{ non_blocking::WorkerGuard, rolling::{ RollingFileAppender, Rotation } };
use tracing_log::LogTracer;
use tracing_subscriber::{ fmt::{ self, MakeWriter }, layer::SubscriberExt, EnvFilter, Registry };
use tracing_bunyan_formatter::{ BunyanFormattingLayer, JsonStorageLayer };
// 创建订阅者
fn create_subscriber<W>(
    name: &str,
    env_filter: EnvFilter,
    writer: W
) -> impl Subscriber + Sync + Send
    where W: for<'a> MakeWriter<'a> + Send + Sync + 'static
{
    // 创建格式化层
    let fmt_layer = fmt::Layer
        ::default()
        // 显示日志来源的目标信息
        .with_target(true)
        // 显示线程 ID
        .with_thread_ids(true)
        // 线程名称信息
        .with_thread_names(true)
        // 启用 ANSI 转义码来支持彩色输出
        .with_ansi(true)
        .compact();
    // 注册订阅者
    Registry::default()
        .with(env_filter)
        .with(fmt_layer)
        // 以 JSON 格式进行处理
        .with(JsonStorageLayer)
        // 以文本格式进行输出到控制台
        .with(BunyanFormattingLayer::new(name.into(), std::io::stdout))
        // 以文本格式进行输出到文件
        .with(BunyanFormattingLayer::new(name.into(), writer))
}
// 初始化订阅者
pub fn init_subscriber<S>(subscriber: S) -> Result<(), Box<dyn Error>>
    where S: Subscriber + Send + Sync + 'static
{
    // 初始化日志
    LogTracer::init()?;
    // 设置全局默认订阅者
    subscriber::set_global_default(subscriber)?;
    Ok(())
}
// 初始化并返回文件句柄
pub fn init() -> Result<WorkerGuard, Box<dyn Error>> {
    // 构建每日日志，前缀为app.log
    let file_appender = RollingFileAppender::new(Rotation::DAILY, "logs", "app.log");
    // 构建非阻塞日志
    let (file_appender, file_appender_guard) = tracing_appender::non_blocking(file_appender);
    // 初始化订阅者，从环境变量设置日志级别
    init_subscriber(create_subscriber("app", EnvFilter::from_default_env(), file_appender))?;
    Ok(file_appender_guard)
}
```

`src\dto\mod.rs` 定义请求和响应的DTO

```rust
use serde::{ Deserialize, Serialize };
use uuid::Uuid;
// 更新请求DTO
#[derive(Debug, Deserialize)]
pub struct UpdateTodo {
    pub text: Option<String>,
    pub completed: Option<bool>,
}
// 创建请求DTO
#[derive(Debug, Deserialize)]
pub struct CreateTodo {
    pub text: String,
}
// 查询请求DTO
#[derive(Debug, Deserialize, Default)]
pub struct Pagination {
    pub offset: Option<usize>,
    pub limit: Option<usize>,
}
// 响应DTO
#[derive(Debug, Serialize, Clone)]
pub struct Todo {
    pub id: Uuid,
    pub text: String,
    pub completed: bool,
}
```

`src\handler\todo_handler.rs` 处理业务（真实业务不在这里做，应当再加一层`service`，这里仅为了展示`tracing`用法）

```rust
use crate::dto::{CreateTodo, Pagination, Todo, UpdateTodo};
use axum::{
    extract::{Path, Query, State},
    http::StatusCode,
    response::IntoResponse,
    Json,
};
use std::{
    collections::HashMap,
    sync::{Arc, RwLock},
};
use tracing::{event, instrument, span, Level};
use uuid::Uuid;
// 查询todo
pub async fn todos_index(
    pagination: Option<Query<Pagination>>,
    State(db): State<Db>,
) -> impl IntoResponse {
    let todos = db.read().unwrap();

    let Query(pagination) = pagination.unwrap_or_default();

    let todos = todos
        .values()
        .skip(pagination.offset.unwrap_or(0))
        .take(pagination.limit.unwrap_or(usize::MAX))
        .cloned()
        .collect::<Vec<_>>();

    Json(todos)
}
// 创建todo
pub async fn todos_create(
    State(db): State<Db>,
    Json(input): Json<CreateTodo>,
) -> impl IntoResponse {
    let todo = Todo {
        id: Uuid::new_v4(),
        text: input.text,
        completed: false,
    };
    db.write().unwrap().insert(todo.id, todo.clone());
    (StatusCode::CREATED, Json(todo))
}
// 更新todo
pub async fn todos_update(
    Path(id): Path<Uuid>,
    State(db): State<Db>,
    Json(input): Json<UpdateTodo>,
) -> Result<impl IntoResponse, StatusCode> {
    let mut todo = db
        .read()
        .unwrap()
        .get(&id)
        .cloned()
        .ok_or(StatusCode::NOT_FOUND)?;

    if let Some(text) = input.text {
        todo.text = text;
    }

    if let Some(completed) = input.completed {
        todo.completed = completed;
    }

    db.write().unwrap().insert(todo.id, todo.clone());

    Ok(Json(todo))
}
// 删除todo
pub async fn todos_delete(Path(id): Path<Uuid>, State(db): State<Db>) -> impl IntoResponse {
    if db.write().unwrap().remove(&id).is_some() {
        StatusCode::NO_CONTENT
    } else {
        StatusCode::NOT_FOUND
    }
}
// 模拟数据库
pub type Db = Arc<RwLock<HashMap<Uuid, Todo>>>;
```

运行此项目你应当能看到以下日志

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/d986fd6998534f1fb1042b880dbce7ca~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=n7mYjmyWGrFDKPXZOVFxL4vbiyQ%3D)

### 自定义创建span

方式一：使用`span!`宏创建`span`

```rust
let span = span!(Level::TRACE, "这是一个span，指定span的级别为trace");
// 忽略后离开作用域被drop()
let _enter = span.enter();
```

例如

```rust
pub async fn todos_create(
    State(db): State<Db>,
    Json(input): Json<CreateTodo>
) -> impl IntoResponse {
    // 定义一个span，指定span的级别为trace，指定span的名字，这是最低的级别
    let span = span!(Level::TRACE, "这是一个span, 指定span的级别为trace");
    // span开始位置
    let _enter = span.enter();
    let todo = Todo {
        id: Uuid::new_v4(),
        text: input.text,
        completed: false,
    };
    db.write().unwrap().insert(todo.id, todo.clone());
    (StatusCode::CREATED, Json(todo))
} // drop(enter);

```

方式二：创建`span`并手动设置跨度范围

```rust
pub async fn todos_create(
    State(db): State<Db>,
    Json(input): Json<CreateTodo>
) -> impl IntoResponse {
    // 定义一个span，指定span的级别为trace，指定span的名字，这是最低的级别
    let span = span!(Level::TRACE, "这是一个span, 指定span的级别为trace");
    // span开始位置
    let enter = span.enter();
    let todo = Todo {
        id: Uuid::new_v4(),
        text: input.text,
        completed: false,
    };
    db.write().unwrap().insert(todo.id, todo.clone());
    // 手动结束span
    drop(enter);
    (StatusCode::CREATED, Json(todo))
}
```

方式三：使用`in_scope`闭包创建`span`

> 此案例无法使用，由于Json离开作用域会被Drop，使用案例请见以下面的：`其他Rust应用使用tracing`

```rust
span.in_scope(|| {})
```

方式四：使用` #[instrument]`创建`span`

> 见下文

创建`span`后还需要添加日志，当代码运行到这个`span`就会将日志收集到`Collector`收集器，当你

### #[instrument]宏创建span

**#[instrument]** 将函数标记为`span`，默认创建`INFO`级别的`span3`，`tracing`会自动为函数创建一个 `span`即代码执行范围，`span`名跟函数名相同，用于跟踪和记录函数执行相关的信息，比如耗时等），并且会在函数进入和退出时自动记录对应的事件到日志中，方便追踪函数的执行情况

`tracing::Level`指定跨度的级别

> level：`ERROR`、`WARN`、`DEBUG`、`INFO`、`TRACE`或者1-5，级别从低到高

```rust
// 创建todo
#[instrument]
pub async fn todos_create(
    State(db): State<Db>,
    Json(input): Json<CreateTodo>
) -> impl IntoResponse {
    error!("error! 创建todo成功");
    warn!("warn! 创建todo成功");
    debug!("debug! 创建todo成功");
    info!("info! 创建todo成功");
    trace!("trace! 创建todo成功");
    let todo = Todo {
        id: Uuid::new_v4(),
        text: input.text,
        completed: false,
    };
    db.write().unwrap().insert(todo.id, todo.clone());
    event!(Level::INFO, "创建todo成功");
    (StatusCode::CREATED, Json(todo))
}
```

当你以以下内容请求`http://127.0.0.1:8080/todos`时

```json
{
    "text":"测试"
}
```

即可看到日志输出

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/50dc5ad4dc8c49c3813d741f9ccc9b5c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=oP6XH2DVXeHqN7N7m3YlrVO%2BDqM%3D)

`name`覆盖默认创建的`span`名字

```rust
 #[instrument(name = "my_name")]
 async fn hello()-> String{
     info!("hello tracing");
     "hello".to_string()
 }
```

- `target`：覆盖生成的跨度的目标
- `parent`：覆盖生成的跨度的父级
- `follows_from`：覆盖生成的跨度跟随关系
- `skip`：跳过记录参数
- `fields`：向跨度添加其他上下文
- `ret`：函数返回时发出带有函数返回值的事件
- `err`：覆盖事件的级别，事件的级别默认为`ERROR`

```sh
 #[instrument(err(level = Level::INFO))]
```

## Opentelemetry可观测性

可观测性：

- `Logs`：日志

- ```
  Traces
  ```

  ：链路追踪，请求在分布式系统中的整个路径

  - `span`包含的数据：追踪的标识符，跨度的标识符，开始/结束时间戳，键值元数据，事件

- `Metrics`：指标信息，CPU、内存、请求计数等信息

## Opentelemetry可视化

修改`src/handler/todo_handler.rs`为每个方法创建span

```rust
#[instrument]
pub async fn todos_index(
    pagination: Option<Query<Pagination>>,
    State(db): State<Db>
) -> impl IntoResponse {
    let todos = db.read().unwrap();

    let Query(pagination) = pagination.unwrap_or_default();

    let todos = todos
        .values()
        .skip(pagination.offset.unwrap_or(0))
        .take(pagination.limit.unwrap_or(usize::MAX))
        .cloned()
        .collect::<Vec<_>>();

    Json(todos)
}
// 创建todo
#[instrument]
pub async fn todos_create(
    State(db): State<Db>,
    Json(input): Json<CreateTodo>
) -> impl IntoResponse {
    error!("error! 创建todo成功");
    warn!("warn! 创建todo成功");
    debug!("debug! 创建todo成功");
    info!("info! 创建todo成功");
    trace!("trace! 创建todo成功");
    let todo = Todo {
        id: Uuid::new_v4(),
        text: input.text,
        completed: false,
    };
    db.write().unwrap().insert(todo.id, todo.clone());
    event!(Level::INFO, "创建todo成功");
    (StatusCode::CREATED, Json(todo))
}
// 更新todo
#[instrument]
pub async fn todos_update(
    Path(id): Path<Uuid>,
    State(db): State<Db>,
    Json(input): Json<UpdateTodo>
) -> Result<impl IntoResponse, StatusCode> {
    let mut todo = db.read().unwrap().get(&id).cloned().ok_or(StatusCode::NOT_FOUND)?;
    info!("更新todo成功");
    if let Some(text) = input.text {
        todo.text = text;
    }

    if let Some(completed) = input.completed {
        todo.completed = completed;
    }

    db.write().unwrap().insert(todo.id, todo.clone());

    Ok(Json(todo))
}
// 删除todo
#[instrument]
pub async fn todos_delete(Path(id): Path<Uuid>, State(db): State<Db>) -> impl IntoResponse {
    if db.write().unwrap().remove(&id).is_some() {
        StatusCode::NO_CONTENT
    } else {
        StatusCode::NOT_FOUND
    }
}
// 模拟数据库
pub type Db = Arc<RwLock<HashMap<Uuid, Todo>>>;
```

添加依赖

```toml
# web框架
axum = "0.7.7"
rand = "0.8.5"
# 序列化与反序列化
serde = { version = "1.0.127", features = ["derive"] }
# 异步运行时
tokio = { version = "1.40.0", features = ["full"] }
uuid = { version = "1.11.0", features = ["v4", "serde"] }
# 在函数定义前添加 #[tracing::instrument]，当函数被调用时，tracing 库会自动记录函数的进入、退出情况以及执行时长等信息，并且可以携带函数的参数等作为额外的上下文信息
tracing = { version = "0.1.40", features = ["attributes"] }
# 将 Rust 的错误类型与 Tracing 的 span 相关联的库
# 当出现错误时，通常希望能将错误的发生与当时的执行上下文（也就是 tracing 所记录的 span 相关信息）结合起来，以便更好地理解错误产生的背景和原因
tracing-error = "0.2.0"
# 配置和管理 tracing 所产生的日志和追踪数据的收集、格式化以及输出等操作
tracing-subscriber = { version = "0.3", default-features = true, features = [
    "std",
    "env-filter",
    "registry",
    "local-time",
    "fmt",
] }
# 处理日志数据的写入目标和写入方式
tracing-appender = "0.2.3"
# 用于将 Rust 标准库中的 log 宏记录的日志与 tracing 库的日志记录机制进行集成，使得使用 log 宏编写的旧有日志记录代码能够无缝地与基于 tracing 的新日志系统协同工作，方便在项目迁移或者同时使用两种日志记录方式的场景下进行统一管理和输出
tracing-log = "0.2.0"
# 用于将 tracing 库产生的日志按照 Bunyan 格式进行格式化的库
tracing-bunyan-formatter = "0.3.10"
# 读取env到环境变量
dotenvy = "0.15.7"
# 链路追踪
tracing-opentelemetry = "0.28.0"
opentelemetry = "0.27.1"
opentelemetry-otlp = { version = "0.27.0", features = [
    "trace",
    "metrics",
    "grpc-tonic",
] }
opentelemetry_sdk = { version = "0.27.1", features = ["rt-tokio"] }
anyhow = "1.0.68"
```

修改`src/config/tracing.rs`，注意每个版本API不一样（仔细看文档提供了哪些方法）

```rust
use opentelemetry_otlp::WithExportConfig;
use tracing::{ subscriber, Subscriber };
use tracing_appender::{ non_blocking::WorkerGuard, rolling::{ RollingFileAppender, Rotation } };
use tracing_log::LogTracer;
use tracing_opentelemetry::OpenTelemetryLayer;
use tracing_subscriber::{ fmt::{ self, MakeWriter }, layer::SubscriberExt, EnvFilter, Registry };
use tracing_bunyan_formatter::{ BunyanFormattingLayer, JsonStorageLayer };
use opentelemetry::KeyValue;
use opentelemetry_sdk::Resource;
use std::time::Duration;
use opentelemetry::trace::TracerProvider;
use std::error::Error;
// 创建订阅者
fn create_subscriber<W>(
    name: &str,
    env_filter: EnvFilter,
    writer: W
) -> impl Subscriber + Sync + Send
    where W: for<'a> MakeWriter<'a> + Send + Sync + 'static
{
    // 跨度导出器
    let exporter = opentelemetry_otlp::SpanExporter
        ::builder()
        .with_tonic()
        .with_endpoint("http://localhost:4317")
        .with_timeout(Duration::from_secs(3))
        .build()
        .expect("OTLP exporter failed");
    // 追踪导出器
    let tracer_provider = opentelemetry_sdk::trace::TracerProvider
        ::builder()
        .with_batch_exporter(exporter, opentelemetry_sdk::runtime::Tokio)
        .with_resource(Resource::new(vec![KeyValue::new("service.name", name.to_string())]))
        .build();
    // 创建追踪器对象
    let tracer = tracer_provider.tracer(name.to_string());
    // 创建格式化层
    let fmt_layer = fmt::Layer
        ::default()
        // 显示日志来源的目标信息
        .with_target(true)
        // 显示线程 ID
        .with_thread_ids(true)
        // 线程名称信息
        .with_thread_names(true)
        // 启用 ANSI 转义码来支持彩色输出
        .with_ansi(true)
        .compact();
    // 注册订阅者
    Registry::default()
        .with(env_filter)
        .with(fmt_layer)
        .with(OpenTelemetryLayer::new(tracer))
        // 以 JSON 格式进行处理
        .with(JsonStorageLayer)
        // 以文本格式进行输出到控制台
        .with(BunyanFormattingLayer::new(name.into(), std::io::stdout))
        // 以文本格式进行输出到文件
        .with(BunyanFormattingLayer::new(name.into(), writer))
}
// 初始化订阅者
pub fn init_subscriber<S>(subscriber: S) -> anyhow::Result<()>
    where S: Subscriber + Send + Sync + 'static
{
    LogTracer::init()?;
    subscriber::set_global_default(subscriber)?;
    Ok(())
}
// 初始化并返回文件句柄
pub fn init() -> Result<WorkerGuard, Box<dyn Error>> {
    // 构建每日日志，前缀为app.log
    let file_appender = RollingFileAppender::new(Rotation::DAILY, "logs", "app.log");
    // 构建非阻塞日志
    let (file_appender, file_appender_guard) = tracing_appender::non_blocking(file_appender);
    // 初始化订阅者，从环境变量设置日志级别
    init_subscriber(create_subscriber("app", EnvFilter::from_default_env(), file_appender))?;
    Ok(file_appender_guard)
}
```

部署`jaegertracing`

```sh
docker run -d -p4317:4317 -p16686:16686 jaegertracing/all-in-one:latest
```

访问:`localhost:16686`，使用APifox发起创建todo请求即可看到追踪内容

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/1a1e7089d03a490d9c7e148748906274~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=1s3HFwOumeU29dLRGELs5bKGLKI%3D)

可以看到日志也被收集到了这里

![](https://p9-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/4941910a977746ceae51cb5eb7550f5c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgY2Np:q75.awebp?rk3s=f64ab15b&x-expires=1750496922&x-signature=2YGL0yjCpNe1pmOh4ZEifm832nA%3D)