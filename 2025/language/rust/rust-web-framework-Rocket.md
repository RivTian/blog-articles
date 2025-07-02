> version: 0.4

参考：

* https://rocket.rs/

## 一、介绍

Rust 的 web 框架，Rocket的设计围绕着三个核心理念：

* 安全性，正确性和开发体验最重要
* All request handling information should be typed and self-contained.
* Decisions should not be forced.
    * 模板、序列化、session管理是可插拔的插件

总的来说

* Rocket 是一个一站式的非响应式传统 Web 框架
    * rust 社区排名第一的 web 框架
    * 核心路由通过宏实现
    * 一站式模板、序列化反序列化、ORM数据库连接通过扩展库可选实现
    * 必须依赖 nightly 版本 rust 工具包 （下一个版本将支持在 stable 中编译）

## 二、Getting Started

### 1、安装 Rust

参见 [Rust语言](/posts/rust语言/)

注意：Rocket 依赖一些实验特性，必须使用 `nightly` 版本

```bash
rustup show # 查看当前环境的 rust 版本
rustup default nightly # 系统全局 rust 版本配置成 nightly
rustup default stable # 系统全局 rust 版本配置成 stable （恢复）
rustup override set nightly # 当前目录 rust 版本配置成 nightly （推荐）
```

Rocket 依赖最新版的 rust nightly，如果发现无法编译请使用如下命令更新

```bash
# 更新
rustup update && cargo update
```

### 2、Hello world

新建项目

```bash
cargo new rocket-learn --bin
cd rocket-learn
```

添加依赖 `Cargo.toml`

```toml
[dependencies]
rocket = "0.4.2"
```

编写代码

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;

#[get("/")] // <- route 属性
fn index() -> &'static str { // <- request 处理函数
    "Hello, world!"
}

fn main() {
    rocket::ignite().mount("/", routes![index]).launch();
}
```

访问： http://localhost:8000

关闭 输出日志的 表情和颜色：配置环境变量 `ROCKET_CLI_COLORS` 为 `0`、`off`

```bash
ROCKET_CLI_COLORS=off cargo run
```

## 三、核心功能

### 1、Requests

`route` 等相关宏和函数的参数列表共同决定了队请求的映射和解析

基本语法如下：

```rust
#[get("/world")]
fn handler() { .. }
```

Rocket 提供的宏，除了像上面通过路径匹配处理函数之外，还按照如下类型进行校验和映射

* 动态路径片段（路径属性）的类型
* 多个路径片段（路径属性）的类型
* Body 类型
* 查询字符串，表单和表单值的类型
* 请求的预期传入或传出格式
* 任何用户定义的安全性或验证策略

Rocket 的 `route` 相关宏 的代码生成器负责生成相关的代码

#### （1）HTTP方法

* 方式1：使用内置的相关类属性宏
    * `get`
    * `put`
    * `post`
    * `delete`
    * `head`
    * `patch`
    * `options`
* 方式2：使用 `route` 并明确指定，例如：`#[route(GET, path = "/")]`

参见 [rocket_codegen文档](https://api.rocket.rs/v0.4/rocket_codegen/attr.route.html)

**HEAD 方法**

> HEAD 方法： HEAD方法跟GET方法相同，只不过服务器响应时不会返回消息体。

当存在匹配的GET路由时，Rocket会自动处理HEAD请求。它通过从响应中删除主体（如果有）来实现。您还可以通过声明HEAD请求的路由来专门处理HEAD请求。Rocket不会干扰您的应用程序明确处理的HEAD请求。

**HTTP 方法重新诠释**

当 `HTTP_Method=POST` 且 `Content-Type: application/x-www-form-urlencoded` 时，且**第一个参数**名为 `_method` 的参数值为 合法的 HTTP Method值时（比如 `PUT`、`DELETE`）时。将会映射到 `#[put]` 或 `#[delete]`。

这么设计的原因是：兼容原生HTML Form表单，其只支持 `POST` 和 `GET` 方法。

例子如下

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;

// post form 模拟 put: curl -d "_method=PUT" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:8000/methods
// post form 模拟 put: curl -d "_method=PUT&a=1" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:8000/requests/methods
// post form 模拟 put: curl -d "a=1&_method=PUT" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:8000/requests/methods
#[put("/methods")]
fn methods() -> &'static str {
    "put"
}


fn main() {
    rocket::ignite()
        .mount("/", routes![methods])
        .launch();
}

```

#### （2）动态路径（路径参数）

rocket 支持通过 `<field_name>` 方式捕获映射路径参数

```rust
use rocket::http::RawStr;

// curl http://localhost:8000/requests/dynamic-paths/this.is.name
// curl http://localhost:8000/requests/dynamic-paths/%e4%b8%ad%e6%96%87
#[get("/dynamic-paths/<name>")]
fn dynamic_paths(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![dynamic_paths])
        .launch();
}
```

`#[get("/dynamic-paths/<name>")]` 中的 `<name>` 声明了一个动态路径段（路径参数）。该参数的类型为处理函数的同名参数类型，本例中为：`name: &RawStr` 类型为 `RawStr`

* Rocket 支持任意数目的动态路径段
* 其对应的函数处理函数参数必须实现 `FromParam` 特质
* Rocket 为标准库中的许多类型实现了 `FromParam`
* 关于 `FromParam` 更多参见 [FromParam API docs](https://api.rocket.rs/v0.4/rocket/request/trait.FromParam.html)

```rust
// curl http://localhost:8000/requests/dynamic-paths/std-from-param/%e4%b8%ad%e6%96%87/1/true
#[get("/dynamic-paths/std-from-param/<name>/<age>/<cool>")]
fn std_from_param(name: String, age: u8, cool: bool) -> String {
    if cool {
        format!("You're a cool {} year old, {}!", age, name)
    } else {
        format!("{}, we need to talk about your coolness.", name)
    }
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![std_from_param])
        .launch();
}
```

RawStr 和 String 等 区别

* `RawStr` 是未经处理的原始字符串（UrlEncode编码，也就是说中文是不能直接使用的）
* `String` 是经过解码的字符串，类似的还有个 `&str`、 `Cow<str>`

**多片段（Multiple Segments）**

使用 `<param..>` 与创建多片段，显然此语法后不允许出现任何字符，其对应的参数必须实现 `FromSegments` 特质

```rust
use std::path::PathBuf;

// curl http://localhost:8000/requests/dynamic-paths/multiple-segments/%e4%b8%ad%e6%96%87/1/true
// curl http://localhost:8000/requests/dynamic-paths/multiple-segments/../1/true # 404
#[get("/dynamic-paths/multiple-segments/<path..>")]
fn multiple_segments(path: PathBuf) -> String {
    if let Some(p) = path.to_str() {
        String::from(p)
    } else {
        String::from("None")
    }
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![multiple_segments])
        .launch();
}
```

* 可以看出Rocket 为 `PathBuf` 实现的 `FromSegments` 考虑到了 `../` 攻击
* 以下四行即可实现一个简单的静态文件服务器（推荐使用 [rocket_contrib](https://api.rocket.rs/v0.4/rocket_contrib/) 的 [StaticFiles](https://api.rocket.rs/v0.4/rocket_contrib/serve/struct.StaticFiles.html) ，`rocket.mount("/public", StaticFiles::from("/static"))`）

```rust
#[get("/<file..>")]
fn files(file: PathBuf) -> Option<NamedFile> {
    NamedFile::open(Path::new("static/").join(file)).ok()
}
```

#### （3）路由匹配规则（Forwarding）

* 请求会和每个路由匹配，直到找到一个匹配的为止
* Rocket 的路由匹配顺序存在一个优先级数值，数值越小优先级越高
    * 如果优先级数值用户不配置Rocket会根据规则配置一个（从-6到-1）
    * 用户自定义优先级，例如 `#[get("/user/<id>", rank = 2)]`
    * 当出现两个路由都可以匹配同一个请求，不配置 `rank` 将报错
* 可以使用 `Result` 或者 `Option` 类型捕获类型失配错误
    * `id: Option<i32>` 如果 id 不是 i32 类型，`id == None`
    * `id: Result<i32, &RawStr>` 如果 id 不是 i32 类型，`id == Err(RawStr(id的值))`

```rust
// curl http://localhost:8000/requests/forwarding/rank/1
#[get("/forwarding/rank/<id>")]
fn rank_default(id: usize) -> String {
    String::from(format!("[rank] id={}", id))
}

#[get("/forwarding/rank/<id>", rank = 2)]
fn rank_2(id: isize) -> String {
    String::from(format!("[rank_2] id={}", id))
}

// curl http://localhost:8000/requests/forwarding/rank/%e4%b8%ad%e6%96%87
#[get("/forwarding/rank/<id>", rank = 3)]
fn rank_3(id: &RawStr) -> String {
    String::from(format!("[rank_3] id={}", id.url_decode().unwrap()))

// curl http://localhost:8000/requests/forwarding/option/1
// curl http://localhost:8000/requests/forwarding/option/string
#[get("/forwarding/option/<id>")]
fn option_param(id: Option<i32>) -> String {
    if let Some(v) = id {
        String::from(format!("param match id={}", v))
    } else {
        String::from("param not match")
    }
}

// curl http://localhost:8000/requests/forwarding/result/1
// curl http://localhost:8000/requests/forwarding/result/string
#[get("/forwarding/result/<id>")]
fn result_param(id: Result<i32, &RawStr>) -> String {
    if let Ok(v) = id {
        String::from(format!("param match id={}", v))
    } else if let Err(v) = id {
        String::from(format!("param not match id={}", v.url_decode().unwrap()))
    } else {
        String::from("dead code")
    }
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![rank_default, rank_2, rank_3])
        .mount("/requests", routes![option_param, result_param])
        .launch();
}
```

**默认优先级规则如下**

|静态路径| query 查询字符串 | 优先级 | 例子 |
|-------|-------|------|--------|---------|
|yes    |partly static | -6 | `/hello?world=true` |
|yes    |fully dynamic | -5 | `/hello/?<world>` |
|yes    |none | -4 | `/hello` |
|no     |partly static | -3 | `/<hi>?world=true` |
|no     |fully dynamic | -2 | `/<hi>?<world>` |
|no     |none | -1 | `/<hi>` |

#### （4）查询字符串（Query Strings）

类似于动态路径，查询查询字符串支持 `<field>` 语法。例如 `hello?wave&<name>`

* `wave` 不允许有值，否则报错
* `name` 和 `wave` 顺序不限制
* 允许存在其他字段

查询字符串对应的类型必须实现 `FromFormValue` 特质

```rust
// curl http://localhost:8000/requests/query-string/hello?wave # 404
// curl "http://localhost:8000/requests/query-string/hello?wave&name=xiaoming"
// curl "http://localhost:8000/requests/query-string/hello?wave=1&name=xiaoming" # 404
// curl "http://localhost:8000/requests/query-string/hello?name=xiaoming&wave"
// curl "http://localhost:8000/requests/query-string/hello?name=xiaoming&wave&age=10"
#[get("/query-string/hello?wave&<name>")]
fn query_string_hello(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![query_string_hello])
        .launch();
}

```

**可选参数**

使用函数声明使用 `Option<String>` （如果像`?wave`的将构造 `Some("")` 而不是 `None`）

```rust
// curl "http://localhost:8000/requests/query-string/option?name=xiaoming"
// curl "http://localhost:8000/requests/query-string/option?wave&name=xiaoming"
// curl "http://localhost:8000/requests/query-string/option?wave=1&name=xiaoming"
// curl "http://localhost:8000/requests/query-string/option?name=xiaoming&wave"
// curl "http://localhost:8000/requests/query-string/option?name=xiaoming&wave=10"
#[get("/query-string/option?<wave>&<name>")]
#[allow(unused_variables)]
fn query_string_option(wave: Option<String>, name: &RawStr) -> String {
    format!("Hello, wave={:?}, name={}!", wave, name.as_str())
}


fn main() {
    rocket::ignite()
        .mount("/requests", routes![query_string_option])
        .launch();
}
```

**多片段（Multiple Segments）**

使用 `<param..>` 与创建多片段，显然此语法后不允许出现任何字符，其对应的参数必须实现 `FromQuery` 特质。

除此之外，大多数情况，还可以使用 `Form` 对象或者 `LenientForm` 对象。

```rust
// curl "http://localhost:8000/requests/query-string/multiple-segments?name=xiaoming&account=10&id=1"
#[get("/query-string/multiple-segments?<id>&<user..>")]
fn query_string_multiple_segments(id: usize, user: Form<User>) -> String{
    format!("id={}, user={:?}", id, user.0)
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![query_string_multiple_segments])
        .launch();
}
```

更多例子参见：https://github.com/SergioBenitez/Rocket/tree/v0.4/examples/query_params

#### （5）请求守卫（Request Guards）

处理 `<name>` 这种路径参数外，Rocket支持通过自定义类型实现 `FromRequest` 的方式进行路由匹配，参数提取到路由函数，在Rocket中称为请求守卫

以上的路径参数就可以理解为一种内置的请求守卫。

```rust
use rocket::request::Outcome;
use rocket::Request;
use rocket::request::FromRequest;

struct ApiKey {
    key: String
}

impl<'a, 'r>  FromRequest<'a, 'r>  for ApiKey {
    type Error = String;

    fn from_request(request: &'a Request<'r>) -> Outcome<Self, Self::Error> {
        if let Some(key) = request.headers().get_one("Authorization") {
            Outcome::Success(ApiKey { key: String::from(key) })
        } else {
            Outcome::Forward(())  // 没有的话继续匹配下一个
        }
    }
}

//  curl -H "Authorization: 123 " "http://localhost:8000/requests/request-guards/custom/api-key"
//  curl -H "authorization: 123 " "http://localhost:8000/requests/request-guards/custom/api-key"
#[get("/request-guards/custom/api-key")]
fn request_guards_custom_api_key1(key: ApiKey) -> String {
    format!("api-key={}", key.key)
}

// curl "http://localhost:8000/requests/request-guards/custom/api-key"
#[get("/request-guards/custom/api-key", rank = 1)]
fn request_guards_custom_api_key2() -> String {
    format!("No Authorization")
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![request_guards_custom_api_key1, request_guards_custom_api_key2])
        .launch();
}

```

`FromRequest` 特质

* 定义了一个函数 `from_request`，接收 `Request` 参数，返回 `Outcome`
* `Outcome` 枚举包含三个值
    * `Success` 成功匹配成功
    * `Failure` 匹配失败，并直接返回，不继续进行匹配
    * `Forward` 交友下一个路由处理

#### （6）Cookie

```rust
use rocket::http::{RawStr, Cookies, Cookie};

// curl -i -H "Cookie: message=messageCookieValue" "http://localhost:8000/requests/cookies"
// curl -i -H "Cookie: k=v" "http://localhost:8000/requests/cookies"
// curl -i "http://localhost:8000/requests/cookies"
#[get("/cookies")]
fn request_cookies(mut cookies: Cookies) -> String {
    // 公有cookies
    cookies.add(Cookie::new("pub_key", "value"));
    cookies.remove(Cookie::named("pub_key2"));
    // 私有cookies 生产环境应该配置 secret_key 可以通过 openssl rand -base64 32 生成
    // 依赖 ring 库
    cookies.add_private(Cookie::new("pri_key", "value"));
    cookies.remove_private(Cookie::named("pri_key2"));
    cookies.get("message")
        .map(|value| format!("Message: {}", value))
        .unwrap_or("No message cookie".to_string())
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![request_cookies])
        .launch();
}
```

当Cookie和请求守卫共同使用时，Cookie应该在最后一个参数

```rust
// 报错
#[get("/")]
fn bad(cookies: Cookies, custom: Custom) { .. }

// 正确
#[get("/")]
fn good(custom: Custom, cookies: Cookies) { .. }
```

#### （7）Format

**Format**

```rust
#[post("/user", format = "application/json", data = "<user>")]
fn new_user(user: Json<User>) -> T { ... }
```

* 当 http 方法为 `HEAD` `OPTIONS` `GET` 时，将匹配 `format` 是否与 请求头的 `Accept` 一致
* 否则 将匹配 `format` 与请求头 `Content-Type`
* 本例中将匹配 `Content-Type: application/json`
* 更多参见：format支持的可选值参见https://api.rocket.rs/v0.4/rocket/http/struct.ContentType.html#method.parse_flexible
* 另外 json 等常用格式支持 `format = "json"` 这种的简写

#### （8）请求体匹配和读取

基本语法

```rust
#[post("/", data = "<input>")]
fn new(input: T) -> String { ... }
```

其中 `T` 必须实现 [`FromData`](https://api.rocket.rs/v0.4/rocket/data/trait.FromData.html) 类型

**Forms 类型**

```rust
use rocket::request::LenientForm;


#[derive(FromForm, Debug)]
struct Task {
    // 字段重命名
    #[form(field = "complete")]
    complete: bool,
    description: String,
}

// curl -d "complete=true&description=description" -H "Content-Type: application/x-www-form-urlencoded" -X POST http://localhost:8000/requests/body-data/form
// curl -d "complete=true&description=description" -X POST http://localhost:8000/requests/body-data/form
// curl -d "description=description" -X POST http://localhost:8000/requests/body-data/form
// curl -d "complete=1&description=description" -X POST http://localhost:8000/requests/body-data/form
// curl -d "a=1&complete=true&description=description" -X POST http://localhost:8000/requests/body-data/form
// 如果解析失败将返回 400 - Bad Request or 422 - Unprocessable Entity
// 默认情况下会解析是严格模式：不允许多或少参数
#[post("/body-data/form", data = "<task>")]
fn body_data_form(task: Form<Task>) -> String {
    format!("{:?}", task.0)
}

// curl -d "a=1&complete=true&description=description" -X POST http://localhost:8000/requests/body-data/lenient-form
// 宽容模式
#[post("/body-data/lenient-form", data = "<task>")]
fn body_data_lenient_form(task: LenientForm<Task>) -> String {
    format!("{:?}", task.0)
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![body_data_form, body_data_lenient_form])
        .launch();
}
```

* 参数校验，自定义类型实现 `FromFormValue` 特质，更多参见：https://rocket.rs/v0.4/guide/requests/#field-validation

**JSON**

依赖 `Cargo.toml`

```toml
[dependencies]
rocket = "0.4.2"
rocket_contrib = "0.4.2"
serde = "1.0"
serde_json = "1.0"
serde_derive = "1.0"
```

```rust
#![feature(proc_macro_hygiene, decl_macro)]

#[macro_use] extern crate rocket;
// #[macro_use] extern crate rocket_contrib;
#[macro_use] extern crate serde_derive;


use rocket_contrib::json::Json;

// 引入 rocket_contrib serde serde_json serde_derive 依赖
#[derive(Deserialize, Debug)]
struct Task2 {
    description: String,
    complete: bool
}

// curl -d '{"complete": true, "description": "description"}' -X POST http://localhost:8000/requests/body-data/json
// curl -d '{"a": 1, "complete": true, "description": "description"}' -X POST http://localhost:8000/requests/body-data/json
// curl -d '{"complete1": true, "description": "description"}' -X POST http://localhost:8000/requests/body-data/json
// 默认是宽松模式
#[post("/body-data/json", data = "<task>")]
fn body_data_json(task: Json<Task2>) -> String {
    format!("{:?}", task.0)
}

fn main() {
    rocket::ignite()
        .mount("/requests", routes![body_data_json])
        .launch();
}
```

**Streaming**

一般用于上传文件

```rust
use rocket::Data;

#[post("/upload", format = "plain", data = "<data>")]
fn request_upload(data: Data) -> std::io::Result<String> {
    // 需要防止DoS攻击
    data.stream_to_file("/tmp/upload.txt").map(|n| n.to_string())
}


fn main() {
    rocket::ignite()
        .mount("/requests", routes![request_upload])
        .launch();
}
```

* 更多参见：https://rocket.rs/v0.4/guide/requests/#streaming

#### （9）错误捕捉

```rust
#[catch(404)]
fn not_found(req: &Request) -> String {
    format!("Sorry, '{}' is not a valid path.", req.uri())
}

fn main() {
    rocket::ignite()
        .register(catchers![not_found])
        .launch();
}
```

* 更多参见：https://github.com/SergioBenitez/Rocket/tree/v0.4/examples/errors
* [reuqest](https://api.rocket.rs/v0.4/rocket/struct.Request.html)

### 2、Response

#### （1）基本情况

请求处理器的返回值必须返回 实现了 [`Responder`](https://api.rocket.rs/v0.4/rocket/response/trait.Responder.html) 类型的值。Rocket 为常用标准库类型实现了该特质：

* `&str`
* `String`
* `&[u8]`
* `Vec<u8>`
* `File`
* `()`
* `Option<T>`
* `Result<T, E> where E: Debug` 返回 `Err` 将在控制台简单打印
* `Result<T, E> where E: Debug + Responder` 返回 `Err` 将调用 其 `Responder` 的 `respond_to` 方法

同时 Rocket 提供了几个常用的 `Responder` 包装器类型，用于设置请求头和确定返回方式（这些类型均支持嵌套）

* [content](https://api.rocket.rs/v0.4/rocket/response/content/) - 自定义 Content-Type 响应头.
    * Json 仅设置响应头，不会有而外操作
    * Xml
    * MsgPack
    * Html
    * Plain
    * Css
    * JavaScript
* NamedFile - Streams a file to the client; automatically sets the Content-Type based on the file's extension.
* Redirect - Redirects the client to a different URI.
* Stream - Streams a response to a client from an arbitrary Reader type.
* [status](https://api.rocket.rs/v0.4/rocket/response/status/) - 修改状态码
    * Accepted 202
    * BadRequest 400
    * Created 201
    * Custom 自定义
    * NotFound 404
* Flash - Sets a "flash" cookie that is removed when accessed.
* Json - Automatically serializes values into JSON.
* MsgPack - Automatically serializes values into MessagePack.
* Template - Renders a dynamic template using handlebars or Tera.

`Responder` 特质定义了一个函数，`fn respond_to(self, request: &Request) -> response::Result<'r>;`。原理显而易见。

```rust
use rocket::response::{content, status};

// curl -i http://localhost:8000/responses/status/accepted
#[get("/status/accepted")]
fn response_status_accepted() -> status::Accepted<String> {
    status::Accepted(Some("status::Accepted".to_string()))
}

// curl -i http://localhost:8000/responses/content/json
#[get("/content/json")]
fn response_content_json() -> content::Json<&'static str> {
    content::Json("{ 'hi': 'world' }")
}

fn main() {
    rocket::ignite()
        .mount("/responses", routes![response_content_json, response_status_accepted])
        .launch();
}
```

#### （2）通过派生实现 Responder

Rocket 提供了派生宏用于快速创建自定义 Responder

```rust
#[derive(Responder)]
#[response(status = 500, content_type = "json")]
struct MyResponder {
    inner: OtherResponder,
    header: SomeHeader,
    more: YetAnotherHeader,
    #[response(ignore)]
    unrelated: MyType,
}
```

在这个例子中

* status 为 500，返回类型为 json
* `inner` 为相应体的内容
* `header` 和 `unrelated` 将附加到请求头

更多参见：https://api.rocket.rs/v0.4/rocket_codegen/derive.Responder.html

#### （3）自定义 Responder

参见 标准库 对 基础数据类型的实现

```rust
impl Responder<'static> for String {
    fn respond_to(self, _: &Request) -> Result<Response<'static>, Status> {
        Response::build()
            .header(ContentType::Plain)
            .sized_body(Cursor::new(self))
            .ok()
    }
}
```

#### （4）常见 Responder 之 Stream

为了防止内存占用过大，建议使用 Stream

```rust
#[get("/stream")]
fn stream() -> io::Result<Stream<UnixStream>> { // io::Result 本质上也是Result
    UnixStream::connect("/path/to/my/socket").map(|s| Stream::from(s))
}
```

#### （5）常见 Responder 之 Json

依赖 `Cargo.toml`

```toml
[dependencies]
rocket = "0.4.2"
rocket_contrib = "0.4.2"
serde = "1.0"
serde_json = "1.0"
serde_derive = "1.0"
```

主代码

```rust
use rocket_contrib::json::Json;

// curl http://localhost:8000/responses/content/rocket-contrib-json
#[get("/content/rocket-contrib-json")]
fn response_content_rocket_contrib_json() -> Json<Task3> {
    Json(
        Task3 {
            description: "description".to_string(),
            complete: true
        }
    )
}

fn main() {
    rocket::ignite()
        .mount("/responses", routes![response_content_rocket_contrib_json])
        .launch();
}

```

#### （6）常见 Responder 之 Templates

参见：

* [文档](https://rocket.rs/v0.4/guide/responses/#templates)
* [rocket_contrib template](https://api.rocket.rs/v0.4/rocket_contrib/templates/struct.Template.html)

### 3、类型化的URI（Typed URIs）

参见：

* [文档](https://rocket.rs/v0.4/guide/responses/#typed-uris)

```rust
#[get("/dynamic-paths/<name>")]
fn dynamic_paths(name: &RawStr) -> String {
    format!("Hello, {}!", name.as_str())
}

// curl http://localhost:8000/typed-uris
#[get("/")]
fn typed_uris() -> String {
    let u1 = uri!(dynamic_paths: "test");
    let mut s = u1.to_string();
    s.push_str("\n");
    s.push_str(uri!(dynamic_paths: "中文").to_string().as_str());
    s.push_str("\n");
    s
}

fn main() {
    rocket::ignite()
        .mount("/typed-uris", routes![typed_uris])
        .launch();
}

```

### 4、状态（State）和数据库

参见 https://rocket.rs/v0.4/guide/state/

### 5、中间件（Fairings）

参加 https://rocket.rs/v0.4/guide/fairings/

简单实用：

* 实现定义一个结构实现 [`Fairing`](https://api.rocket.rs/v0.4/rocket/fairing/trait.Fairing.html)
* `rocket::ignite().attach(该结构的实例)`

### 6、测试

参见 https://rocket.rs/v0.4/guide/testing/

### 7、配置

参见 https://rocket.rs/v0.4/guide/configuration/

## 四、例子 Pastebin

参见 https://rocket.rs/v0.4/guide/pastebin/

## 五、总结

https://rocket.rs/v0.4/guide/conclusion/
