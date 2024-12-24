#Rust 

> 参考教程：[Here](https://www.perfcode.com/rust-serde/serde-json.html)
> Github 参考链接：[Here](https://github.com/chinanf-boy/serde_json-zh/blob/master/readme.md)
> Serde.rs 官网文档：[Here](https://serde.rs)

JSON一种常用的由键值对组成的数据对象；本文将通过多个例子讲解在Rust中如何解析JSON内容，以及如何将结构体转换成JSON字符串。

在Rust中解析JSON文本通常需要使用一个JSON库。Rust标准库中有一个名为`serde`的库，它提供了序列化和反序列化结构体和其他数据类型的功能，包括JSON。

## 添加依赖

要使用`serde`库解析JSON文本，你需要添加`serde`和`serde_json`依赖到你的项目中；

在`Cargo.toml`文件中添加以下行：

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

## 解析JSON

### 对未类型化的JSON进行解析

任何有效的JSON数据都可以转换成`serde_json::Value`数据结构：

```rust
enum Value {
    Null,
    Bool(bool),
    Number(Number),
    String(String),
    Array(Vec<Value>),
    Object(Map<String, Value>),
}
```

以下函数可用于将JSON数据解析成`serde_json::Value`结构：

- `serde_json::from_str`，用于解析JSON字符串；
- `serde_json::from_slice`，用于对字节切片`&[u8]`进行解析；
- `serde_json::from_reader`，从支持`io::Read`特性的对象中读取数据并解析，比如一个文件或TCP流；

一个使用`serde_json::from_str`的例子：

```rust
fn main() {

    // 一个&str类型的JSON数据
    let data = r#"
        {
            "name": "James Bond",
            "age": 33,
            "pet_phrase": [
                "Bond, James Bond.",
                "Shaken, not stirred."
            ]
        }"#;

    // 转换成serde_json::Value结构
    let v: serde_json::Value = serde_json::from_str(data).unwrap();

    // 通过方括号建立索引来访问部分数据
    println!("NAME: {}\nAGE: {}\n\t{}\n\t{}",
        v["name"],
        v["age"],
        v["pet_phrase"][0],
        v["pet_phrase"][1],
    );
}
```

程序运行结果

```sh
NAME: "James Bond"
AGE: 33
        "Bond, James Bond."
        "Shaken, not stirred."
```

### 将JSON解析到数据结构

`serde`提供了一种将JSON数据自动映射到Rust数据结构的方法；

修改前一个例子：

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct Person {
    name: String,
    age: u8,
    pet_phrase: Vec<String>,
}

fn main() {

    // 一个&str类型的JSON数据
    let data = r#"
        {
            "name": "James Bond",
            "age": 33,
            "pet_phrase": [
                "Bond, James Bond.",
                "Shaken, not stirred."
            ]
        }"#;

    // 转换成 Person 结构
    let p: Person = serde_json::from_str(data).unwrap();

    // 通过方括号建立索引来访问部分数据
    println!("NAME: {}\nAGE: {}\n\t{}\n\t{}",
        p.name,
        p.age,
        p.pet_phrase[0],
        p.pet_phrase[1],
    );
}
```

该程序运行效果与上一个例子相同，但这一次我们将`serde_json::from_str`函数的返回值分配给了一个自定义的类型`Person`；

> JSON数据与结构体定义不符时将产生错误；

## 将数据结构转换成JSON字符串

以下是一些可以将数据结构转换成JSON数据的函数：

- `serde_json::to_string`，将数据结构转换成JSON字符串；
- `serde_json::to_vec`，将数据结构序列化为`Vec<u8>`；
- `serde_json::to_writer`，可以序列化到任何实现了`io::Write`特性的对象中，例如文件或 TCP 流；

使用`serde_json::to_string`的一个例子：

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize)]
struct Person {
    name: String,
    age: u8,
    pet_phrase: Vec<String>,
}

fn main() {

    let mut pp = Vec::new();
    pp.push("hello world".to_string());
    pp.push("perfcode.com".to_string());

    let p = Person {
        name: "ho".to_string(),
        age: 18,
        pet_phrase: pp,
    };

    // 序列化为JSON字符串
    let s = serde_json::to_string(&p).unwrap();

    println!("{}",s);
    
}
```

程序运行效果

```json
{"name":"ho","age":18,"pet_phrase":["hello world","perfcode.com"]}
```

## json!宏

serde提供了一个`json!`宏来非常自然的构建`serde_json::Value`对象：

```rust
use serde_json::json;

fn main() {

    let info = json!(
        {
            "name": "James Bond",
            "age": 33,
            "pet_phrase":[
                "Bond, James Bond.",
                "Shaken, not stirred."
                ]
        }
    );

    // 通过方括号建立索引来访问部分数据
    println!("NAME: {}\nAGE: {}\n\t{}\n\t{}",
        info["name"],
        info["age"],
        info["pet_phrase"][0],
        info["pet_phrase"][1],
    );

    //序列化
    println!("{}",info.to_string());

}
```

程序运行效果

```sh
NAME: "James Bond"
AGE: 33
        "Bond, James Bond."
        "Shaken, not stirred."
{"age":33,"name":"James Bond","pet_phrase":["Bond, James Bond.","Shaken, not stirred."]}
```

`Value::to_string`方法可以将`serde_json::Value`对象转换成 JSON 字符串；