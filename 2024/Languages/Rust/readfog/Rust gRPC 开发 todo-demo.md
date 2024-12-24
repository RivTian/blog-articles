
在這篇文章中，我們將使用 gRPC 創建一個基本的 Todo 應用程序。首先，我們將非常快速的概述一下 gRPC 和 Protocol Buffers。

什麼是 gRPC？

gRPC 是一個現代的開源的高性能遠程過程調用 (RPC) 框架，可以在任何環境下運行。RPC 代表遠程過程調用(Remote Procedure Call)，開頭的 g 代表通用(General Purpose)，對某些人來說，它代表谷歌。

我假設你熟悉常見的 REST api，它們通過 JSON 對象進行通信。在 gRPC 中，我們使用 Protocol Buffers 來序列化數據，而不是 JSON。

Protocol Buffers

Protocol Buffers 是谷歌定製的用於語言無關、平臺無關、可擴展的，序列化結構性數據的一種協議。

在 gRPC 中，傳輸的 (序列化的) 數據是二進制的。這意味着它比 JSON 或 XML 更快，因爲它佔用更少的空間，而更少的空間帶來更小的帶寬。

Rust gRPC 開發

首先，創建一個新的工程：

```sh
cargo new rust-grpc
```

現在，在創建工程之後，我們爲 gRPC 添加一些依賴項，並在 Cargo.toml 中定義二進制程序，用於我們的服務器和客戶端：

```toml
[[bin]]
name = "grpc-server"
path = "src/server.rs"
[[bin]]
name = "grpc-client"
path = "src/client.rs"
[dependencies]
tonic = "0.7"
prost = "0.10"
tokio = { version = "1.19", features = ["rt-multi-thread", "macros"] }
[build-dependencies]
tonic-build = "0.7"
```

- tonic：一個基於 HTTP/2 的 Rust gRPC 庫
    
- tonic-build：編譯 proto 文件成 Rust 代碼
    
- prost：Rust 的 Protocol Buffers 實現庫
    
- tokio：Rust 的異步運行時
    

創建 proto 文件

在項目路徑下創建 proto/todo.proto：

```protobuf
syntax = "proto3";

import "google/protobuf/empty.proto";

package todo;

message TodoItem {
    string name = 1;
    string description = 2;
    int32 priority = 3;
    bool completed = 4;
}

message GetTodosResponse {
    repeated TodoItem todos = 1;
}

message CreateTodoRequest {
    string name = 1;
    string description = 2;
    int32 priority = 3;
}

message CreateTodoResponse {
    TodoItem todo = 1;
    bool status = 2;
}

service Todo {
    rpc GetTodos(google.protobuf.Empty) returns (GetTodosResponse);
    rpc CreateTodo(CreateTodoRequest) returns (CreateTodoResponse);
}
```

這是 Proto 文件的語法。讓我們大概看一下：

- syntax：定義了 proto 文件的版本
    
- import：允許使用來自其他 proto 文件的定義。例如，由於 GetTodos 函數沒有請求參數，所以導入 empty.proto。
    
- message：將 message 看作定義請求和響應的接口。分配給字段的數字表示字段編號。字段號是 Protobuf 的重要組成部分。它們用於標識二進制編碼數據中的字段，這意味着它們不能在 service 的不同版本之間進行更改。
    
- service：可以將其視爲一個接口，在 service 中定義了函數、它們需要什麼參數以及它們返回什麼值。
    

編譯 proto 文件

在項目目錄下創建 build.rs ：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    tonic_build::configure(
        .compile(&["proto/todo.proto"], &["proto"])
        .unwrap();

    Ok(())
}
```

目錄結構如下：

![](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/wPEUv6.png)

Server 端代碼

src/server.rs

```rust
use std::sync::Mutex;

use todo::{
    todo_server::{Todo, TodoServer},
    CreateTodoRequest, CreateTodoResponse, GetTodosResponse, TodoItem,
};
use tonic::transport::Server;

pub mod todo {
    tonic::include_proto!("todo");
}

#[derive(Debug, Default)]
pub struct TodoService {
    todos: Mutex<Vec<TodoItem>>,
}

#[tonic::async_trait]
impl Todo for TodoService {
    async fn get_todos(
        &self,
        _: tonic::Request<()>,
    ) -> Result<tonic::Response<GetTodosResponse>, tonic::Status> {
        let message = GetTodosResponse {
            todos: self.todos.lock().unwrap().to_vec(),
        };

        Ok(tonic::Response::new(message))
    }

    async fn create_todo(
        &self,
        request: tonic::Request<CreateTodoRequest>,
    ) -> Result<tonic::Response<CreateTodoResponse>, tonic::Status> {
        let payload = request.into_inner();

        let todo_item = TodoItem {
            name: payload.name,
            description: payload.description,
            priority: payload.priority,
            completed: false,
        };

        self.todos.lock().unwrap().push(todo_item.clone());

        let message = CreateTodoResponse {
            todo: Some(todo_item),
            status: true,
        };

        Ok(tonic::Response::new(message))
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let addr = "0.0.0.0:50051".parse().unwrap();
    let todo_service = TodoService::default();

    Server::builder()
        .add_service(TodoServer::new(todo_service))
        .serve(addr)
        .await?;

    Ok(())
}
```

讓我們快速看看發生了什麼：

- 首先，我們引入了來自 tonic, Request, Response 等所需的結構體。
    
- 同樣，我們也引入了從 todo.proto 編譯的 Rust struct，可以在 / target/debug/build 目錄中看到它們。
    
- 現在我們需要創建 TodoService 併爲該服務實現 Todo trait。
    
- 然後，實現我們的函數，get_todos 將簡單地返回帶有 todo 項的 GetTodosResponse。create_todo 函數將獲取 request 請求，從 TodoItem struct 創建 todo 項，將其加入到 self.todos 中並返回 CreateTodoResponse，創建 todo 並且狀態爲 true。
    
- 最後，我們在 0.0.0.0:50051 上啓動服務器
    

Client 端代碼

src/client.rs

```rust
use crate::todo::{todo_client::TodoClient, CreateTodoRequest};

pub mod todo {
    tonic::include_proto!("todo");
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut client = TodoClient::connect("http://0.0.0.0:50051").await?;

    let request = tonic::Request::new(());

    let response = client.get_todos(request).await?;

    println!("{:?}", response.into_inner().todos);

    let create_request = tonic::Request::new(CreateTodoRequest {
        name: "test name".to_string(),
        description: "test description".to_string(),
        priority: 1,
    });

    let create_response = client.create_todo(create_request).await?;

    println!("{:?}", create_response.into_inner().todo);

    Ok(())
}
```

- 同樣，讓我們快速看看這裏發生了什麼：
    
- 首先，我們引入了從 todo.proto 編譯的 TodoClient 和 CreateTodoRequest。
    
- 現在，我們在 main 函數中，連接到 0.0.0.0:50051，併爲 get_todos 創建一個空請求 (你應該記得，我們在 proto 文件中沒有 GetTodos 的請求參數)。然後發送請求，等待響應，最後打印；對於 create_todo 幾乎相同，但這次我們將 CreateTodoRequest struct 傳遞給請求，然後再次打印。
    

運行

```sh
cargo build
cargo run --bin grpc-server
cargo run --bin grpc-client
```

本文翻譯自：

https://betterprogramming.pub/how-to-create-grpc-server-client-in-rust-4e37692229f0