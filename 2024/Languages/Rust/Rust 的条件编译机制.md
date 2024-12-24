#Rust 
## 基本原理　

Rust 提供了一个 `[cfg]` 的编译选项，允许你基于一个传递给编译器的标记编译代码，有两种形式：

- `#[cfg(foo)]`

  如果 `foo` 设置了编译对应代码；

- `#[cfg(bar = "baz")]`

  如果 `bar = "baz"` ，则编译对应代码；

比如：

```rust
fn main() {
    #[cfg(feature = "foo")]
    println!("foo enabled");

    #[cfg(bar)]
    println!("bar enabled");

    println!("hello");
}
```

如果我们**利用 `rustc` 和 `--cfg` 进行编译**：

```sh
# 什么都不开启
$ rustc main.rs
$ ./main
hello

# 开启 bar
$ rustc --cfg 'bar' main.rs
$ ./main 
bar enabled
hello

# 开启 foo
$ rustc --cfg 'feature="foo"' main.rs
$ ./main 
foo enabled
hello

# 全都开启
$ rustc --cfg 'feature="foo"' --cfg 'bar' main.rs
$ ./main
foo enabled
bar enabled
hello
```

Rust 还提供了一些**断言语句**：

- **`all()`**

  所有断言成立才可以编译，比如：

  ```rust
  fn main() {
      #[cfg(all(feature = "foo", bar, hello = "world"))]
      println!("foo enabled");
  
      println!("hello");
  }
  ```

  需要满足 3 个条件才进行编译:

  ```sh
  $ rustc --cfg 'feature="foo"' --cfg 'bar' --cfg 'hello="world"' main.rs
  ```

- **`any()`**

  只要有一个条件满足才进行编译。比如：

  ```rust
  fn main() {
      #[cfg(any(feature = "foo", bar, hello = "world"))]
      println!("foo enabled");
  
      println!("hello");
  }
  ```

- **`not()`**

  比如：

  ```rust
  fn main() {
      #[cfg(not(feature = "foo"))]
      println!("foo enabled");
  
      println!("hello");
  }
  ```

  如果不配置 `feature="foo"`，下面那段代码将被编译。



这几种语句还可以相互之间嵌套：

```rust
#[cfg(any(not(unix), all(target_os="macos", target_arch = "powerpc")))]
```



## 与 Cargo 配合

`Cargo.toml` 有一个 `[features]` 字段：

```toml
[features]
default = ["foo"]
foo = []
```

其中 `default` 字段配置了默认启动那些 feature。具体每一个 feature 都是一个列表，列表中表示将依赖于那些 feature，比如上文中 `foo` 这个 feature 就不依赖其他 feature：

```rust
fn main() {
    #[cfg(feature = "foo")]
    println!("foo enabled");

    println!("hello");
}
```

如果设置 `default` 开启 `foo` 则将打印出 `foo enabled`，如果保持 `default` 为空列表，则不会打印。

看一个稍微复杂点的例子：

```toml
[features]
default = ["ico", "webp"]
bmp = []
png = []
ico = ["bmp", "png"]
webp = []
```

`ico` 的开启依赖于 `bmp` 和 `png`，也就是说，只要开启了 `ico`，`bmp` 和 `png` 也开启了：

```rust
#[cfg(feature = "ico")]
fn enable_ico() {
    #[cfg(feature = "bmp")]
    println!("bmp enabled");

    #[cfg(feature = "png")]
    println!("png enabled");

    println!("ico enabled");
}

fn main() {
    #[cfg(feature = "ico")]
    enable_ico();

    println!("hello");
}
```

将打印出：

```sh
bmp enabled
png enabled
ico enabled
hello
```

## `cfg_attr` 的使用

`cfg_attr` 的使用稍微有点绕，可以看下面这个例子：

```rust
fn main() {
    #[cfg_attr(target_os = "macos", cfg(os = "mac"))]
    println!("macos");
}
```

这句话表达是：当 `target_os = "macos"` 成立时，当前的条件编译将使用 `cfg(os = "mac")`，即变成：

```rust
fn main() {
    #[cfg(os = "mac"))]
    println!("macos");
}
```

如果是在 macos 环境下，必须用如下命令进行编译才能打印 `macos`：

```sh
 rustc --cfg='os="mac"' main.rs
```



还有一个例子，比如：

```rust
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}
```

如果 `magic` 这个 feature 开启了，则对应的条件编译将展开成：

```rust
#[sparkles]
#[crackles]
fn bewitched() {}
```

## `cfg!` 的使用

可以利用 `cfg!` 来在运行时使用一些编译时的静态信息进行条件判断:

```rust
fn main() {
    if cfg!(target_os = "macos") {
        println!("macos");
    } else {
        println!("other os");
    }
}
```

# 参考

- [conditional-compilation](https://doc.rust-lang.org/reference/conditional-compilation.html)
- [cargo-features](https://doc.rust-lang.org/cargo/reference/features.html)