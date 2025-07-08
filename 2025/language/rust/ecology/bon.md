# Rust Builder模式详解与Bon库最佳实践：深入理解与应用

## Builder Pattern

> [!NOTE]
>
> Builder 模式概述
>
> - 作为一种 <font color="#DF718A"><b>创建型设计模式</b></font>，主要用于构建复杂对象。它通过逐步设置对象的属性来创建对象，而不是在一个庞大的构造函数中传入所有参数，从而提升了代码的 <font color="#80AEE7"><b>可读性</b></font>与<font color="#80AEE7"><b>维护性</b></font>。

> [!TIP]
>
> 为什么选择 Builder 模式
>
> 在开发过程中，如果遇到以下场景，
>
> - **构造函数参数过多**：Builder模式允许开发者逐步构建对象，通过链式调用来设置属性，这样可以避免大量参数传递所带来的困扰。
> - **处理可选参数**：Builder模式提供了一种优雅的方式处理可选参数，避免多层的`Option`嵌套。
>
> Builder模式可以极大地提升代码的可读性和可维护性

```rust
// 不使用Builder模式 - 难以阅读和维护
let user = User::new("John", "Doe", 25, "john@example.com", "123 Street", true, false);

// 使用Builder模式 - 清晰直观
let user = User::builder()
    .name("John")
    .email("john@example.com")
    .age(25)
    .address("123 Street")
    .build();

// PS: 此外如果 User 新增属性也可不影响现有代码
```



## Typestate Pattern

> [!TIP]
>
> Typestate 模式利用 Rust 强大的类型系统来确保对象在<font color="#f1c17d"><b>正确的状态下</b></font>被使用。它能有效防止对象在无效状态下执行操作，从而减少运行时的错误风险。

以下代码展示了如何使用Typestate模式在构建对象时确保参数完整性：

```rust
use std::marker::PhantomData;

struct Uninitialized;
struct HasName;
struct HasEmail;

// Builder实现
struct UserBuilder<State> {
    name: String,
    email: String,
    _state: PhantomData<State>
}

impl UserBuilder<Uninitialized> {
    fn new() -> Self {
        UserBuilder {
            name: String::new(),
            email: String::new(),
            _state: PhantomData
        }
    }

    fn name(self, name: String) -> UserBuilder<HasName> {
        UserBuilder {
            name,
            email: self.email,
            _state: PhantomData
        }
    }
}
```

通过上述实现，开发者可以确保在编译阶段就完成对对象状态的检查。



## Bon Crate with examples

### Bon 库的基础使用

Bon库为Rust提供了强大的Builder模式实现，可以轻松地构建复杂对象。

```rust
use bon::Builder;

#[derive(Builder)]
struct User {
    name: String,
    #[builder(default)]
    age: Option<u32>,
    email: String,
}
```

### Bon 库的高级特性

1. **自定义验证规则**

   可以通过`#[builder(validate)]`属性为字段添加自定义验证逻辑。

   ```rust
   #[derive(Builder)]
   struct Server {
       #[builder(validate = port > 1000)]
       port: u16,
       #[builder(validate = |host: &str| host.contains("."))]
       host: String,
   }
   ```

2. **默认值设置**

   使用`#[builder(default)]`或直接指定默认值来简化对象构建。

   ```rust
   #[derive(Builder)]
   struct Config {
       #[builder(default = 8080)]
       port: u16,
       #[builder(default = String::from("localhost"))]
       host: String,
   }
   ```

3. **类型转换**

   Bon支持自动类型转换，例如`into`和`try_into`，使得构建器更加灵活。

   ```rust
   #[derive(Builder)]
   struct Connection {
       #[builder(into)]
       address: String,
       #[builder(try_into)]
       timeout: Duration,
   }
   ```

   

### Bon 与其他 Builder 库的对比分析

- VS `typed-builder`

```rust
// typed-builder
#[derive(TypedBuilder)]
struct User {
    name: String,
    email: Option<String>,
}

// Bon
#[derive(bon::Builder)]
struct User {
    name: String,
    email: Option<String>,
}
```

#### 主要区别

1. 类型状态表示
   - Bon使用更简洁的嵌套类型，而typed-builder依赖更为复杂的元组类型。
2. 编译性能
   - Bon生成更少的代码，因此具有更快的编译速度。

### Bon 性能优化策略

1. **零成本抽象**

```rust
#[derive(bon::Builder)]
struct OptimizedConfig {
    #[builder(inline)]
    name: String,
    #[builder(no_clone)]
    data: Vec<u8>,
}
```

2. **内存优化**

- Bon支持`#[builder(no_std)]`，帮助开发者在内存受限的环境中构建对象。

```rust
#[derive(bon::Builder)]
#[builder(no_std)]
struct MinimalStruct {
    value: u32,
}
```



## Bon Builder Demo

### 数据库配置构建器

```rust
#[derive(bon::Builder)]
struct DatabaseConfig {
    host: String,
    port: u16,
    #[builder(default = 30)]
    timeout_seconds: u32,
    #[builder(default)]
    max_connections: Option<u32>,
}

// 使用示例
let config = DatabaseConfig::builder()
    .host("localhost".to_string())
    .port(5432)
    .timeout_seconds(60)
    .build()?;
```

### HTTP客户端构建器

```rust
#[derive(bon::Builder)]
struct HttpClient {
    #[builder(default = "https://api.example.com")]
    base_url: String,
    #[builder(default = Duration::from_secs(30))]
    timeout: Duration,
    #[builder(default)]
    headers: HashMap<String, String>,
}

// 使用示例
let client = HttpClient::builder()
    .base_url("https://api.custom.com".to_string())
    .timeout(Duration::from_secs(60))
    .build()?;
```

## 高级使用技巧与最佳实践建议

### 设计原则

1. **保持简单性**：仅为必要的字段添加构建器，以避免代码复杂化。
2. **类型安全**：通过Rust类型系统保证对象构建的正确性。
3. **文档完备**：为每个字段添加注释，提供使用示例以提高代码的易用性。

### examples

```rust
/// 应用配置构建器
#[derive(bon::Builder)]
#[builder(doc = "构建应用配置")]
struct AppConfig {
    /// 服务器监听端口
    #[builder(default = 8080)]
    port: u16,
    
    /// 数据库连接URL
    #[builder(validate = |url: &str| url.starts_with("postgres://"))]
    database_url: String,
    
    /// 日志级别
    #[builder(default = "info")]
    log_level: String,
}
```



---

## 总结与参考链接

Bon库提供了Rust开发中非常强大且灵活的Builder模式实现，其优势在于：

- **简化开发**：自动生成构建器代码，减少样板代码。
- **保证安全**：通过编译时类型检查和自定义验证，保证对象的正确性。
- **优化性能**：提供零成本抽象，最小化运行时开销。

通过合理使用Bon库，我们可以编写出更加健壮、可维护的Rust代码。

### Reference

- [Next-gen builder macro Bon 3.0 release. Revolutional typestate design 🚀](https://bon-rs.com/blog/bon-v3-release)

- [Rust实现构建器模式和使用Bon库中的构建器](https://www.cnblogs.com/vinciyan/p/18351094)