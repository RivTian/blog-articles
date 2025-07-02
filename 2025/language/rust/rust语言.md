> 版本 1.46.0

参考：

* https://www.rust-lang.org/zh-CN/
* https://doc.rust-lang.org/stable/book/
* [标准库doc官方](https://doc.rust-lang.org/std/)
* [标准库doc中文](https://s0doc0rust-lang0org.icopy.site/std/index.html)
* [Rust 程序设计语言 简体中文版](https://kaisery.github.io/trpl-zh-cn/)
* [深入浅出 Rust](https://book.douban.com/subject/30312231/) （微信读书/京东读书可在线阅读）
* [通过例子学 Rust](https://rustwiki.org/zh-CN/rust-by-example/)
* [Rust 社区文档](https://learnku.com/rust/docs)
* [Rust 高级编程](https://learnku.com/docs/nomicon/2018/brief-introduction/4702)
* [The Rust Reference 官方权威参考手册](https://doc.rust-lang.org/stable/reference/)
* [绅士地介绍 Rust](http://llever.com/gentle-intro/readme.zh.html)
* [The Little Book of Rust Macros 元编程](https://danielkeep.github.io/tlborm/book/index.html)
* [Rust 单页手册](https://cheats.rs/)
* [Rust 版本指南](https://erasin.wang/books/edition-guide-cn/)
* [Rust 官方论坛](https://users.rust-lang.org/)
* [Rust 技术论坛](https://learnku.com/rust)
* [Rust 语言中文社区](https://rust.cc/)
* [Rust 库热度](https://lib.rs/)

## 一、安装和配置

### 1、*nix安装

```bash
curl https://sh.rustup.rs -sSf | sh
```

默认安装位置为：`~/.cargo/bin`

如果使用VSCode集成开发环境，安装完成后，需要彻底重启VSCode。

### 2、配置环境变量

正常情况下，默认会将环境变量配置好，即在 `~/.bash_profile` 文件中添加一行 `export PATH="$HOME/.cargo/bin:$PATH"`

### 3、配置集成开发环境（VSCode）

开发环境安装

* [旧版不需要安装](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust)
* [推荐安装](https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer)

调试器安装

https://jason-williams.co.uk/debugging-rust-in-vscode

https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb

https://github.com/vadimcn/vscode-lldb/blob/v1.2.3/MANUAL.md#cargo-support

例子配置

```jsonc
{
	"version": "0.2.0",
	"configurations": [
		{
			"name": "test",
			"type": "lldb",
			"request": "launch",
			"cargo": {
				// "args": ["test", "--no-run", "--lib"], // Cargo command line to build the debug target
				"args": ["build"], // is another possibility
			}
		}
	]
}
```

新版扩展：

* [扩展商店](https://marketplace.visualstudio.com/items?itemName=matklad.rust-analyzer)
* [官网](https://rust-analyzer.github.io/)
* [用户文档](https://github.com/rust-analyzer/rust-analyzer/tree/master/docs/user)
* [特性文档](https://github.com/rust-analyzer/rust-analyzer/blob/master/docs/user/features.md)

使用指南

* `cmd + shift + o` 转到文件中的符号
* `cmd + t` 查找符号
    * 后缀说明
        * `#` 搜索类型
        * `*` 搜索工作空间依赖
    * `#Foo` 在当前工作空间搜索类型
    * `#foo#` 在当前工作空间搜索函数 searches for foo function in the current workspace
    * `#Foo*` 搜索所有依赖的类型
    * `#foo#*` 搜索所有依赖的函数
* 扩展选择：`cmd + ctl + shift + ←/→`
* GOTO系列
    * 跳转到定义
    * 跳转到实现
    * 跳转到类型定义
* 内置命令 analyzer
    * `rust analyzer: run` 和 `ctl+r` 运行命令
    * `rust analyzer: Locate Parent Module` 查找父模块
    * `rust analyzer: Find Matching Brace` 匹配括号
    * `rust analyzer: Join Lines` 多行合并为1行
    * `rust analyzer: Show Syntax Tree` 查看语法树
    * `rust analyzer: Expand Macro Recursively` 展开宏
    * `rust analyzer: Status` LS状态
    * `rust analyzer: Run garbage collection` 运行LS垃圾回收器
* Assists (Code Actions)
    * https://github.com/rust-analyzer/rust-analyzer/blob/master/docs/user/assists.md
* 魔法完成
    * `expr.if -> if expr {}`
    * `expr.match -> match expr {}`
    * `expr.while -> while expr {}`
    * `expr.ref -> &expr`
    * `expr.refm -> &mut expr`
    * `expr.not -> !expr`
    * `expr.dbg -> dbg!(expr)`
    * `pd -> println!("{:?}")`
    * `ppd -> println!("{:#?}")`
    * `tfn -> #[test] fn f(){}`

### 4、实用开发工具

参考：https://kaisery.github.io/trpl-zh-cn/appendix-04-useful-development-tools.html

* `rustfmt` 自动格式化
    * 安装 `rustup component add rustfmt`
    * 使用 `cargo fmt`
* `rustfix` 修复代码
    * 使用 `cargo fix`
* 通过 `clippy` 提供更多 lint 功能
    * `rustup component add clippy`
    * 使用 `cargo clippy`
* 使用 Rust Language Server 的 IDE 集成
    * 安装 `rustup component add rls`
* [Cargo宏展开](https://github.com/dtolnay/cargo-expand)
    * 安装 `cargo install cargo-expand`
    * 展开当前项目顶层模块 `cargo expand`
    * 展开当前项目指定子模块
        * `cargo expand mod_a`
        * `cargo expand mod_a::mod_b`
    * 展开后输出到文件 `cargo expand mod_a::mod_b > file.rs`

### 5、crates国内镜像配置

https://lug.ustc.edu.cn/wiki/mirrors/help/rust-crates

## 二、起步

### 0、语言特点

Rust是mozilla推出的一款系统级的编程语言，其两大特点在于零开销抽象和安全性。

* 手动内存管理
* 静态类型语言，自动类型推断
* 系统级别语言效率类似于C++
* 无null类型设计

### 1、HelloWorld

```bash
mkdir hello_world
cd hello_world
```

创建文件 `main.rs`

```rust
fn main() {
    println!("Hello, world!");
}
```

编译运行

```bash
rustc main.rs
./main
```

从HelloWorld可以看出Rust语言的一些特性：

* 支持顶层函数（不像Java，只能写方法）
* 需要分号
* C家族的语法风格（花括号）
* 奇怪的 `!`，文档说是宏

### 2、Hello cargo

cargo是rust包管理器，定义了标准rust项目的目录结构，并解决依赖

查看 cargo 版本

```bash
cargo --version
```

#### （1）创建Cargo项目

```bash
cargo new hello_cargo
```

* 将会创建一个hello_cargo的目录（VSCode插件全部功能使用，必须将cargo项目作为工作空间根目录）
* 同时初始化一个git仓库
* 包含src目录
* 包含cargo配置文件Cargo.toml

`Cargo.toml` 文件内容如下

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["rectcircle <rectcircle96@gmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

#### （2）常用命令

* `cargo new project_name`
* `cargo build`
* `cargo run`
* `cargo check` 语法检查

## 三、基本语法

### 1、体验——猜数字游戏

创建新的项目进行试验

```bash
cargo new guessing_game
```

修改 `Cargo.toml` 添加依赖

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["rectcircle <rectcircle96@gmail.com>"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.3.14"
```

* `"0.3.14"` 表示 `^0.3.14` 表示与该版本兼容的版本
* `Cargo.lock` 保证全部用户使用相同的版本，除非手动的`cargo update`或者更新`Cargo.toml`文件

修改 `src/main.rs`

```rust
use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

* 导入库语法 `use`，父子模块使用`::`分割
* 创建变量，使用 `let`，`mut`表示可变，不加表示不可变变量
* `new()` 相当于构造函数，`::`调用函数表示调用静态函数
* 使用 `use std::io;`，就可以使用`io::stdin()`调用，否则只能使用`std::io::stdin()`调用
    * 可以这么做 `use std::io::stdin;`，直接引入函数，`stdin()`调用
* `loop` 类似 `while(true)`
* `&` 表示传引用，默认引用不可便，`&mut`声明引用可变
* `read_line` 将返回一个Result对象，可能是Ok或Err，expect方法将返回真正的结果，如果Err将展示异常信息然后直接退出程序（调用panic!）
* 支持类似Scala的模式匹配、智能类型推测
* `println!` 支持模板字符串

### 2、变量与不可变

创建新项目

```bash
cargo new variables
```

修改 `src/main.rs`

变量默认不可变，不可以重新赋值（包括传递引用后修改也不允许），否者编译报错

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6; // 此行将编译报错，因为修改了不可便变量
    println!("The value of x is: {}", x);
}

```

允许变量可变使用 mut 声明

```rust
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
```

#### （1） 变量（let）、常量（const）和 静态变量（static）

* const语义上是常量，在运行期不可改改变
* const不能使用`mut`修饰
* const必须显示指定类型
* const必须在声明时初始化
* const可以在任何范围内声明，包括全局范围，这使得它们对许多代码部分需要了解的值很有用。
* const只设置为常量表达式，不是能是函数调用的结果或只能在运行时计算的任何其他值（const fn）。
* const可能被编译器内联优化掉，因此不能取引用、模式匹配等

```rust
    const MAX_POINTS: u32 = 100_000;
```

* const在程序运行的整个时间内有效，在它们声明的范围内，使它们成为应用程序域中程序的多个部分可能需要知道的值的有用选择，例如最大点数、允许游戏的玩家获得光速或光速。
* 将整个程序中使用的硬编码值命名为const，有助于将该值的含义传达给代码的未来维护者。如果将来需要更新硬编码值，那么在代码中只需要更改一个位置也是有帮助的。

static

* static是静态变量
    * 静态体现生命周期与整个程序一致
    * 变量表示可以更改（但是带有 mut 的所有操作都是是 unsafe 的）
* static初始化只设置为常量表达式，不是能是函数调用的结果或只能在运行时计算的任何其他值（const fn）

#### （2）变量覆盖

```rust
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);

    let spaces = "   ";
    let spaces = spaces.len();
    println!("The value of spaces is: {}", spaces);

    // let mut spaces1 = "   ";
    // spaces1 = spaces1.len(); // 编译报错
```

* 在同一作用域可以多次声明同名不可便对象，下面声明的将覆盖上面声明的变量，可以实现类似于可变变量赋值的效果
* 变量覆盖还支持不同类型
* 当然赋值不允许

### 3、数据类型

主要分为两类：标量（scalar基本数据类型）和复合（compound）

#### （1）标量数据——整型

| 长度   | 有符号 | 无符号 |
|-------|--------|------|
| 8-bit | `i8` | `u8` |
| 16-bit | `i16` | `u16` |
| 32-bit | `i32` | `u32` |
| 64-bit | `i64` | `u64` |
| 128-bit | `i128` | `u128` |
| arch | `isize` | `usize` |

#### （2）标量数据——浮点类型

```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

#### （3）数字计算

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;

    // remainder
    let remainder = 43 % 5;
}

```

* 在 debug 模式下 整数溢出 将抛出 panic，在 release 模式下不抛出 panic
    * `rustc -C overflow-checks=`可以写`yes`或者`no`控制是否触发
* 浮点数不具备全序性 `let nan = std::f32::NAN; println!("{} {} {}", nan < nan, nan == nan, nan > nan);` 输出 `false false false`

#### （4）标量数据——bool类型

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

#### （4）标量数据——字符类型

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

* 32 位 Unicode 码

#### （5）复合类型——元组

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

#### （6）复合类型——数组

```rust
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
    let b = [3; 5]; // 长度为5，元素全部都是3的数组


    let first = a[0];
    let second = a[1];
}
```

数组越界将产生panic（异常）

数组相关：[切片类型](#8-切片类型)

#### （7）结构体

参见 [第五章](#五-结构体)

#### （8）枚举

参见 [第六章 - 枚举和模式匹配](#1-枚举和模式匹配)

#### （9）Range 类型

* Rust 中 实现了多种 Range 类型，并为之实现了语法糖
* 为之实现了迭代器可以用于 for 循环
* 可以配合数组/切片，进行切片操作

```rust
        let r = 0..3;  // [0, 3)
        let r2:std::ops::Range<i32> = 0..3;  // [0, 3)
        let r3 = 0..=3;  // [0, 3]
        let r4 = 0..; // [0, +∞)
        let r5 = ..0; // (-∞, 0)
        let r6 = ..=0; // (-∞, 0]
        let r7 = ..=0; // (-∞, 0]
        let r7 = ..; // (-∞, +∞)

        let a:[i32;3] = [1,2,3];
        let b = &a[..];
        use std::ops::Index;
        use std::ops::Deref;
        let b: &[i32;3] = &a;
        let b = b.index(..);
        println!("{:?}", b);
```

#### （10）其他标准库常见类型

参见

* [四、所有权系统 - 8、切片类型](#8-切片类型)
* [八、常见的集合](#八-常见的集合)

### 4、函数

#### （1）基本特性

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

* 定义顺序和调用顺序不相关
* 函数体可以包含表达式和声明（rust是一门表达式语言）
    * 表达式必须有返回值（比如：宏、函数调用、`{}`包裹的语句）
    * 声明没有返回值（比如let, fn）
* `{}` 包裹的（scala和传统语法的合体）
    * 最后一个语句如果没有分号，则最后一个表达式的结果是该语句块的结果
    * 最后一个语句有分号，则语句块无返回值

```rust
        fn add(a: i32, b: i32) -> i32 {
            let c = {
                a+b // 不需要加分号
            };
            c
        }
```

#### （2）返回值

* 函数的返回值是其最后一个表达式的值
* `return` 在设计上用于提前返回

```rust
    fn five() -> i32 {
        5
    }

    // fn plus_one(x: i32) -> i32 {
    //     x + 1; // 报错
    // }
```

#### （3）语句和表达式

* 类似 Scala，Rust 是 全表达式语言（有返回值），准确的说：
    * 有分号就是语句
    * 无分号就是表达式
* 赋值语句的表达式返回值为 `()`
* `panic!`、`loop`（死循环）和 `std::process::exit` 等退出语句 的返回值是 `!` （发散类型），发散类型可以赋值给任意类型，设计这个类型的目的在于满足类型推断自洽，比如 `let x = if true { 1 } else { panic!("false") }`，此时 `if else` 表达式的返回值是 `i32` 类型

```rust
    {
        println!("赋值表达式返回值为()");
        let b;
        let r = b = 1 + 1;
        println!("{:?} {}", r, b);
    }
    {
        fn p() -> ! {
            panic!("test");
        }
        let x: i32 = p();
        let y: f32 = p();
    }
```

#### （4）函数类型

* rust 中函数是一等公民，有对应的函数类型，函数可以赋值给变量，作为函数参数、返回值
* 将函数赋值给变量，默认类型推断的函数类型与函数绑定，无法将其他签名一致的函数赋值给他。如果需要需要明确声明

```rust
    {
        fn add1(t: (i32, i32)) -> i32 {
            t.0 + t.1
        }
        fn add2((a, b): (i32, i32)) -> i32 {
            a + b
        }
        let mut fn_var = add1;
        // fn_var = add2; // 报错 mismatched types expected fn item `fn((_, _)) -> _ {part01::ch04::func::add1}` found fn item `fn((_, _)) -> _ {part01::ch04::func::add2}`
        fn_var((1,2));

        let mut fn_var2 = add1 as fn((i32, i32)) -> i32;
        fn_var2 = add2;
        fn_var2((1, 3));
    }
```

#### （5）main 函数

* 只有一种，签名为 `fn main() -> ()`

#### （6）const fn

可以在编译器运行的函数，RFC：[const_fn](https://rust-lang.github.io/rfcs/0911-const-fn.html)，目前存在诸多限制，主要用于声明 const 或 static 变量。

```rust
pub const fn cube(num: usize) -> usize {
    num * num * num
}

pub static STATIC_VAR: usize = cube(3usize);
```

### 5、注释

```rust
// hello, world
```

* 普通注释，使用双斜杠
* 文档注释使用 `/**/`

### 6、控制流

#### （1）条件语句

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

* 类似Java条件只能是bool类型
* 类似Scala，if也是表达式（前提类型匹配）

#### （2）循环

使用loop，死循环，使用break终止

```rust
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);

    'outer: loop {
        while true {
            break 'outer;
        }
    }
```

使用while

```rust
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!");

    let mut x = vec![1, 2, 3];

    while let Some(y) = x.pop() {
        println!("y = {}", y);
    }

    while let _ = 5 {
        println!("Irrefutable patterns are always true");
        break;
    }
```

使用for

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }

    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

* 都支持 break, continue 类似Java支持label, 使用break goto
* while 支持模式匹配

### 7、错误处理

Rust 异常一般分为两类：可恢复错误（recoverable）和 不可恢复错误（unrecoverable）可恢复错误通常代表向用户报告错误和重试操作是合理的情况，比如未找到文件。不可恢复错误通常是 bug 的同义词，比如尝试访问超过数组结尾的位置。

在Rust中表现不是异常，而是`Result<T, E>`以及 `panic!`

#### （1） `panic!` 与不可恢复错误

`panic!` 一旦被触发程序将直接 exit！

`panic!` 宏的调用默认将会打印程序调用堆栈。当然可以通过配置在生产环境中关闭（编译文件更小，相当于exit）：

```toml
[profile.release]
panic = 'abort'
```

自己的程序触发的panic

```rust
fn main() {
    panic!("crash and burn");
}
```

运行结果

```
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/error_handle`
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

第三方库触发的panic，比如数组越界

```rust
    let v = vec![1, 2, 3];
    v[99];
```

运行结果

```
$ cargo run
   Compiling error_handle v0.1.0 (/Users/sunben/Workspace/learn/rust/error_handle)
    Finished dev [unoptimized + debuginfo] target(s) in 0.64s
     Running `target/debug/error_handle`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /rustc/4560ea788cb760f0a34127156c78e2552949f734/src/libcore/slice/mod.rs:2717:10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace.
```

以上只会报告异常出现的位置，但是不会打印调用堆栈。根据提示使用 `RUST_BACKTRACE=1` 环境变量可以打印错误堆栈

```bash
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/error_handle`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /rustc/4560ea788cb760f0a34127156c78e2552949f734/src/libcore/slice/mod.rs:2717:10
stack backtrace:
   0: backtrace::backtrace::libunwind::trace
             at /Users/runner/.cargo/registry/src/github.com-1ecc6299db9ec823/backtrace-0.3.37/src/backtrace/libunwind.rs:88
  省略...
  26: std::rt::lang_start
             at /rustc/4560ea788cb760f0a34127156c78e2552949f734/src/libstd/rt.rs:64
  27: error_handle::main
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

注意以上追踪必须启用 debug 标识。当不使用 --release 参数运行 cargo build 或 cargo run 时 debug 标识会默认启用。

注意：在 panic 发生时，Rust 的 RAII 仍会正常工作，会进行内存回收和 drop 函数的调用

#### （2） Result 与可恢复的错误

类似于go语言，可能出错的调用一般返回被封装到 `std::result::Result` 中，针对 `Result`，我们可选的处理方式：

* 使用模式匹配进行处理
* 调用快捷方法，当出现错误时触发panic方式退出

```rust
use std::fs::File;
use std::io::ErrorKind;
use std::io;
use std::io::Read;
use std::fs::File;
use std::fs;


fn main() {
    let f = File::open("hello.txt");

    // 模式匹配处理错误
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };

    // 匹配不同错误
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Tried to create file but there was a problem: {:?}", e),
            },
            other_error => panic!("There was a problem opening the file: {:?}", other_error),
        },
    };

    //调用快捷方法，以简化模式匹配
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });

    // 出现错误时直接退出的快捷方法
    let f = File::open("hello.txt").unwrap(); //`unwarp` 直接展开错误
    let f = File::open("hello.txt").expect("Failed to open hello.txt"); // `expect` 可以打印自定义字符串
}

// 错误传递：返回一个Result<R,E>
// 去取文件到内存字符串
fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}

// 使用? 简写异常传递 和read_username_from_file完全等价
// 注意 ? 只能被用于返回 Result 的函数
fn read_username_from_file1() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?; // 表示如果出现异常直接返回 Err(e)
    Ok(s)
}

// 当然对于读取到字符串的操作rust提供了一个函数
fn read_username_from_file2() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

#### （3） 选择`panic!`还是`Result`

* 一般情况最好使用 `Result`，因为其给调用者更多的选择（调用者可以选择是处理还是`panic`）
* 示例、代码原型和测试都非常适合 panic
* 在当有可能会导致有害状态的情况下建议使用 `panic!`，比如：用于规范调用者的输入

#### （4）接受 `panic` 防止线程退出

```rust
fn happen_panic() {
    // let x = None::<i32>;
    // x.unwrap();
    panic!("panic message");
}

fn catch_panic() {
    panic::set_hook(Box::new(|panic_info| {
        // do nothing
        if let Some(s) = panic_info.payload().downcast_ref::<&str>() {
            println!("panic occurred: {:?}", s);
        } else {
            println!("panic occurred");
        }
    }));
    let result = panic::catch_unwind(happen_panic);
    println!("======");
    println!("{:?}", result.err().unwrap());
    println!("======");
}
```

注意事项

* 该方式不建议作为业务异常的处理方式，仅建议在如下场景使用
    * FFI ，C语言调用 Rust 时，防止出现未定义行为
    * Web 框架/线程池等防止程序退出
* 该方式在 通用 PC 上可用，在嵌入式设备中，可能无法生效，原因在于在不同的系统，panic 的实现方式不同
    * stack unwind 堆栈展开 （占用较多资源）
    * abort

#### （5）`panic` 异常安全

在实现类似标准库这种核心库时，在调用用户定义的函数时，需要考虑若其抛出 panic 时，需要保证不能破坏 Rust 的安全性，这种安全性称为 异常安全（panic safety）。异常安全分为几个级别

* No-throw 所有异常都会在内部得到处理
* Strong exception safety 强异常安全保证，保证异常发生时，所有状态可回滚到初始状态，不会导致不一致问题
* Basic exception safety 基本异常保证，发成异常时，不会发生资源泄漏
* No exception safety 没有任何异常保证

例如 `Box<[T]>` 的 `Clone` 方法的实现，为了保证 Basic exception safety，为了保证 调用 T 的 clone 方法时可能发生的 Panic，rust 专门定义了一个 `BoxBuilder` 结构体

### 8、内存管理

赋值、函数参数传递、函数返回均按值传递（也就是说是拷贝），和 C 语一致

```rust
    let s = String::from("str");
    println!("0x{:x}", &s as *const String as usize);
    // 所有权转移的同时进行栈内存拷贝
    let s2 = s;
    println!("0x{:x}", &s2 as *const String as usize);

    let slen = mem::size_of::<String>();
    println!("{}", slen);
```

内存申请和回收相关的关于所有权相关细节参见下文

### 9、抽象方式

* struct、方法、trait抽象（类似于Golang）
* 函数式编程特性
    * 模式匹配
    * 闭包
    * 迭代器
* 元编程（宏系统）

细节参见下文

### 10、特别说明字面量

创建字面量 `cargo new literal` 项目

#### （1） 多行字符串 (Row String)

```rust
    // raw-string-literals
    // https://doc.rust-lang.org/reference/tokens.html#raw-string-literals
    // https://rahul-thakoor.github.io/rust-raw-string-literals/
    println!(r"多行字符串
    多行字符串，不能表达引号
    ");
    println!(r#"多行字符串
    多行字符串，不能表达引号""""" 可以表达引号，可以表达#号
    不能表达"紧接着#
    "#);
    println!(r##"多行字符串
    多行字符串，不能表达引号""""" 可以表达引号，可以表达#号，可以表达"#
    不能表达"紧接着##
    "##);
```

## 四、所有权系统

> https://zhuanlan.zhihu.com/p/27571264

所有权系统是 Rust 不同其他语言的最重要的部分。是为了解决内存分配问题而设计的。

同其他编程语言一样rust内存也被划分为堆（heap）和栈（stack）。在函数中：

* 值类型的变量将被防止在栈中
* 复合类型的引用放置于栈中，数据放置与堆中

### 1、所有权规则

* 所有的值都有一个叫做owner的变量
* 一次只能有一个owner
* 当owner超出的作用域，值将被回收

### 2、变量作用域

和其他预览类似：一个花括号将创建一个作用域，超出作用域的变量将无法访问

```rust
    {                      // s 非法，因为还没有声明
        let s = "hello";   // s 是合法的
        println!("{}", s);
        // 使用 s 做一些事情
    }                      // 超出作用域 s 不可用
```

#### `String` 类型示例

`String`是标准库提供的一个可变字符串。示例如下：

```rust
    let s = String::from("hello");    // 这样声明表示不可变，从一个字符串字面量创建String

    let mut s = String::from("hello"); // 声明可变的string

    s.push_str(", world!"); // push_str() 字符串拼接

    println!("{}", s); // 将打印出 `hello, world!`
```

#### 内存分配

`String::from` 实际上是在堆上申请了内存空间用于存放字符串。但是何时free，一般有两种做法：

* 自动化垃圾回收器Java等
    * 缺点：带来额外的开销
* 手动回收类似于C、C++
    * 极易出现错误和严重的漏洞

Rust不同于以上两种：

* 一旦变量超出作用域，将自动回收

```rust
    {
        let s = String::from("hello"); // s 从改点可以访问

        // do stuff with s
    }                                  // 作用域结束，s不可访问（伴随drop）
                                   // longer valid
```

#### 所有权转移

针对赋值，分配在堆上的变量将发生所有权转移，在栈上的变量将进行赋值

栈上的情况

```rust
    {
        let x = 5;
        let y = x; // x 分配在栈上，x将copy到y上，所以x, y都可以访问
        println!("{}", x);
    }
```

堆上的情况

```rust
    let s1 = String::from("hello");
    let s2 = s1; // 可以称之为所有权转移，浅拷贝，同时让s1失效

    // println!("{}, world!", s1); // 报错：borrow of moved value: `s1` value borrowed here after moverustc(E0382)
```

某结构体变量内部字段所有权转移后，该字段的所有权也转移了（析构的时候编译器会自动添加析构调用，不会导致重复析构，可能会生成一堆if-else来实现析构）

```rust
    {
        #[derive(Debug)]
        struct S {
            a: String,
            b: String,
        }
        let mut c: String;
        let mut d: String;
        let mut s = S {
            a: String::from("a"),
            b: String::from("b")
        };
        c = s.a;
        d = s.b;
        println!("{:?}", c);
        println!("{:?}", d);
        // println!("{:?}", s); // borrow of moved value: `s` move occurs because `s.a` has type `std::string::String`, which does not implement the `Copy` trait
    }
```

堆上拷贝的方法

```rust
    {
        let s1 = String::from("hello");
        let s2 = s1.clone();

        println!("s1 = {}, s2 = {}", s1, s2); // 不会报错
    }
```

本质上，不管是堆上还是栈上 用 拥有 所有权的变量 赋值给其他变量（`let new_owner = owner`），有两种可能： move 和 copy

* move 的情况，当 owner 没有实现 `Copy` (`std::marker::Copy`) 时，owner 将 拷贝到（`memcpy`） new_owner中，同时，`owner` 不可用
* copy 的情况，当 owner 实现了 `Copy` 时，调用 Copy 方法，赋值给 new_owner，同时，`owner` 仍可以使用。

注意，`owner` 可以是结构体的成员

```rust
    {
        #[derive(Debug)]
        struct S {
            a: String,
            b: String,
        }
        let mut s = S {
            a: String::from("a"),
            b: String::from("b")
        };
        let mut c = s.a;
        // println!("{:?}", s); // borrow of moved value: `s` move occurs because `s.a` has type `std::string::String`, which does not implement the `Copy` trait
    }
    {
        // Copy
        #[derive(Debug, Clone, Copy)]
        struct S {
            a: i32,
            b: i32,
        }
        let owner = S {
            a: 1,
            b: 2
        };
        let new_owner = owner;
        println!("{:?}", owner);
    }
```

Copy 类型没有方法，只是告诉所有权系统的如上的内容，本质上还是`memcopy`，只能通过 `derive` 实现，且需要满足

* 结构体成员不能是 `&mut` 指针类型
* 元组结构体枚举的所有子类型必须都是 `Copy` 类型
* 必须实现 `Clone` （但是 赋值操作不会使用）
* 结构体本身没有实现 `Drop`

```rust
    {
        // #[derive(Debug, Copy)] //the trait `Copy` may not be implemented for this typerustc(E0204)
        struct S {
            a: String,
            b: String,
        }
    }
    {
        println!("Copy 调用的是 Clone 的实现？");
        #[derive(Debug, Copy)]
        struct S {
            a: i32,
            b: i32,
        }
        impl Clone for S {
            fn clone(&self) -> Self {
                println!("Clone called");
                S {
                    a: self.a.clone(),
                    b: self.b.clone(),
                }
            }
        }
        let owner = S {
            a: 1,
            b: 2
        };
        let new_owner = owner;
        println!("{:?}", owner);
    }
```

`Clone` (`std::clone::Clone`) 只是一个普通的 特质，用来Clone一个实例。可以通过 `derive` 实现，也可以手动实现

### 3、变量覆盖和所有权

```rust
    {
        let s1 = String::from("hello");
        let s1 = String::from("hello"); // 变量覆盖不会带来内存回收，在花括号结束后自动回收
        let s1 = String::from("hello");
    }
```

### 4、所有权和函数

函数的传参和变量赋值类似

```rust
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 被移动到函数内部
                                    // ... 不在合法
    // println!("所有权已转移 {}", s); // 报错
    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 所有权转移到函数
                                    // but i32 被拷贝，所以x仍能访问
                                    // 可以使用x
    println!("标量数据类型直接拷贝，仍然可访问{}", x);

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 在这, some_string 超出作用域 `drop` 被调用. 返回
  // 内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 在这, some_integer 超出作用域. 没有什么发生.
```

函数返回值同样包含所有权转移

```rust
    let s1 = gives_ownership();         // 函数返回值的所有权转移到s1
                                        // value into s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 移动到函数内
                                        // takes_and_gives_back, 返回值
                                        // 移动到s3
    // println!("s2 {}", s2); // 报错
    println!("s3 {}", s3);



fn gives_ownership() -> String {             // gives_ownership 所有权将移动到作用域内
                                             // 返回值所有权将移动到调用者所在作用域

    let some_string = String::from("hello"); // some_string 进入作用域

    some_string                              // some_string 所有权将移动到调用者所在作用域
}

// takes_and_gives_back 将接受一个参数并返回一个参数
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入
                                                      // 作用域

    a_string  // a_string 所有权将移动到调用者所在作用域
}
```

### 5、所有权总结

变量的所有权每次都遵循相同的模式：

* 赋值（函数传参、返回值）给另一个变量，所有权将转移到被赋值变量，原有变量将不可访问
* 当指向堆上数据的变量（所有权转移除外）超出作用域，将会调用 `drop` 回收内存

在不使用引用的情况下，如果需要同时想使用函数参数和返回值，则需要使用元组在返回回来：

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}

```

### 6、引用

为了解决上文的问题，引入引用。

```rust
    {
        let s1 = String::from("hello");

        let len = calculate_length2(&s1);

        println!("The length of '{}' is {}.", s1, len);
    }

fn calculate_length2(s: &String) -> usize {
    s.len()
}
```

以上引用声明的方式不允许修改，因此可以使用`&mut`声明可变引用

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

可变引用存在如下限制

* 对于特定范围内的特定数据，您只能有一个可变引用。
* 这样设计的原因：竞争发生不一致
    * 两个或更多指针同时访问同一数据。
    * 至少有一个指针被用来写入数据。
    * 没有同步数据访问的机制。

```rust
        {
            let mut s = String::from("hello");

            let r1 = &mut s;
            // let r2 = &mut s; // 报错：cannot borrow `s` as mutable more than once at a time

            // println!("{}, {}", r1, r2);
        }
```

可以通过创建作用域解决

```rust
    {
        let mut s = String::from("hello");

        {
            let r1 = &mut s;

        } // r1 goes out of scope here, so we can make a new reference with no problems.

        let r2 = &mut s;
    }
```

同样可变与不可变混用也会导致报错

```rust
    {
        let mut s = String::from("hello");

        let r1 = &s; // no problem
        let r2 = &s; // no problem
        // let r3 = &mut s; // BIG PROBLEM：cannot borrow `s` as mutable because it is also borrowed as immutable

        // println!("{}, {}, and {}", r1, r2, r3);
    }
```

判断的依据是是否同时使用：

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// r1 and r2 are no longer used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

`let mut` 和 `&mut`

* `let mut` 是一个整体，修饰一个变量，表示该变量指向的内存是可以改变的，体现在：
    * 该变量可以重新绑定一个变量
    * 该变量的成员（比如结构体），可以被赋值
    * 该变量的成员可以获取可变引用及 `&mut`
* `&mut` 是取可变引用

```rust
    {
        // mut 和 &mut
        let a = &mut 1;
        // a = &mut 2; // cannot assign twice to immutable variable `a`
        *a = 2;
        let mut b = &mut 1;
        let mut c = 2;
        b = &mut c;
        *b = 3;
    }
```

总结

* 在任意给定时间，要么 只能有一个可变引用，要么 只能有多个不可变引用。
* 以上规则在调用有改变的时候回触发检查

### 7、引用悬空

编辑器保证不会出现引用悬空：通过检测引用作用域必须在变量的作用域及子孙作用域内。

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String { // 报错引用超出变量作用域
    let s = String::from("hello");

    &s
}
```

总结

* 引用必须总是有效。

### 8、切片类型

切片是另一种没有所有权的类型（就是一种引用）。

字符串切片

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

一个例子：查找字符串的第一个单词

```rust
    {
        let mut s = String::from("hello world");
        s.push_str(" 123");
        let word = first_word(&s);

        // s.clear(); // error! 类似引用计数机制，因为 work 引用了 s

        println!("the first word is: {}", word);
    }
    {
        let mut s = String::from("hello world");

        let word = first_word(&s);

        // s.clear(); // error! cannot borrow `s` as mutable because it is also borrowed as immutable

        println!("the first word is: {}", word);
    }

fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

调用 `s.clear();` 报错的原因：

* 在获取一个不可变引用时，就不能使用可变引用了。

### 9、引用使用规则

为了确保引用不会带来很多运行时错误（悬空、竞争），应用有如下规则：

* 引用作用域必须在变量的作用域内（防止悬空）
* 出现一个不可变引用后，将
    * 可变引用或变量将**不允许**调用其声明为 `&mut self` 的方法
* 出现一个可引用后，则
    * **不允许**在同一作用域再声明一个可变引用
    * **不允许**使用上边声明的不可变对象

### 10、引用总结

共享不可变，可变不共享。

* 在任意给定时间，**要么** 只能有一个可变引用，**要么** 只能有多个不可变引用。
* 引用必须总是有效
* 在获取可变引用之后，原变量将不可修改（称为 Frozen）

> http://llever.com/gentle-intro/pain-points.zh.html#a%E5%8F%AF%E5%8F%98%E5%BC%95%E7%94%A8

关于可变引用规则是:

* 一次只有一个可变引用。原因在于，当 到处都是 都是可变性引用，那跟踪他们就很难。在笨蛋小程序中不明显，但在大型代码库中可能会变得糟糕。编译优化方面参见 [Rust 高级编程](https://learnku.com/docs/nomicon/2018/32-alias/4713)
* 进一步的限制是，当已有一个可变引用时，你不能再拥有不可变引用， 否则，任何有这些引用的人都不能保证他们不会改变。 C++也有不可变的引用 (例如`const string&`) ，但是 不能 给你这个保证，因为有人可能在你背后，保留一个string&引用并修改它。

### 11、所有权系统总结

> https://blog.logrocket.com/introducing-the-rust-borrow-checker/

对于某个变量的传递（赋值、函数传参、结构体传参等），Rust 中有三种策略

* 移动 数据 并 放弃所有权（本质上是 copy、被移动的变量在当前作用域不可使用）
    * 未 实现 `Copy` 特质的结构体 和 枚举
* 创建 数据 的 copy
    * 实现 了 `Copy` 特质的结构体 和 枚举 （Copy 本质上 是 `=` 的运算符重载）只有如下条件的结构体可以实现 `Copy`
        * 结构体本身没有实现 `Drop`
        * 结构体内的所有成员均实现了 `Copy`
        * 结构体自身实现了 `Clone`
    * 基本数据类型
* 传递 数据 的 引用

### 12、借用检查器实现原理

* 方式一：基于词法分析的括号作用域解析方法，会造成引用的生命周期过长，检查过于严格
* 方式二：NLL（Non Lexical Lifetime），利用 AST 生成的 MIR（控制流图）找到引用的最后使用地点（Rust 2018 1.31.0 已稳定）

## 五、结构体、枚举和模式匹配（解构）

### 1、定义并实例化结构体

#### （1）基本示例

```rust
// 结构体
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// 方便的构造函数
fn build_user(email: String, username: String) -> User {
    User {
        email, // 相同可以省略
        username,
        active: true,
        sign_in_count: 1,
    }
}

// 元组结构体
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("username123"),
        active: true,
        sign_in_count: 1,
    };
    println!("User: username={} email={} sign_in_count={} active={}", user1.username, user1.email, user1.sign_in_count, user1.active);
    let user1_1 = build_user(String::from("test@example.com"), String::from("test"));
    let user2 = User {
        email: String::from("another@example.com"),
        username: String::from("anotherusername567"),
        ..user1 // 从其他实例中拷贝
    };

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);

    println!("Hello, world!");
}
```

#### （2）结构体与所有权

可以使结构体存储被其他对象拥有的数据的引用，不过这么做的话需要用上 生命周期（lifetimes），这是一个第十章会讨论的 Rust 功能。生命周期确保结构体引用的数据有效性跟结构体本身保持一致。如果你尝试在结构体中存储一个引用而不指定生命周期将是无效的，比如这样：

```rust
struct User {
    username: &str, // 报错
    email: &str, //  报错
    sign_in_count: u64,
    active: bool,
}

fn main() {
    let user1 = User {
        email: "someone@example.com",
        username: "someusername123",
        active: true,
        sign_in_count: 1,
    };
}
```

### 2、方法

```rust
#[derive(Debug)] // 注解
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle { //实现方法
    fn area(&self) -> u32 {
        self.width * self.height
    }

}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
    println!("rect1 is {:#?}", rect1);

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );

    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

* 方法必须定义在结构头上下文（或者是枚举或 trait 对象的上下文）
* 第一个参数总是self，可以是：
    * `&self`
    * `&mut self`
    * `self` 不常见（仅在将当前对象转换为另一个对象）
* 类似于go，rust不使用`->`，只使用`.`，会进行自动解引用，针对 `&self` 和 `&mut self` rust 会自动引用，而手动传递需要手动引用

关于 `self`、`Self`

* `self` 本质上是一个关键字，当第一个参数名为关键字`self` （形式可能为 `&self`、`&mut self`、`self: &Self` 甚至 `self: Box<Self>`） 时，该函数为 该结构体的 方法，可以通过 `.` 号调用（当然通过`结构体名::方法名`、第一个参数传递结构体变量也可以）；否则该函数为静态方法，只能通过`结构体名::方法名`。
* `Self` 当前结构体类型的类型别名（可以理解为 `type Self = 结构体名`），Self 在 声明 `trait` 时，没有指向，一旦 `impl` 编译器将明确其类型。

方法 `self` 变量的几种写法

* `self` 和 `mut self` 都会消费掉自己，只能通过变量调用（不允许通过引用），可变不可便都可以
    * `self` 只允许在方法体内读
    * `mut self` 允许在方法体内写
* `&self` 和 `&mut self` 不会消费自己，只会传引用
    * `&self` 所有类型均可调用（`Self`，`&Self`，`mut Self`，`&mut Self`）
    * `&mut self` 只允许`&mut Self`，`mut Self`（注意可变引用唯一规则） 调用
* 其他编译器特别支持的类型
    * `self: Box<Self>`
    * `self: Rc<Self>`
    * `self: Arc<Self>`
    * `self: Pin<P>` (`P` 可选 `&self`, `&mut self`, `self: Box<Self>`, `self: Rc<Self>`, `self: Arc<Self>`)

示例

```rust
    struct S {
        a: i32
    }
    impl S {
        fn a(self) {
            println!("fn a(self) {}", self.a);
        }
        fn b(mut self) {
            println!("fn b(mut self) {}", self.a);
        }
        fn c(&self) {
            println!("fn c(&self) {}", self.a);
        }
        fn d(&mut self) {
            println!("fn d(&mut self) {}", self.a);
        }
        fn e(self: std::pin::Pin<&Self>) {
            println!("fn e(self: std::pin::Pin<&Self>) {}", self.a)
        }
    }
    {
        let s = S {a:1};
        s.a();
        // s.a();  // use of moved value: `s`
        let s1 = S {a:1};
        let sr = &s1;
        // sr.a(); // cannot move out of `*sr` which is behind a shared reference
    }
    {
        let s = S {a:1};
        s.b();
        // s.b();  // use of moved value: `s`
        let mut s1 = S {a:1};
        let sr = &mut s1;
        // sr.b(); // cannot move out of `*sr` which is behind a mutable reference
    }
    {
        let s = S {a:1};
        s.c();
        s.c();
    }
    {
        let s = S {a:1};
        // s.d();  // cannot borrow `s` as mutable, as it is not declared as mutable
        let mut s = S {a:1};
        s.d();
        s.d();
    }
    {
        let s = S {a:1};
        let sp = std::pin::Pin::new(&s);
        sp.e();
    }
```

### 3、枚举

创建测试项目 `cargo new enums`

基本语法

```rust
// 定义枚举
enum IpAddrKind {
    V4,
    V6,
}

    // 使用枚举
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

* 通过`::`引用枚举

枚举作为函数参数

```rust
// 枚举值作为函数参数
fn route(ip_type: IpAddrKind) {

}

    // 调用参数为枚举方法
    route(four);
```

枚举值作为结构体成员

```rust
// 枚举值作为结构体成员
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

    // 使用枚举结构体
    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };
    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
```

枚举与值绑定

```rust
// 将数值与枚举属性绑定
enum IpAddr2 {
    V4(String),
    V6(String),
}

    let home = IpAddr2::V4(String::from("127.0.0.1"));
    let loopback = IpAddr2::V6(String::from("::1"));
```

标准库中的IPAddr的例子

```rust

// 标准库中ip的封装
struct Ipv4Addr {
    // --snip--
}
struct Ipv6Addr {
    // --snip--
}
enum IpAddr3 {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

更复杂的关联数据与关联方法

```rust
//更复杂的关联数据的例子
enum Message {
    Quit, //没有关联任何数据
    Move { x: i32, y: i32 }, //包含一个匿名结构体
    Write(String), //包含单独一个 String
    ChangeColor(i32, i32, i32), //包含三个 i32
}

// 结构体同样支持方法
impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

    // 方法调用
    let m = Message::Write(String::from("hello"));
    m.call();
```

option实现与无null设计

```rust
// 标准库中的enum实现例子
/*
enum Option<T> {
    Some(T),
    None,
}
*/
    // Option例子
    let some_number = Some(5);
    let some_string = Some("a string");

    let absent_number: Option<i32> = None;

    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    // let sum = x + y; //报错 no implementation for `i8 + std::option::Option<i8>`

```

### 4、模式匹配（解构）

* 支持的模式匹配的语法（`$destructure` 为 匹配/解构）：
    * `let $destructure = var`
    * `match var { $destructure => {} }`
    * `if let $destructure = var {}`
    * `while let $destructure = var {}`
    * 函数参数
    * `for $destructure in var {}`
* Rust 中 `_` 是一个关键字，表示忽略一个变量，`_` 代指的变量无法读取
* 在所有支持模式匹配的语法中，存在一个概念——Refutability（可反驳性）: 模式是否会匹配失效
    * 只可以接收不可反驳性的模式（表达式模式匹配不允许失效）
        * let
        * for
        * 函数参数
    * 只可以接收可反驳性模式（表达式模式匹配允许失败）
        * if let
        * while let

#### （1）简单示例

例子1：枚举类型模式匹配

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

* 一个硬币类型的模式匹配
* 根据类型返回不同值

例子2：带有绑定值的模式匹配

```rust
enum Coin2 {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents2(coin: Coin2) -> u32 {
    match coin {
        Coin2::Penny => 1,
        Coin2::Nickel => 5,
        Coin2::Dime => 10,
        Coin2::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

例子3：匹配Option

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

例子4：通配符

```rust
    let some_u8_value = 0u8;
    match some_u8_value {
        1 => println!("one"),
        3 => println!("three"),
        5 => println!("five"),
        7 => println!("seven"),
        _ => (),
    }
```

* 支持类似scala的通配符

例子5：if let 单条件模式匹配符

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

等价于

```rust
if let Some(3) = some_u8_value {
    println!("three");
}
```

```rust
    let mut count = 0;
    let coin = Coin2::Dime;
    match coin {
        Coin2::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
```

等价于

```rust
    let mut count = 0;
    let coin1 = Coin2::Dime;
    if let Coin2::Quarter(state) = coin1 {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
```

#### （2）全部语法

解构的语法和构造基本一致，`$destructure` 支持的全部语法：

* 字面量 （不可反驳模式）
    * `let x = 1`
    * `match var { 1 =>, _ => }`
* 枚举结构体元组内的匿名值（结构体元组是不可反驳模式，结构体时可反驳模式）
    * `struct S (i32); let S(v) = S(1);`
* 枚举结构体元组内的有名值（结构体元组是不可反驳模式，结构体时可反驳模式）
    * `struct NS {v: i32}; let NS{v: x} = NS{v: 1};`
* 忽略单个/多个值
    * 单个值 `let (a, _, c) = (1, 2, 3);`
    * 多个值 `let (a, ..) = (1, 2, 3);`
* 多个条件（支持 `if let`、`while let`、`match`）
    * `enum E {A(i32), B(i32), C, D}`
    * `use E::*;`
    * `if let C | D = E::C { 1 } else { 2 };`
    * `if let A(x) | B(x) = E::A(1) { x } else { 2 };`
* 取引用和取可变（在 match 语法下，可以 `match &mut var`，这样将会自动解决推断是否使用 ref、mut）
    * 取引用 `let ref x  = 1;` （此时 x 为 `&i32`）
    * 取可变 `let mut x = 1; let ref mut y = x;` （此时 y 为 `&mut i32`）
* match 目前特别支持的语法（未来在 `if let` 和 `while let` 可能稳定）
    * 守卫语法 `match Option::Some(1) { Some(x) if x > 0 => -x, _ => 0, };`
    * 范围匹配（稳定仅支持 `..=`）`match 1 { 0..=10 => true, _ => false };`
    * 变量绑定 `match 1 { x @ 0..=10 | x @ 100..=200 => true, _ => false };`

示例

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    _ => EXPRESSION, // 可选项类似于Default，不加的话，编译器会检测是否已经穷举所有的值
}

// 模式匹配字面值
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}

// 匹配命名变量

    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);

// 多模式匹配

    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }

// 通过 ..= 匹配值的范围

let x = 5;

match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

let x = 'c';

match x {
    'a'..='j' => println!("early ASCII letter"),
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}

// 解构结构体
struct Point {
    x: i32,
    y: i32,
}

    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }

// 解构枚举

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }

// 解构嵌套的结构体和枚举

enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!(
                "Change the color to hue {}, saturation {}, and value {}",
                h,
                s,
                v
            )
        }
        _ => ()
    }

// 结构元组和结构体
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });

```

if let 语法

```rust
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
```

where let 语法

```rust
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
```

for 循环 语法

```rust
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
```

let 语法

```rust
    let PATTERN = EXPRESSION;
    let x = 5;
    let (x, y, z) = (1, 2, 3);
    let (x, _, _) = (1, 2, 3);

// 解构结构体
struct Point {
    x: i32,
    y: i32,
}

    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
```

函数参数的模式提取

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

使用 `_` 忽略整个值

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}


    foo(3, 4);

    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }

    println!("setting is {:?}", setting_value);

    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {}, {}, {}", first, third, fifth)
        },
    }
```

下划线开头的变量不会进行未使用检测

```rust
    let _x = 5;
    let y = 10;

// 下划线开头的变量解构任然会获取所有权
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

// println!("{:?}", s); // 报错

// _ 不会获取所有权

let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);

```

用 `..` 忽略剩余值

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}

    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }

    let numbers = (2, 4, 8, 16, 32);

    // match numbers { // 有歧义报错
    //     (.., second, ..) => {
    //         println!("Some numbers: {}", second)
    //     },
    // }
}
```

匹配守卫提供的额外条件

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}

// 与 | 配合使用

let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"), // 优先级为 (4 | 5 | 6) if y => ...
    _ => println!("no"),
}
```

`@` 绑定 创建变量同时提供匹配条件

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

## 六、函数式语言特性

### 1、闭包

新建项目 `cargo new closures` 编辑 `src/main.rs`

闭包的基本语法

```rust
use std::thread;
use std::time::Duration;


/// 一个生成训练计划的程序
/// 根据用户提供的强度值和随机因子计算接下来要做的事情
fn generate_workout(intensity: u32, random_number: u32) {
    // 用来计算运动项目的次数
    let expensive_closure = |num| { // 一个闭包，类型参数根据下方调用传的参数推断出来
        println!("缓慢计算中...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!(
            "今天, 先做 {} 个 俯卧撑 !",
            expensive_closure(intensity) // 在此处推断出闭包的参数类型和返回值类型
        );
        println!(
            "接下来, 在做 {} 个 仰卧起坐 !",
            expensive_closure(intensity) // 此处传参和返回值必须与第一次调用一致
        );
        // expensive_closure(1.2); // 报错：expected u32, found floating-point number
    } else {
        if random_number == 3 {
            println!("今天休息一下！记住要保持水分！ ");
        } else {
            println!(
                "今天, 跑 {} 分钟步!",
                expensive_closure(intensity)
            );
        }
    }
}

    generate_workout(25, 10);
```

闭包与函数

```rust
fn add_one(a: u32, version: u32) -> u32 {
    fn  add_one_v1   (x: u32) -> u32 { x + 1 }; // 这是定义了一个函数
    let add_one_v2 = |x: u32| -> u32 { x + 1 }; // 定义闭包方式1
    // 以下两种定义必须在作用域内使用才能使编译器推断出参数类型，不适用的将报错，让用户明确声明类型
    let add_one_v3 = |x|             { x + 1 }; // 定义闭包方式2
    let add_one_v4 = |x|               x + 1  ; // 定义闭包方式3
    let add_one_v5 = | |               a + 1  ; // 闭包可以捕获作用域内的变量，但是函数不能
    // fn  add_one_v6   () -> u32       { a + 1 }; // 报错：can't capture dynamic environment in a fn item


    match version {
        1 => add_one_v1(a),
        2 => add_one_v2(a),
        3 => add_one_v3(a),
        4 => add_one_v4(a),
        5 => add_one_v5(),
        _ => a+1,
    }
}

    add_one(1,5);
```

闭包作为结构体成员

```rust

struct Cacher<T>
    where T: Fn(u32) -> u32 // 闭包有三种triat类型参见下文
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}


    let mut add_one_cacher = Cacher::new(|x: u32| x+1);
    println!("{}", add_one_cacher.value(1));
```

闭包与所有权系统（闭包的三种特质）

* 闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：
    * 获取所有权
    * 可变引用借用
    * 不可变引用借用
* 闭包都实现如下几个特质，然后根据调用上下文选择其中的一个特质，下面的self表示对自由变量使用的方式
    * FnOnce(self)
    * FnMut(&mut self)
    * Fn(&self)
* 更多参考 https://tonydeng.github.io/2019/11/09/rust-closure-type/

示例

```rust

fn main() {

    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x; // 强制指定为FnOnce，且将 x 所有权移动到闭包中，当闭包执行完毕x将被回收

    // println!("can't use x here: {:?}", x); // 报错： value borrowed here after moverustc(E0382)

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

* 闭包本质上是一个编译器魔法，实现方式大概如下
    * 分析并捕获闭包变量，构建一个结构体
    * 分析闭包使用情况，将逻辑代码，生成到 `FnOnce`、 `FnMut`、 `Fn` 的实现中
    * 替换所有闭包调用和传递的地方

示例

```rust

        trait FnOnce2<Args> {
            type Output;
            fn call_once(self, args: Args) -> Self::Output;
        }

        struct Adder {
            c: i32,
        }

        impl FnOnce2<(i32, i32)> for Adder {
            type Output = i32;
            fn call_once(self, args: (i32, i32)) -> Self::Output {
                return args.0 + args.1 + self.c
            }
        }
        let c = 2;
        let add = |a: i32, b: i32| a + b + c;
        println!("add(1,2) = {}", add(1,2));
        let add2 = Adder{c};
        println!("add2(1,2) = {}", add2.call_once((1,2)));
```

原理参见：https://zhuanlan.zhihu.com/p/64417628

### 2、迭代器

迭代器的原理与基本使用

* 本质上是一个系列 系统定义 的特质

示例

```rust
        // 迭代器原理与直接使用
        // Iterator 特质定义的方法如下
        // pub trait Iterator {
        //     // 这段代码表明实现 Iterator trait 要求同时定义一个 Item 类型，这个 Item 类型被用作 next 方法的返回值类型。换句话说，Item 类型将是迭代器返回元素的类型。
        //     type Item;
        //     fn next(&mut self) -> Option<Self::Item>;
        //     // 此处省略了方法的默认实现
        // }
        let values = vec![1, 2, 3];
        // iter 必须声明为可变的
        let mut iter = values.iter(); // 声明为 pub fn iter(&self) -> Iter<'_, T>
        println!("迭代器原理与直接使用");
        loop {
            match iter.next() {
                Some(val) => println!("{}", val),
                None => break,
            }
        }
```

可变迭代器在遍历的过程中修改值

```rust
        println!("使用可变迭代器，所有值+1");
        let mut values = vec![1, 2, 3];

        let mut iter = values.iter_mut(); // 声明为 pub fn iter_mut(&mut self) -> IterMut<'_, T>
        // 修改值
        loop {
            match iter.next() {
                Some(val) => {
                    *val = *val + 1;
                },
                None => break,
            }
        }
        // 打印查看
        for val in values.iter() {
            println!("{}", val);
        }
```

迭代器的常见用法

```rust
        // 迭代器常见用法
        // https://rustforce.net/article?id=3874fb6c-30d8-4409-b78f-6d39763074c6
        let v = vec![1, 2, 3];
        println!("同时获取索引号");
        for (i, n) in v.iter().enumerate() {
            println!("v[{}] = {}", i, n);
        }
        println!("最大最小值");
        let max = v.iter().max();
        let min = v.iter().min();
        println!("max = {:?}, min = {:?}", max, min);

        // 迭代器流式处理
        println!("迭代器流式处理，高阶函数（消费）");
        let v1 = vec![1, 2, 3];
        // let v1_iter = v1.iter();
        // let total: i32 = v1_iter.sum();
        println!("sum = {}", v1.iter().sum::<i32>());
        let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
        println!("{:?}", v2);
        let v3: Vec<_> = v1.iter().filter(|&&x| x != 1).collect();
        println!("{:?}", v3);
        let v4 = (0..10).filter(|x| x % 2 == 0).collect::<Vec<_>>();
        println!("{:?}", v4);
        let v5 = v1.iter().fold(0, |acc, x| acc + x * x);
        println!("{}", v5);
        println!("{:?}", v1.iter().rev()); // 翻转
        println!("{:?}", v1.iter().chain( (vec![4, 5, 6]).iter() ).collect::<Vec<_>>()); // 连接
        println!("{:?}", v1.iter().zip( (vec![4, 5, 6]).iter() ).collect::<Vec<_>>()); // 连接
        // 更多工具: https://docs.rs/itertools/0.6.0/itertools
```

for in 语法糖的原理探究

```rust
fn for_in_debug(){
    // https://doc.rust-lang.org/std/iter/index.html
    // let values = vec![1, 2, 3];
    // for val in values {
    //     println!("Got: {}", val);
    // }
    let values = vec![1, 2, 3];
    {
        let result = match IntoIterator::into_iter(values) {
            mut iter => loop {
                let next;
                match iter.next() {
                    Some(val) => next = val,
                    None => break,
                };
                let x = next;
                let () = { println!("{}", x); };
            },
        };
        result
    }
}

fn for_in_debug2() {
    // for val in values {
    //     println!("Got: {}", val);
    // }
    let values:Vec<i32> = vec![1, 2, 3];
    {
        // let mut iter = IntoIterator::into_iter(values);
        // IntoIterator 特质定义内容如下
        // pub trait IntoIterator {
        //     type Item;

        //     type IntoIter: Iterator<Item=Self::Item>;

        //     fn into_iter(self) -> Self::IntoIter;
        // }
        // IntoIterator 的作用 1.返回一个迭代器 2.所有权转移
        let result = loop {
            let next;
            let next_option = iter.next();
            if next_option.is_some() {
                next = next_option.unwrap();
            } else {
                break;
            }
            let x = next;
            let () = { println!("{}", x); };
        };
        result
    }
}
fn main() {
    {
        // for 语法糖原理探究
        // https://doc.rust-lang.org/std/iter/index.html#implementing-iterator
        let values = vec![1, 2, 3];
        // 使用 for 迭代迭代器（注意所有权已经转移）
        println!("for 语法糖原理探究");
        for val in values.iter() { // 这样写不会转移values的所有权，values.iter()返回值的所有权被转移了
            println!("{}", val);
        }
        for val in values {
            println!("{}", val);
        }
        // println!("{:?}", values); // 报错
        // 以上将编译成如下内容
        for_in_debug();
        // 原理如下
        for_in_debug2();
    }
}
```

为结构体实现迭代器

```rust
// 自定义结构体

pub struct MyRange {
    start: i32,
    end: i32,
    step: i32,
}

impl MyRange {
    pub fn new(start: i32, end: i32, step: i32) -> Self {
        MyRange { start, end, step }
    }

    pub fn iter(&self) -> MyRangeIteratorRef<'_> { // '_ 表示匿名生命周期
    // pub fn iter<'a>(&'a self) -> MyRangeIteratorRef<'a> {
        MyRangeIteratorRef {
            range: self,
            now: self.start,
        }
    }
}

// 为自定义结构体实现的迭代器（为引用类型实现）

pub struct MyRangeIteratorRef<'a> {
    range: &'a MyRange,
    now: i32,
}

impl Iterator for MyRangeIteratorRef<'_> {
    type Item = i32;
    fn next(&mut self) -> Option<i32> {
        if self.now == self.range.end {
            None
        } else {
            let r = Some(self.now);
            self.now += self.range.step;
            r
        }
    }
}

impl<'a> IntoIterator for &'a MyRange { // 这个impl是对 MyRange 引用类型的实现
    type Item = i32;
    type IntoIter = MyRangeIteratorRef<'a>;

    fn into_iter(self: &'a MyRange) -> Self::IntoIter { // 这里的self是个引用类型
        self.iter()
    }
}

// 为自定义结构体实现的迭代器（为生命周期转移实现）

pub struct MyRangeIterator{
    range: MyRange,
    now: i32,
}

impl Iterator for MyRangeIterator {
    type Item = i32;
    fn next(&mut self) -> Option<i32> {
        if self.now == self.range.end {
            None
        } else {
            let r = Some(self.now);
            self.now += self.range.step;
            r
        }
    }
}

impl IntoIterator for MyRange { // 这个方法是对 MyRange 本身类型的实现
    type Item = i32;
    type IntoIter = MyRangeIterator;

    fn into_iter(self: MyRange) -> Self::IntoIter { // 这里的self是个本身类型
        MyRangeIterator {
            now: self.start,
            range: self,
        }
    }
}


    // 自定义迭代器
    // 参考 https://stackoverflow.com/questions/30218886/how-to-implement-iterator-and-intoiterator-for-a-simple-struct
    println!("自定义迭代器");
    let r = MyRange::new(0, 10, 1);
    for i in r.iter() {
        println!("{}", i);
    }
    for i in r {  // 调用了 `MyRange::into_iter`
        println!("{}", i);
    }
    // println!("{}", r.start); // 报错：borrow of moved value: `r`
    // 重复问题：暂时无法解决，只能期望 GAT 特性 https://github.com/rust-lang/rfcs/blob/master/text/1598-generic_associated_types.md
    let r2 = &MyRange::new(0, 10, 1);
    for i in r2 { // 调用了 `&MyRange::into_iter`
        println!("{}", i);
    }
    println!("{}", r2.start); // 不报错
```

## 七、模块化系统

### 1、基本概念

一般情况下，一个Cargo项目就是模块系统中的一个包；一个包可以包含多个二进制crate项

* 包（Packages）： Cargo 的一个功能，它允许你构建、测试和分享 crate。
* Crates ：一个模块的树形结构，它形成了库或二进制项目。
* 模块（Modules）和 use： 允许你控制作用域和路径的私有性。
* 路径（path）：一个命名例如结构体、函数或模块等项的方式

### 2、餐馆模拟样例

一个餐馆一般有两个部分：

* 前台 front of house
* 后台 back of house

以下命令将创建一个名为 restaurant 的库

```bash
cargo new --lib restaurant
```

和不带 `--lib` 创建的项目区别在于 `src/main.rs` 变成了 `src/lib.rc`

### 3、定义模块

删除 `src/lib.rc` 原本的内容，填写如下内容

```rust
// mod 定义了一个模块
mod front_of_house {
    // 定义了一个子模块
    mod hosting {
        // 模块的内容
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn server_order() {}

        fn take_payment() {}
    }

}
```

* `mod 模块名 {}` 表示定义了一个有名字的模块
* 花括号内可以定义：子模块、结构体、枚举、常量、特性、或者函数
* 模块树的根节点叫做 `crate`

上述代码定义的模块树结构如下：

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

### 4、路径来使用模块中的项

修改 `src/lib.rs` 如下：

```rust
// Version1： 模拟餐馆前台的模块
// mod 定义了一个模块
mod front_of_house {
    // 定义了一个子模块
    pub mod hosting {
        // 模块的内容
        pub fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    pub mod serving {
        pub fn take_order() {}

        fn server_order() {}

        fn take_payment() {}
    }

}

pub fn eat_at_restaurant() {

    // 绝对路径调用
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径调用：当前内容在 crate 模块，相对于 crate
    front_of_house::hosting::add_to_waitlist();
}

```

* rust中定义的所有元素默认都是私有的。可以通过 `pub` 关键字来使其可见
    * 子模块中的定义可以访问祖宗模块的所有内容
    * 同一模块内的内容可以相互访问
    * 只能访问子孙模块的pub定义的内容（整个路径都必须是pub的）
* 调用支持使用相对路径和绝对路径
    * 绝对路径以 crate 开头（类似于文件系统 `/`）
    * 其他情况为 相对路径，相对于当前所在模块

### 5、使用`super`访问父路径

`super` 类似于 文件系统中的 `../`

在 `src/lib.rc` 中添加

```rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order(); // 调用父模块中的成员
    }

    fn cook_order() {}
}
```

### 6、创建公有的结构体和枚举

在 `src/lib.rs` 模块 `back_of_house` 中添加

```rust
    // 定义了一个公有的结构体
    pub struct Breakfast {
        pub toast: String, // 公有字段
        seasonal_fruit: String, // 默认是私有字段
    }

    // 实现结构体方法
    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }

    // 定义一个公有结构体
    pub enum Appetizer {
        Soup, // 默认是公有
        Salad, // 默认是公有
    }
```

在 `src/lib.rs` 函数 `eat_at_restaurant` 中添加

```rust
    // 夏天订购黑麦面包早餐 Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // 改变主意，想吃什么面包 Change our mind about what bread we'd like
    meal.toast = String::from("Wheat"); // 修改公有字段
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // 如果我们取消注释，则下一行将不会编译；我们不允许
    // 查看或修改随餐提供的时令水果
    // meal.seasonal_fruit = String::from("blueberries"); // 不可以修改

    // 公有的可直接访问
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
```

### 7、使用use引入到作用域

和其它语言类似

`src/lib.rc`

```rust
use crate::front_of_house::hosting;

pub fn eat_at_restaurant1() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}

use crate::front_of_house::hosting::add_to_waitlist; // 直接引入函数

pub fn eat_at_restaurant2() {
    add_to_waitlist();
    add_to_waitlist();
    add_to_waitlist();
}

use std::fmt::Result;
use std::io::Result as IoResult; // 重命名

// 引入并暴露，这样在外部就可以通过 类似 serving::take_order 方式调用
// 相当于在当前作用域定义了 serving 模块并 pub
pub use crate::front_of_house::serving;

use rand::Rng;

use std::collections::HashMap;

use std::{cmp::Ordering, alloc};
use std::io::{self, Write};

use std::collections::*;

```

* `use xxx::xxx::A` 导入路径，导入后，内部就可以 `A` 名使用 `A` 访问其内容
* `use xxx::xxx::A as B` 重命名，内部就可以通过名字B访问A
* `pub use xxx::xxx::A` 外部可以访问A
* `use xxx::xxx` 使用外部包
    * `Cargo.toml` 添加依赖 `rand = "0.5.5"`
    * 此时 `use xxx` 中 `xxx` 就是外部包名
* `use std::xxx` 使用 `std` 包
    * `std` 包和其他外部包使用方式一致
    * `std` 不需要显示引入依赖，是标准库，直接可以使用
* `use std::{cmp::Ordering, alloc};` 一次性引入多个包
* `use std::io::{self, Write};`
    * self表示同时引入`io`
* `use xxx::xxx::*;` 一次性引入全部

### 8、多文件模块

`src/lib.rs` 添加模块声明

```rust
mod front_of_house2;

pub use crate::front_of_house2::hosting as hosting2 ;

pub fn eat_at_restaurant3() {
    hosting2::add_to_waitlist();
    hosting2::add_to_waitlist();
    hosting2::add_to_waitlist();
}

mod front_of_house3;
```

`src/front_of_house2.rs`

```rust
pub mod hosting;
```

`src/front_of_house2/hosting.rs`

```rust
pub fn add_to_waitlist() {}
```

`src/front_of_house3/mod.rs`

```rust
pub mod hosting;
```

`src/front_of_house3/hosting.rs`

```rust
pub fn add_to_waitlist() {}
```

* 使用 `mod xxx;` 或者 `pub mod xxx;` 声明一个模块后，有两种方式实现定义：
    * 方式1：定义文件 `xxx.rs`，文件内直接编写模块定义
    * 方式2：创建目录 `xxx`，创建文件 `xxx/mod.rs`，并在该文件内直接编写模块定义
* 推荐方式（可读性更高）：
    * 针对非叶子模块使用方式2
    * 针对叶子节点使用方式1

### 9、Cargo 和 模块发布

#### （1）自定义构建

在 Rust 中 发布配置（release profiles）是预定义的、可定制的带有不同选项的配置，他们允许程序员更灵活地控制代码编译的多种选项。每一个配置都彼此相互独立。

Cargo 有两个主要的配置：运行 `cargo build` 时采用的 dev 配置和运行 `cargo build --release` 的 release 配置。`dev` 配置被定义为开发时的好的默认配置，`release` 配置则有着良好的发布构建的默认配置。

覆盖 `[profile.*]` 配置

```toml
[profile.dev]
opt-level = 0 # 优化级别

[profile.release]
opt-level = 3
```

#### （2）让模块更好用的建议

**编写文档注释**

* 支持Markdown
* `cargo doc --open` 可以进行文档预览
* `cargo test` 文档注释中的代码可以作为测试样例

例子

```rust
/// 将给定的数字加一
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

**模块文档注释**

文件名: `src/lib.rs`

```rust

//! # My Crate
//!
//! `my_crate` 是一个使得特定计算更方便的
//! 工具集合

/// 将给定的数字加一。
// --snip--
```

**使用 pub use 重导出合适的公有 API**

* 相当于批量导出模块
* 同时给模块一个别名
* 生成的模块在首页会有一个超链接

文件名: `src/lib.rs`

```rust
//! # Art
//!
//! 一个描述美术信息的库。

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

文件名: `src/main.rs`

```rust
use art::PrimaryColor;
use art::mix;

fn main() {
    // --snip--
}
```

#### （3）将 crate 发布到 Crates.io

**创建登录账号**

* 创建 Crates.io 账号
* 查看位于 https://crates.io/me/ 的账户设置页面并获取 API token。
* 接着使用该 API token 运行 `cargo login $token` 命令

**确定元数据**

* `name` 确定模块名（不能重复）
* `license` 许可证
* `license-file` 私有许可证

```toml
[package]
name = "guessing_game"
license = "MIT"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
edition = "2018"
description = "A fun game where you guess what number the computer has chosen."
```

**执行发布**

* `cargo publish`

**发布新版**

* 更新 `version` 字段 遵循 [语义化版本规则](http://semver.org/)
* `cargo publish`

**撤回版本**

* `cargo yank --vers 1.0.1`
* 撤销撤回 `cargo yank --vers 1.0.1 --undo`
* 撤回 并没有 删除任何代码。举例来说，撤回功能并不意在删除不小心上传的秘密信息。如果出现了这种情况，请立即重新设置这些秘密信息。

#### （4）Cargo 自定义扩展命令

Cargo 的设计使得开发者可以通过新的子命令来对 Cargo 进行扩展，而无需修改 Cargo 本身。如果 `$PATH` 中有类似 cargo-something 的二进制文件，就可以通过 cargo something 来像 Cargo 子命令一样运行它。像这样的自定义命令也可以运行 cargo --list 来展示出来。能够通过 cargo install 向 Cargo 安装扩展并可以如内建 Cargo 工具那样运行他们是 Cargo 设计上的一个非常方便的优点！

### 10、工作空间

创建工作空间，并创建配置文件 `Cargo.toml`

```bash
mkdir workspace
cd workspace
vim Cargo.toml
```

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

创建项目

```bash
cargo new adder
cargo new add-one --lib
```

编写库 `add-one` 程序依赖配置 `add-one/Cargo.toml`

```toml
[dependencies]
rand = "0.5.5"
```

编写库 `add-one` 库文件 `add-one/src/lib.rs`

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

编写 `adder` 程序依赖配置 `adder/Cargo.toml`

```toml
[dependencies]

add-one = { path = "../add-one" }
```

编写 `adder` 程序入口 `adder/src/main.rs`

```rust
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

在 `workspace` 目录中运行 编译 调试

```bash
cargo test
cargo test -p add-one
cargo run -p adder
cargo build
```

最终目录结构如下

```
.
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
├── adder
│   ├── Cargo.toml
│   └── src
└── target
    └── debug
```

### 11、Cargo 如何解决 版本冲突

[译文](https://juejin.im/post/6844903833844318222)
[原文](https://stephencoakley.com/2019/04/24/how-rust-solved-dependency-hell)

Rust 通过编译符号重命名来解决这个问题（如果是 Java 实现的话应该就是用不同的 ClassLoader 加载两次），这样造成几个问题

* 同一个包的变量不能相互传递（因为本质上不是相同的编译结果）
* 编译产物尺寸会变大
* 依赖静态全局变量的库可能会失效

## 八、常见的集合

### 1、Vector

```rust
    // 1. Vector 可变数组

    // 通过构造函数创建
    let v: Vec<i32> = Vec::new();
    // 通过宏创建，可以推断出类型
    let v = vec![1, 2, 3];

    // 更新Vector
    let mut v = Vec::new(); // 必须是不可变，否则报错
    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);

    // 垃圾回收
    {
        let v = vec![1,2,3,4,5];
        // 处理变量
    } //  <- 这里 v 离开作用域并被丢弃，包括内部的整数元素

    // 读取元素
    let v = vec![1,2,3,4,5];
    let third: &i32 = &v[2];
    println!("第三个元素是 {}", third);
    match v.get(2) { // get 返回一个Option
        Some(third) => println!("第三个元素是 {}", third),
        None => println!("没有第三个元素"),
    }

    let v = vec![1, 2, 3, 4, 5];
    // 使用 [] 访问不存在的元素将触发 panic
    // let does_not_exist = &v[100];
    // 使用get会返回一个None 不会触发异常
    let does_not_exist = v.get(100);

    let mut v = vec![1, 2, 3, 4, 5];
    let first = &v[0];
    // 此句将编译报错，因为上一句已经将 可变的v借用给不可便的first了
    // 原因是：v是可变数组，push可能触发内存分配，这样会破坏first的引用，导致引用悬空
    // v.push(6);
    println!("The first element is: {}", first);

    // 元素遍历
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{}", i);
    }

    // 遍历过程中改变元素值
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }

    // 使用枚举来存储多种类型
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];

    // 访问并修改值
    let mut v = vec![1, 2, 4];
    if v[2] != 3 {
        v[2] = 3;
    }
```

### 2、字符串

```rust
    // 2. String 可变字符串
    // Rust 中的字符串常用的有两种：
    //   str rust核心字符串，字面量字符串类型，utf8编码
    //   String 标准库字符串，可变字符串，utf8编码
    // 除了以上两种还有其他字符串实现，比如：OsString、OsStr、CString 和 CStr

    // 创建一个空的String字符串，通过构造函数
    let mut s = String::new();

    // 通过字面量字符串&str创建字符串String
    let data = "initial contents";
    let s = data.to_string();
    // 该方法也可直接用于字符串字面值：
    let s = "initial contents".to_string();

    // 以上方法等价于 String::from
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");

    // 更新字符串
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2); // 并不会获取s2所有权
    println!("s2 is {}", s2);

    let mut s = String::from("lo");
    s.push('l'); // 添加一个字符

    // 字符串拼接
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    // + 的签名类似与： fn add(self, s: &str) -> String {
    // 注意 s1 被移动了，不能继续使用，因为self声明类型为 String 而不是 引用，所有权转移了
    // 同时 s2 使用 解引用强制多态 转换为 &str 类型（&s2[..]）所以s2仍能使用
    let s3 = s1 + &s2;

    // 使用 format! 宏进行拼接
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{}-{}-{}", s1, s2, s3);

    // String 不支持 索引
    let s1 = String::from("hello");
    // let h = s1[0]; // 报错

    // String 内部实现为 Vec<u8> 的封装，编码方式为utf8
    // 不提供索引的原因是：utf8是边长编码。无法精确定位字符
    // str支持索引，slice，但是遇到多字节字符可能引发panic
    let hello = "Здравствуйте";
    let s = &hello[0..4]; // 实际上返回 Зд
    // let s = &hello[0..1]; // 报错，因为返回的字符串是不合法的utf8编码

    // 遍历字符串
    for c in "नमस्ते".chars() {
        println!("{}", c);
    }
```

* `String` 和 `&str` 均为 utf8为边长编码，索引的时间复杂度为`O(n)`，不抛出异常索引方式为 `.chars().nth(1)`

### 3、HashMap

```rust
    // 3. HashMap
    // key需要实现 Hash 和 Eq 特质，才能全功能使用
    // 创建
    use std::collections::HashMap;
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    // 通过 zip创建
    let teams  = vec![String::from("Blue"), String::from("Yellow")];
    let initial_scores = vec![10, 50];
    let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();

    // 哈希 map 和所有权
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");
    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // 这里 field_name 和 field_value 不再有效，
    // 尝试使用它们看看会出现什么编译错误！
    // println!("{}", field_name); // 报错
    // println!("{}", field_value); // 报错

    // 访问HashMap中的值
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    let team_name = String::from("Blue");
    let score = scores.get(&team_name);

    // 遍历HashMap
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }

    // 更新哈希 map
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);
    println!("{:?}", scores);

    // 只在键没有对应值时插入
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);
    println!("{:?}", scores);

    // 根据旧值更新一个值
    let text = "hello world wonderful world";
    let mut map = HashMap::new();
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }
    println!("{:?}", map);
```

## 九、泛型、特质

### 1、泛型

rust 的 泛型类型实现方式C++中的模板，在编译时会被具象化出二进制代码（单态化（monomorphization））。（与Java泛型不同，Java泛型运行时擦除实现）

* 单态化优缺点
    * 运行时没有额外性能损失
    * 编译产物体积相对较大
* 类型擦除
    * 运行时有额外的性能损失
    * 编译产物体积相对较小

实验代码

```rust
    // 结构体使用泛型声明
    struct Point<T> {
        x: T,
        y: T,
    }
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };

    // 结构体使用多个泛型
    struct Point1<T, U> {
        x: T,
        y: U,
    }
    let both_integer = Point1 { x: 5, y: 10 };
    let both_float = Point1 { x: 1.0, y: 4.0 };
    let integer_and_float = Point1 { x: 5, y: 4.0 };

    // 枚举中使用泛型
    enum Option<T> {
        Some(T),
        None,
    }
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }

    // 为泛型实现方法
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }

    // 实现一个泛型的具象化方法
    // 本例为浮点类型的点实现计算欧拉距离的方法
    impl Point<f32> {
        fn distance_from_origin(&self) -> f32 {
            (self.x.powi(2) + self.y.powi(2)).sqrt()
        }
    }

    // 为泛型结构体实现一个泛型方法
    impl<T, U> Point1<T, U> {
        fn mixup<V, W>(self, other: Point1<V, W>) -> Point1<T, W> {
            Point1 {
                x: self.x,
                y: other.y,
            }
        }
    }
```

### 2、特质（trait）

类似go语言的接口

```rust
use std::fmt::Display;

fn main() {
    // 结构体使用泛型声明
    struct Point<T> {
        x: T,
        y: T,
    }
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };

    // 结构体使用多个泛型
    struct Point1<T, U> {
        x: T,
        y: U,
    }
    let both_integer = Point1 { x: 5, y: 10 };
    let both_float = Point1 { x: 1.0, y: 4.0 };
    let integer_and_float = Point1 { x: 5, y: 4.0 };

    // 枚举中使用泛型
    enum Option<T> {
        Some(T),
        None,
    }
    enum Result<T, E> {
        Ok(T),
        Err(E),
    }

    // 为泛型实现方法
    impl<T> Point<T> {
        fn x(&self) -> &T {
            &self.x
        }
    }

    // 实现一个泛型的具象化方法
    // 本例为浮点类型的点实现计算欧拉距离的方法
    impl Point<f32> {
        fn distance_from_origin(&self) -> f32 {
            (self.x.powi(2) + self.y.powi(2)).sqrt()
        }
    }

    // 为泛型结构体实现一个泛型方法
    impl<T, U> Point1<T, U> {
        fn mixup<V, W>(self, other: Point1<V, W>) -> Point1<T, W> {
            Point1 {
                x: self.x,
                y: other.y,
            }
        }
    }

    // 定义一个 特质
    pub trait Summary {
        fn summarize(&self) -> String;
    }

    // 为类型实现 trait
    pub struct NewsArticle {
        pub headline: String,
        pub location: String,
        pub author: String,
        pub content: String,
    }

    impl Summary for NewsArticle {
        fn summarize(&self) -> String {
            format!("{}, by {} ({})", self.headline, self.author, self.location)
        }
    }

    pub struct Tweet {
        pub username: String,
        pub content: String,
        pub reply: bool,
        pub retweet: bool,
    }

    impl Summary for Tweet {
        fn summarize(&self) -> String {
            format!("{}: {}", self.username, self.content)
        }
    }

    // 特质默认实现
    pub trait Summary1 {
        fn summarize(&self) -> String {
            String::from("(Read more...)")
        }
    }
    pub struct NewsArticle1 {
        pub headline: String,
        pub location: String,
        pub author: String,
        pub content: String,
    }
    impl Summary1 for NewsArticle1 {
    }
    let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from("The Pittsburgh Penguins once again are the best
        hockey team in the NHL."),
    };
    println!("New article available! {}", article.summarize());

    // trait 作为函数参数
    pub fn notify(item: impl Summary) {
        println!("Breaking news! {}", item.summarize());
    }
    // trait 作为函数参数（Trait Bound）
    pub fn notify2<T: Summary>(item: T) {
        println!("Breaking news! {}", item.summarize());
    }

    pub fn notify3<T: Summary>(item1: T, item2: T) {
    }

    // 通过 + 指定多个 trait bound
    pub fn notify4(item: impl Summary + Display) {
    }

    pub fn notify5<T: Summary + Display>(item: T) {
    }

    // 使用where语法
    trait Debug{}
    fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {0}
    fn some_function1<T, U>(t: T, u: U) -> i32
        where T: Display + Clone,
              U: Clone + Debug
    {
        0
    }

    // trait 作为函数返回值
    fn returns_summarizable() -> impl Summary { // 只能返回单一类型
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from("of course, as you probably already know, people"),
            reply: false,
            retweet: false,
        }
    }

    // 以下代码报错：因为只能返回单一类型
    // fn returns_summarizable1(switch: bool) -> impl Summary {
    //     if switch {
    //         NewsArticle {
    //             headline: String::from("Penguins win the Stanley Cup Championship!"),
    //             location: String::from("Pittsburgh, PA, USA"),
    //             author: String::from("Iceburgh"),
    //             content: String::from("The Pittsburgh Penguins once again are the best
    //             hockey team in the NHL."),
    //         }
    //     } else {
    //         Tweet {
    //             username: String::from("horse_ebooks"),
    //             content: String::from("of course, as you probably already know, people"),
    //             reply: false,
    //             retweet: false,
    //         }
    //     }
    // }

    // 例子：实现 largest 函数
    fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
        let mut largest = list[0];
        for &item in list.iter() {
            if item > largest {
                largest = item;
            }
        }
        largest
    }
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);

    // 使用 trait bound 有条件地实现方法
    struct Pair<T> {
        x: T,
        y: T,
    }
    impl<T> Pair<T> {
        fn new(x: T, y: T) -> Self {
            Self {
                x,
                y,
            }
        }
    }
    impl<T: Display + PartialOrd> Pair<T> {
        fn cmp_display(&self) {
            if self.x >= self.y {
                println!("The largest member is x = {}", self.x);
            } else {
                println!("The largest member is y = {}", self.y);
            }
        }
    }
}
```

调用 trait 上的方法，必须手动引入 trait

```rust
        let a = "test".to_string();
        let b = "2";
        let c = a + b;
        use std::ops::Add;
        std::string::String::add(c, "3");
```

trait 、 self 、 impl 及 自定取解引用

```rust
trait T {
    fn t_self(self);
    fn t_ref_self(&self);
}
struct S {
    a: i32,
}
impl T for S {
    fn t_self(self) {
    // fn t_self(self: S) {
        println!("impl T for S t_self {}", self.a);
    }
    fn t_ref_self(&self) {
    // fn t_ref_self(self: &S) {
        println!("impl T for S t_ref_self, {}", (*self).a);
    }
}
impl T for &S {
    fn t_self(self) {
        println!("impl T for &S t_self {}", (*self).a);
    }
    fn t_ref_self(&self) {
        println!("impl T for &S t_ref_self {}", (*(*self)).a);
    }
}
impl T for &mut S {
    fn t_self(self) {
        (*self).a = 5;
        println!("impl T for &mut S t_self {}", (*self).a);
    }
    fn t_ref_self(&self) {
        let tmp: & &mut S = self;
        let tmp1: &S = *tmp;
        println!("impl T for &mut S t_ref_self {}", (*tmp1).a);
    }
}
{
    let s = S {a:1};
    s.t_self();  // impl T for S t_self 1
    let s = S {a:1};
    s.t_ref_self();  // impl T for S t_ref_self, 1
    &s.t_ref_self();  // impl T for S t_ref_self, 1
}
{
    let s = S {a:1};
    T::t_self(&s);  // impl T for &S t_self 1
    T::t_ref_self(&&s);  // impl T for &S t_ref_self 1
}
{
    let mut s = S {a:1};
    T::t_self(&mut s);  // impl T for &mut S t_self 5
    let mut s = S {a:1};
    T::t_ref_self(& &mut s);  // impl T for &mut S t_ref_self 1s
}
```

高级Trait 参见 第十七.2

### 3、标准库常用的 trait

* Display 和 Debug，主要用于 `println!`
    * Display 一般由用户自己实现，使用 `{}` 可以打印出来，实现了 Display 将自动实现 `ToString`
    * Debug 一般由 `derive` 宏生成，使用 `{:?}` 或者 `{:#?}`，用于调试，所有公开的 结构体建议都实现该特质
* PartialOrd / Ord / PartialEq / Eq
    * PartialOrd 为偏序比较，排序结果与初始状态有关，比如浮点数比较；比较结果可能是 None，未定义
    * Ord 全序比较，比如整数
* Default 类似于默认构造函数

`std::marker` 模块内的特质，作为编译器标记，没有实现

* `Sized`，所有编译期尺寸确定的类型，均自动实现该特质

## 十、生命周期

rust 特有，生命周期也是一种泛型，用来防止引用悬空，辅助借用检查器进行检查。

### 1、显示使用

每个引用都有生命周期，声明方式如下：

```rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
}
```

样例

```rust
    // 1. 生命周期避免了悬垂引用
    // {
    //     let r; // 不可便变量只允许赋值一次
    //     {
    //         let x = 5;
    //         r = &x; // Error `x` does not live long enough，生命周期不同借用检查器不允许
    //     }
    //     println!("r: {}", r);
    // }

    // 2. 生命周期一致才可以
    {
        let x = 5;            // ----------+-- 'b
                              //           |
        let r = &x;           // --+-- 'a  |
                              //   |       |
        println!("r: {}", r); //   |       |
                              // --+       |
    }

    // 3. 函数中的泛型生命周期
    // 'a 表示一个生命周期注解，表示x、y参数和返回值必须拥有相同的生命周期
    // 调用时，'a 取x、y最小的生命周期
    fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
    // 正确调用，'a = string2 的生命周期
    {
        let string1 = String::from("long string is long");
        {
            let string2 = String::from("xyz");
            let result = longest(string1.as_str(), string2.as_str());
            println!("The longest string is {}", result);
        }
    }
    // // 错误调用，'a = string2 的生命周期，result是 'a 的父生命周期
    // {
    //     let string1 = String::from("long string is long");
    //     let result;
    //     {
    //         let string2 = String::from("xyz");
    //         result = longest(string1.as_str(), string2.as_str());
    //     }
    //     println!("The longest string is {}", result);
    // }

    // 4. 结构体定义中的生命周期注解
    // 这个注解意味着 ImportantExcerpt 的实例不能比其 part 字段中的引用存在的更久。
    struct ImportantExcerpt<'a> {
        part: &'a str,
    }
    impl<'a> ImportantExcerpt<'a> {
        fn level(&self) -> i32 {
            3
        }
        fn announce_and_return_part(&self, announcement: &str) -> &str {
            println!("Attention please: {}", announcement);
            self.part
        }
    }
    {
        let novel = String::from("Call me Ishmael. Some years ago...");
        let first_sentence = novel.split('.')
            .next()
            .expect("Could not find a '.'");
        let i = ImportantExcerpt { part: first_sentence };
    }
```

### 2、生命周期省略规则

* 第一条规则是每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，fn `foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
* 第二条规则是如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
* 第三条规则是如果方法有多个输入生命周期参数，不过其中之一因为方法的缘故为 `&self` 或 `&mut self`，那么 self 的生命周期被赋给所有输出生命周期参数。第三条规则使得方法更容易读写，因为只需更少的符号。

### 3、静态生命周期

'static，其生命周期能够存活于整个程序期间。所有的字符串字面值都拥有 'static 生命周期，我们也可以选择像下面这样标注出来：

```rust
let s: &'static str = "I have a static lifetime.";
```

### 4、结合泛型类型参数、trait bounds 和生命周期

```rust
    fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
        where T: Display
    {
        println!("Announcement! {}", ann);
        if x.len() > y.len() {
            x
        } else {
            y
        }
    }
```

### 5、生命周期关系

* 生命周期 `'a` 比 `'b` 更长或相等，则记为 `'a :'b`
* 始终满足 `'static: 'x`

以下两种写法是基本相同的

```rust
        fn test1<'a>(arg: &'a i32) -> &'a i32 { arg }
        fn test2<'a, 'b>(arg: &'a i32) -> &'b i32 where 'a: 'b { arg }
```

### 6、引用生命周期总结

* 核心目的：避免出现引用悬空（引用指向 free 的变量）
* 核心规则：**变量**的生命周期必须 大于等于 **引用**的生命周期
* 非函数场景：通过 变量声明顺序 和 作用域（花括号）即可判断
* 函数场景：传递或者返回引用时，编译器不知道引用的生命周期的关系，所以需要手动指定
* 结构体方法场景：结构体存在引用时，在实现方法时编译器不知道 引用成员 和 函数方法的引用参数和引用返回值的关系
* 函数/方法生命周期检查时机：函数声明时（函数体内检查）和 函数调用时（参数和返回值）
* 函数/方法生命周期调用检查原理：本质上一种泛型约束，调用时，会利用传递参数的声明周期情况特化生命周期参数并进行约束检查
* 一般情况下，函数返回值的生命周期小于等于参数的生命周期，`'static` 除外
* 为了方便使用，rust 可以省略手动指定生命周期，编译器按照一定规则生成

## 十一、测试

### 1、单元测试

* 使用 `#[cfg(test)] ` 注解的模块
    * 在模块中使用 `#[test]` 注解的函数为单元测试函数
* 代码写在 `src` 目录下

创建一个测试模块 `cargo new rusttest --lib`

编写 `src/lib.rc` 文件

```rust
pub fn add_two(base: i32) -> i32{
    base + 2
}

// src 内部的一般是单元测试
#[cfg(test)] // 只在执行 cargo test 时才编译和运行测试代码 cargo build 将忽略该模块
mod tests {
    #[test] // 表示当前函数为测试函数
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }

    #[test]
    #[should_panic] // 抛出所有异常都将测试通过
    fn should_panic() {
        panic!("应该抛出异常");
    }

    #[test]
    #[should_panic(expected = "应该抛出异常")] // 只期望指定的异常
    fn should_special_panic() {
        panic!("应该抛出异常");
    }

    #[test]
    fn should_fail() {
        assert_ne!(1, 2);
        assert!(true, "测试失败的消息");
        assert_eq!(1, 2, "{} == {} 测试失败", 1, 2);
    }

    #[test]
    fn result() -> Result<(), String> { // 使用Result报告测试结果，如果返回Ok测试通过，返回Err测试失败
        if 2 + 2 == 5 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }

    #[test]
    #[ignore]  // 忽略该测试
    fn ignore() {
        assert!(true);
    }
}
```

运行测试

```bash
cargo test  # 运行测试
cargo test --help  # 查看相关参数
cargo test -- --test-threads=1  # 测试二进制文件的测试线程数
cargo test -- --nocapture  # 不显示println! 命令行输出
cargo test it_works  # 运行指定的测试
cargo test should  # 运行指定的测试包含should的测试
cargo test -- --ignored  # 运行`#[ignore]`的测试函数
```

* `--` 分隔符之后的参数将被传递到测试二进制文件

### 2、集成测试

创建 `tests` 目录

编写测试文件 `tests/mytest.rs`

```rust
// test 目录只能测试src/lib.rs中声明的包，不能测试src/main.rs
// use rusttest; // 此种方式也可以
// 声明被测试的外部 crate，就像其他使用该 crate 的程序要声明的那样。
extern crate rusttest;

mod common;


#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, rusttest::add_two(2));
}

```

测试使用的模块建议使用目录方式创建，创建文件 `tests/common/mod.rs`

```rust
pub fn setup() {
    // 编写特定库测试所需的代码
}
```

运行集成测试

```bash
cargo test it_adds_two # 运行指定测试
cargo test --test mytest # 运行指定测试文件中的测试
```

## 十二、最佳实践

### 1、二进制项目的组织结构

main 函数负责多个任务的组织问题在许多二进制项目中很常见。所以 Rust 社区开发出一类在 main 函数开始变得庞大时进行二进制程序的关注分离的指导性过程。这些过程有如下步骤：

* 将程序拆分成 main.rs 和 lib.rs 并将程序的逻辑放入 lib.rs 中。
* 当命令行解析逻辑比较小时，可以保留在 main.rs 中。
* 当命令行解析开始变得复杂时，也同样将其从 main.rs 提取到 lib.rs 中。

经过这些过程之后保留在 main 函数中的责任应该被限制为：

* 使用参数值调用命令行解析逻辑
* 设置任何其他的配置
* 调用 lib.rs 中的 run 函数
* 如果 run 返回错误，则处理这个错误

### 2、newtype 模式封装细节

* `Millimeters` 和 `Meters` 封装了一个 `i32`
* 本质上创建一个新的 struct 内部持有一个成员，这个类型用来
    * 表示一个值的单元
    * 确保某值不被混淆
    * 抽象掉一些类型的实现细节
    * 隐藏其内部的泛型类型

## 十三、例子：grep

创建项目 `cargo new minigrep`

编写主逻辑 `src/lib.rs`

```rust
use std::fs;
use std::error::Error;
use std::env;


// 命令行输入结构体
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}

impl Config {
    // 构造函数
    pub fn new(args: &[String]) -> Result<Config, &'static str> { // 使用Result包裹返回
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();
        let case_sensitive = env::var("CASE_INSENSITIVE").is_err(); // 读取环境变量

        Ok(Config { query, filename, case_sensitive })
    }

    // 使用迭代器的构造函数
    pub fn new1(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}


pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.filename)?; // ? 传递出错误

    // println!("With text:\n{}", contents);

    let results = if config.case_sensitive { // 根据是否区分大小写调用函数
        search(&config.query, &contents)
    } else {
        search_case_insensitive(&config.query, &contents)
    };

    for line in results { // 输出结果
        println!("{}", line);
    }

    Ok(())
}

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> { // 返回结果生命周期与content一致
    // let mut results = Vec::new();
    // // 遍历每一行进行判断
    // for line in contents.lines() {
    //     if line.contains(query) {
    //         results.push(line);
    //     }
    // }

    // results

    // 性能上与上面的实现没有额外损失
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}

// 单元测试： TDD 应该优先实现单元测试函数
#[cfg(test)]
mod tests {
    use super::*;

    // 测试用例：对不区分大消息进行测试
    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

编写入口函数 `src/main.rs`

```rust
use std::env;
use minigrep::Config;
use std::process;


/**
 * 实现一个类似grep的命令行工具
 */
fn main() {
    // println!("Hello, world!");
    // 读取命令行参数
    let args: Vec<String> = env::args().collect(); // 读取命令行参数
    // println!("{:?}", args);

    // 解析命令行参数
    let config = Config::new(&args).unwrap_or_else(|err| { // 闭包函数处理异常
        eprintln!("Problem parsing arguments: {}", err); // 输出到标准出错流
        process::exit(1);
    });

    // println!("Searching for {}", config.query);
    // println!("In file {}", config.filename);

    // 执行主函数
    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

## 十四、智能指针

### 1、Rust结构体内存情况

* 类似与C语言Rust结构体必须大小确定，不允许直接递归定义类型
* 枚举类型存在4字节的类型标签

示例

```rust
// 以下结构体/枚举占用的空间为 32 字节（类型标签4字节，最长成员Write(String) 24，内存对齐4字节）
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

    // Rust 结构体内存大小
    println!("Message Size is {}", std::mem::size_of::<Message>());
    println!("String Size is {}", std::mem::size_of::<String>());
```

### 2、Box 类型——实现递归定义类型

* `Box` 分配包裹的变量将分配到堆内
* `Box` 实现了 `Deref` 特质将其当做引用使用

例子

```rust
// rust 结构体必须能计算出结构体的确定占用内存打下（类似于C的结构体）
// 所以递归类型需要使用Box或者引用

// 以下实现一个函数式的List

enum List {
    // Cons(i32, List), // 报错 recursive type `List` has infinite size
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

    // Box<T>类型
    println!("List Size is {}", std::mem::size_of::<List>());
    println!("Box<List> Size is {}", std::mem::size_of::<Box<List>>());

    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));

    // Box 类型实现了Deref特质，所以允许使用 *y 解引用符
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
```

### 3、通过 Deref trait 将智能指针当作常规引用处理

```rust
    // Box 类型实现了Deref特质，所以允许使用 *y 解引用符
    let x = 5;
    let y = Box::new(x)

    assert_eq!(5, x);
    assert_eq!(5, *y)
```

自定义MyBox并实现Box类似解引用运算符的支持

```rust

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

    // 自定义结构体实现 Deref特质
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // 编译成 *(y.deref())

    fn p(t: &i32) {
        println!("{}", t)
    }
    let ry:&MyBox<i32> = &y;
    p(ry);
```

函数和方法的隐式解引用强制多态（本质上类似于 Scala 隐式类型转换，参考上边的 `p` 函数）

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

    let m = MyBox::new(String::from("Rust"));
    // 以下三者等价
    hello(&m); // 效率最高（编译期分析最终引用，直接传递）
    hello(m.deref().deref());
    hello(&(*m)[..]);
```

解引用强制多态如何与可变性交互

类似于如何使用 Deref trait 重载不可变引用的 `*` 运算符，Rust 提供了 DerefMut trait 用于重载可变引用的 `*` 运算符。
Rust 在发现类型和 trait 实现满足三种情况时会进行解引用强制多态：

* 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
* 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
* 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

头两个情况除了可变性之外是相同的：第一种情况表明如果有一个 `&T`，而 `T` 实现了返回 `U` 类型的 `Deref`，则可以直接得到 `&U`。第二种情况表明对于可变引用也有着相同的行为。

第三个情况有些微妙：Rust 也会将可变引用强转为不可变引用。但是反之是 不可能

### 4、使用 Drop Trait 运行清理代码

`std::ops::Drop` 在变量被回收时（前一刻）调用，编译器保证调用且只一次

```rust
// Drop 特质，在超出作用域后执行清理操作
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("清理 CustomSmartPointer 的 data `{}`!", self.data);
    }
}

    // Drop 特质
    {
        let c = CustomSmartPointer { data: String::from("my stuff") };
        let d = CustomSmartPointer { data: String::from("other stuff") };
        println!("CustomSmartPointers 被创建.");
    }

    // 通过 std::mem::drop 提早丢弃值
    {
        let c = CustomSmartPointer { data: String::from("some data") };
        println!("CustomSmartPointer 被创建");
        // c.drop(); // explicit destructor calls not allowed 不允许显式的析构函数调用
        drop(c); // 本质上就是个空函数 pub fn drop<T>(_x: T) { } 利用所有权转移实现析构😂
        println!("CustomSmartPointer 在这个作用域之前就被Drop了");
    }
```

### 5、`Rc<T>` 引用计数智能指针

```rust

// 使用引用计数实现
enum List2 {
    Cons2(i32, Rc<List2>),
    Nil2,
}

use crate::List2::{Cons2, Nil2};

    // 引用计数指针Rc<T> （仅单线程使用、避免循环引用）
    // Box 不支持多个变量持有同一份数据的所有权
    // let a = Cons(5,
    //     Box::new(Cons(10,
    //         Box::new(Nil))));
    // let b = Cons(3, Box::new(a));
    // let c = Cons(4, Box::new(a)); // 报错 use of moved value: `a`
    let a = Rc::new(Cons2(5, Rc::new(Cons2(10, Rc::new(Nil2)))));
    let b = Cons2(3, Rc::clone(&a));
    let c = Cons2(4, Rc::clone(&a));

    {
        let a = Rc::new(Cons2(5, Rc::new(Cons2(10, Rc::new(Nil2)))));
        println!("创建 a 后的引用计数 = {}", Rc::strong_count(&a)); // 1
        let b = Cons2(3, Rc::clone(&a));
        println!("创建 b 后的引用计数 = {}", Rc::strong_count(&a)); // 2
        {
            let c = Cons2(4, Rc::clone(&a));
            println!("创建 c 后的引用计数 = {}", Rc::strong_count(&a)); // 3
        }
        println!("c 离开作用域后的引用计数 = {}", Rc::strong_count(&a)); // 2
    }
```

### 6、`RefCell<T>` 和内部可变性模式

使用 `RefCell<T>` 实现运行时借用规则检查以实现内不可变性的例子

```rust

// 例子： 测试mock

// 这是待测试对象依赖的外部对象
pub trait Messenger {
    fn send(&self, msg: &str);
}

// 下面是待测试对象：检查消息体大小，如果消息体大小超过阈值，则使用Messenger发送一个消息
pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
             self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        }
    }
}

// 下面是一个Messenger的Mock实现

#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        // sent_messages: Vec<String>, // 此种方案将报错
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            // self.sent_messages.push(String::from(message));  // 使用sent_messages: Vec<String>, 将报错 cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference(E0596)
            // 解决方案1：（不推荐）修改send方法的&self为&mut self（需要修改特质声明和调用者）
            // 解决方案2：使用 RefCell<T> 保证对外不可变，对内可变，如下
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }

}
```

`RefCell<T>` 原理探究

* 标准库实现，不是编译器实现，不是语言核心
* 运行时借用规则检查
* 运行时违反规则将触发 panic
* 有额外的运行时开销

示例

```rust
    // RefCell<T> 在运行时检查借用规则，必须满足借用规则
    {
        let sent_messages:RefCell<Vec<String>> = RefCell::new(vec![]);
        let mut one_borrow = sent_messages.borrow_mut();
        // let mut two_borrow = sent_messages.borrow_mut(); // 编译不报错，运行时触发panic，因为违反了只有一个可变引用 already borrowed: BorrowMutError

        one_borrow.push(String::from("a"));
        // two_borrow.push(String::from("b"));
    }
    {
        // 同样不能同时借出可变和不可变引用
        let sent_messages:RefCell<Vec<String>> = RefCell::new(vec![]);
        let mut one_borrow = sent_messages.borrow_mut();
        // let mut two_borrow = sent_messages.borrow(); // 编译不报错，运行时触发panic，因为已经借出不可便引用了 already mutably borrowed: BorrowError

        one_borrow.push(String::from("a"));
        // println!("{}", two_borrow.len());
    }
```

实现拥有可变List（结合 `Rc<T>` 和 `RefCell<T>` 来拥有多个可变数据所有者）

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List3 {
    Cons3(Rc<RefCell<i32>>, Rc<List3>),
    Nil3,
}

use crate::List3::{Cons3, Nil3};

    // 结合 Rc<T> 和 RefCell<T> 来拥有多个可变数据所有者
    // 实现拥有可变List
    {
        let value = Rc::new(RefCell::new(5));

        let a = Rc::new(Cons3(Rc::clone(&value), Rc::new(Nil3)));

        let b = Cons3(Rc::new(RefCell::new(6)), Rc::clone(&a));
        let c = Cons3(Rc::new(RefCell::new(10)), Rc::clone(&a));

        *value.borrow_mut() += 10;

        println!("a after = {:?}", a);
        println!("b after = {:?}", b);
        println!("c after = {:?}", c);
    }
```

### 7、引用循环与内存泄漏

制造一个循环引用导致的内存泄漏

```rust
#[derive(Debug)]
enum List4 {
    Cons4(i32, RefCell<Rc<List4>>),
    Nil4,
}

use crate::List4::{Cons4, Nil4};

impl List4 {
    fn tail(&self) -> Option<&RefCell<Rc<List4>>> {
        match self {
            Cons4(_, item) => Some(item),
            Nil4 => None,
        }
    }
}

    {
        // 制造一个循环引用导致的内存泄漏
        // a = (5, nil)
        let a = Rc::new(Cons4(5, RefCell::new(Rc::new(Nil4))));

        println!("a 初始化引用计数 = {}", Rc::strong_count(&a));
        println!("a 的 tail = {:?}", a.tail());

        // b = (10, a)
        let b = Rc::new(Cons4(10, RefCell::new(Rc::clone(&a))));

        println!("a 在 b 创建后的引用计数 = {}", Rc::strong_count(&a));
        println!("b 初始化引用计数 = {}", Rc::strong_count(&b));
        println!("b 的 tail = {:?}", b.tail());

        // a = (5, b)
        if let Some(link) = a.tail() {
            *link.borrow_mut() = Rc::clone(&b);
        }

        println!("改变 a 的 tail 为 b 后, b 的引用计数 = {}", Rc::strong_count(&b));
        println!("改变 a 的 tail 为 b 后, a 的引用计数 = {}", Rc::strong_count(&a));

        // Uncomment the next line to see that we have a cycle;
        // it will overflow the stack
        // println!("a next item = {:?}", a.tail());
    }
```

使用 `Weak<T>` 弱引用防止循环引用

原理

* `Rc<T>` 内部存在两个计数器 `strong_count` 和 `weak_count`
* `Rc::downgrade(Rc<T> &self) -> Weak<T>` 调用将导致 `weak_count` + 1
* `Rc::clone(Rc<T> &self) -> Rc<T>` 将导致 `strong_count` + 1
* 当 `strong_count` 为 0 时，T 将被 `drop` 此时 通过 `Weak<T>.upgrade()` 则只能拿到 `None`

实验例子

```rust
use std::rc::Rc;
use std::rc::Weak;

fn test_rc_weak() -> Weak<i32> {
    let rc1 = Rc::new(3);
    let rc2 = rc1.clone();
    let weak1 = Rc::downgrade(&rc1);
    println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&rc1),
            Rc::weak_count(&rc1),  // 0
    );
    weak1
}

fn main() {
    let weak1 = test_rc_weak();
    println!("{:?}", weak1.upgrade());
}
```

官方例子如下

```rust

// 双向链表类似的结构的实现使用弱引用防止循环引
// 例子：树结构，节点持有所有孩子的引用和指向父亲的引用

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>, // 指向父亲的弱引用（不持有所有权，通过Rc::downgrade创建，Rc<T> 类型使用 weak_count 来记录其存在多少个 Weak<T> 引用）
    children: RefCell<Vec<Rc<Node>>>,
}

    {
        // 弱引用应用样例
        // 创建叶子
        let leaf = Rc::new(Node {
            value: 3,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![]),
        });

        println!("leaf 的 父亲 = {:?}", leaf.parent.borrow().upgrade());

        // 创建父节点
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        // 叶子节点的 parent 引用指向 父亲
        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!("leaf 的 父亲 = {:?}", leaf.parent.borrow().upgrade());
    }

    {
        // 查看 Rc 内部 强引用 strong_count 和 弱引用 weak_count 的值
        let leaf = Rc::new(Node {
            value: 3,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![]),
        });

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf), // 1
            Rc::weak_count(&leaf),  // 0
        );

        {
            let branch = Rc::new(Node {
                value: 5,
                parent: RefCell::new(Weak::new()),
                children: RefCell::new(vec![Rc::clone(&leaf)]),
            });

            *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

            println!(
                "branch strong = {}, weak = {}",
                Rc::strong_count(&branch), // 1
                Rc::weak_count(&branch), // 1
            );

            println!(
                "leaf strong = {}, weak = {}",
                Rc::strong_count(&leaf), // 2
                Rc::weak_count(&leaf), // 0
            );
        } // branch.strong_count == 0, branch.T 被回收，

        println!("leaf parent = {:?}", leaf.parent.borrow().upgrade()); // 父亲已经被回收, 返回 None（is_dangling(branch.ptr) 已经悬空了）
        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf), // 1
            Rc::weak_count(&leaf), // 0
        );
    }
```

## 十五、并发

### 1、线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // 创建新线程
    // 为了减小运行时，rust 线程与 操作系统线程 1:1 关系
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("子线程 {}", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("主线程 {}", i);
        thread::sleep(Duration::from_millis(1));
    }
    // 默认主线程结束后子线程也自动结束
    // 使用join等待子线程结束
    handle.join().unwrap();

    // 线程闭包
    {
        let v = vec![1, 2, 3];

        let handle = thread::spawn(move || { // 不适用move将报错，因为编译器推断出是引用
            println!("闭包环境v = {:?}", v);
        });

        // println!("{:?}", v); // 上边 move 作用 是 将 闭包 捕获到的变量 移动到 内部，原理应该是 构建匿名结构体时类型不为引用

        handle.join().unwrap();
    }
}
```

### 2、消息传递

```rust
use std::thread;
use std::time::Duration;
use std::sync::mpsc;


fn main() {
    // 类似Go语言消息通道
    {
        // let (tx, rx) = mpsc::channel::<String>();
        let (tx, rx) = mpsc::channel();
        let tx2 = mpsc::Sender::clone(&tx); // 通过clone创建多个发送者
        // tx 和 rx 通常作为 发送者（transmitter）和 接收者（receiver）的缩写

        // 子线程1发送消息
        thread::spawn(move || {
            let val = String::from("hi");
            // 调用时 rx 没有被 drop 则返回OK，否则返回Err
            tx.send(val).unwrap();

             let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("thread"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                thread::sleep(Duration::from_secs(1));
            }
        });

        // 子线程2发送消息
        thread::spawn(move || {
            let vals = vec![
                String::from("more"),
                String::from("messages"),
                String::from("for"),
                String::from("you"),
            ];

            for val in vals {
                tx2.send(val).unwrap();
                thread::sleep(Duration::from_secs(1));
            }
        });
        // 阻塞直到收到消息
        let received = rx.recv().unwrap();
        println!("收到消息: {}", received);
        // 迭代器接收消息
        for received in rx.iter() { // 当通道关闭不迭代器返回
            println!("迭代器方式: {}", received);
        }
        // 立即返回
        match rx.try_recv() {
            Ok(m) => println!("try_recv 收到消息: {}", m),
            Err(e) => println!("try_recv 异常 {:?}", e),
        };
    }
}
```

### 3、共享状态

```rust
use std::thread;
use std::time::Duration;
use std::sync::mpsc;
use std::sync::{Mutex, Arc};


fn main() {
    // 共享状态

    // 创建一个锁（这个锁持有一个共享变量）
    let m = Mutex::new(5); // 提供了内部可变性
    // Rust 编译器不能避免死锁

    {
        // lock 获取锁，如果被其他进程持有，将阻塞
        // unwrap 获取锁持有变量的智能指针
        let mut num = m.lock().unwrap();
        *num = 6; // 修改共享变量的值
    }

    println!("m = {:?}", m);

    // 多线程共享状态
    // let counter = Mutex::new(0); //直接使用counter报错，因为所有权没移动了多次
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter); // 创建线程安全引用计数
        let handle = thread::spawn(move || {
            // 使用线程安全引用计数
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("多线程共享变量值: {}", *counter.lock().unwrap());

}
```

### 4、使用 Sync 和 Send trait 的可扩展并发

Send 特质

* 只有实现了 Send 特质的类型的所有权才能在多个线程中移动
* Send 特质没有方法，仅仅作为编译器标记
* `Rc<T>` 并未实现该特质

Sync 特质

* 只有实现了 Sync 特质的类型的引用才能发送的其他线程
* Sync 特质没有方法，仅仅作为编译器标记
* `Rc<T>` 并未实现该特质

Send 和 Sync 特质属于 自动实现的 特质（利用 Rust 的 泛型实现），不实现通过 类似于 `!Send` 进行标记

更过参见 https://wiki.jikexueyuan.com/project/rust-primer/marker/sendsync.ht

## 十六、面向对象

### 1、封装

Rust中的封装：通过结构体和impl语句块实现

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}

impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

### 2、继承

Rust 不支持传统意义上的继承，可以通过默认方法实现继承的效果

```rust

trait MyTrait {
    fn defaultMethod(&self){
        println!("Default Impl");
    }
}

struct MyStruct {}

impl MyTrait for MyStruct {}

    let myStruct = MyStruct{};
    myStruct.defaultMethod();

```

### 3、多态

```rust
通过 trait 和 dyn 关键字实现

// 通用方法抽象为Trait
// 本例中为 GUI中通用方法 draw 绘制
pub trait Draw {
    fn draw(&self);
}

// 屏幕持有持有实现了Draw特质的Vec
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>, // Trait对象: 这种方式允许存放异质元素，会带来额外的运行时负担，实现上类似Java类型擦除
    // Trait 对象安全限制，只允许对象安全的Trait使用 dyn 修饰 规则如下两条
    // 返回值类型不为 Self
    // 方法没有任何泛型类型参数
    // （因为运行时类型擦除，返回Self类型，但是运行时并不知道Self和泛型是什么）
}

// screen 有一个 run 方法 调用 draw
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

// 实现Draw子类
// 模拟一个Button
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // 实际绘制按钮的代码
        println!("Button")
    }
}

// 模拟一个选择器
struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
        println!("SelectBox");
    }
}

// 以下这种方式不允许，因为编译后具象化出来了
// pub struct Screen<T: Draw> {
//     pub components: Vec<T>,
// }

// impl<T> Screen<T>
//     where T: Draw {
//     pub fn run(&self) {
//         for component in self.components.iter() {
//             component.draw();
//         }
//     }
// }

    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
```

### 4、状态模式

通过将系统状态维护在各个状态对象中，利用多态，消除代码case when

参考 https://kaisery.github.io/trpl-zh-cn/ch17-03-oo-design-patterns.html

Rust 推荐将 状态转换转化为不同类型之间的转换

## 十七、高级特性

### 1、不安全的Rust

`unsafe` 关键字可以切换到不安全 Rust，可以做到如下功能

* 解引用裸指针
* 调用不安全的函数或方法
* 访问或修改可变静态变量
* 实现不安全 `trait`
* 访问 `union` 的字段

特别注意

* unsafe 并不会关闭借用检查器或禁用任何其他 Rust 安全检查：如果在不安全代码中使用引用，它仍会被检查。
* 为了尽可能隔离不安全代码，将不安全代码封装进一个安全的抽象并提供安全 API 是一个好主意

**解引用裸指针**

不安全 Rust 有两个被称为 裸指针（raw pointers）的类似于引用的新类型。

和引用一样，裸指针是可变或不可变的，分别写作：（这里的星号不是解引用运算符；它是类型名称的一部分）

* 不可变 `*const T`
* 可变`*mut T`

引用和智能指针的区别在于，记住裸指针

* 允许忽略借用规则，可以同时拥有不可变和可变的指针，或多个指向相同位置的可变指针
* 不保证指向有效的内存
* 允许为空
* 不能实现任何自动清理功能

创建裸指针

```rust
    {
        let mut num = 5;
        // 创建裸指针不需要使用unsafe包裹
        let r1 = &num as *const i32; // 通过as强转为裸指针
        println!("addr={:x}", r1 as usize);
        let r2 = &mut num as *mut i32;
        println!("addr={:x}", r2 as usize);
    }
```

创建一个不可用的裸指针

```rust
    {
        let address = 0x012345usize;
        let r = address as *const i32;
        println!("addr={:x}", r as usize);
    }
```

使用裸指针

```rust
    {
        let mut num = 5;

        let r1 = &num as *const i32;
        let r2 = &mut num as *mut i32;

        unsafe {
            println!("r1 is: {}", *r1);
            println!("r2 is: {}", *r2);
        }
    }
```

**调用不安全函数或方法**

```rust
    {
        unsafe fn dangerous() {
            println!("unsafe function")
        }
        // dangerous(); //报错 call to unsafe function is unsafe and requires unsafe function or block
        unsafe {
            dangerous();
        }
    }
```

例子：模拟实现 `split_at_mut`

```rust
    // 例子：模拟实现 split_at_mut
    {
        // 将一个Vec的切片一分为两个
        let mut v = vec![1, 2, 3, 4, 5, 6];

        let r = &mut v[..];

        let (a, b) = r.split_at_mut(3); // 切分函数

        assert_eq!(a, &mut [1, 2, 3]);
        assert_eq!(b, &mut [4, 5, 6]);

        // 模拟实现
        use std::slice;
        fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
            let len = slice.len();
            let ptr = slice.as_mut_ptr(); // 获取裸指针

            assert!(mid <= len);

            unsafe { // 调用不安全的实现
                (slice::from_raw_parts_mut(ptr, mid),
                slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
            }
        }
        let (a, b) = split_at_mut(r, 3); // 切分函数
        assert_eq!(a, &mut [1, 2, 3]);
        assert_eq!(b, &mut [4, 5, 6]);
    }
```

调用外部其他语言库

```rust
    // 使用 extern 函数调用外部代码（c语言函数）
    {
        extern "C" {
            fn abs(input: i32) -> i32;
        }
        unsafe {
            println!("Absolute value of -3 according to C: {}", abs(-3));
        }
    }
```

从其它语言调用 Rust 函数

```rust
#[no_mangle] //还需增加 #[no_mangle] 注解来告诉 Rust 编译器不要 mangle 此函数的名称
pub extern "C" fn call_from_c() { // extern 的使用无需 unsafe。
    println!("Just called a Rust function from C!");
}
```

**访问或修改可变静态变量**

* rust 本身支持全局变量（静态变量）
    * `static HELLO_WORLD: T`
    * `static mut HELLO_WORLD: T`
* 修改或者访问可变的全局变量需要 `unsafe` 代码块

```rust
    // 访问或修改静态变量
    {
        static mut COUNTER: u32 = 0;

        fn add_to_count(inc: u32) {
            unsafe {
                COUNTER += inc;
            }
        }

        add_to_count(3);

        unsafe {
            println!("COUNTER: {}", COUNTER);
        }
    }
```

**实现不安全的trait**

* unsafe 标记 trait 仅仅是作为对于实现者的警告（因为实现者必须也使用`unsafe impl`）
* 编译器不会做上述之外的任何操作
* 一般当一个 trait 的内存安全与否依赖于实现者时，需要增加 `unsafe` 标记以警告实现者
* 一些例子 `Searcher`、`Send`、`Sync`
* [stackoverflow参考](https://stackoverflow.com/questions/31628572/when-is-it-appropriate-to-mark-a-trait-as-unsafe-as-opposed-to-marking-all-the)

示例

```rust
    {
        // 实例参考 Searcher
        unsafe trait Foo {
            // methods go here
            fn test(&self);
        }

        unsafe impl Foo for i32 { // 约束在这，实现者必须也使用unsafe标记
            // method implementations go here
            fn test(&self){
                let ptr = self as *const i32;
                unsafe { println!("unsafe trait value={}", *ptr) };
            }
        }
        let i:i32 = 1;
        i.test();
    }
```

### 2、高级 Trait

关联类型在 trait 定义中指定占位符类型

关联类型和泛型最大的区别就是针对一个类型，关联类型的Trait只能被实现一次，不支持静多态

```rust
    {
        // 带有关联类型的特质
        pub trait MyTrait {
            type A; // 在一个特质中使用type关键字定义，一般作为函数返回值（每一个实现者只需要指定一次，会出现指定多次）
            type B; // 可以定义多个

            fn get_a_option(&self, a: Self::A) -> Option<Self::A>;
            fn get_b_option(&self, b: Self::B) -> Option<Self::B>;
        }
        // 实现带有关联类型的特质
        pub struct MyStruct {}
        impl MyTrait for MyStruct {
            type A = i32;
            type B = &'static str;

            fn get_a_option(&self, a: Self::A) -> Option<Self::A> {
                println!("a = {}", a);
                Some(a)
            }
            fn get_b_option(&self, b: Self::B) -> Option<Self::B> {
                println!("b = {}", b);
                Some(b)
            }
        }
        let s = MyStruct{};
        s.get_a_option(1);
        let b_o = s.get_b_option("str");
        println!("{:?}", b_o);
        // 为什么不使用泛型<T>，因为泛型的话在调用的时候需要手动指定类型，对调用方不友好
        // 例子如下
        pub trait MyTrait2<T> {
            fn get_t_option(&self) -> Option<T>;
        }
        pub struct MyStruct2 {};
        impl MyTrait2<i32> for MyStruct2 {
            fn get_t_option(&self) -> Option<i32>{
                Some(1)
            }
        }
        impl MyTrait2<&'static str> for MyStruct2 {
            fn get_t_option(&self) -> Option<&'static str>{
                Some("str")
            }
        }
        let s2 = MyStruct2{};
        println!("{:?}", s2.get_t_option() as Option<i32>); // 调用时需要明确类型
        let o2:Option<&str> = s2.get_t_option();  // 调用时需要明确类型
        println!("{:?}",o2);
    }
```

默认泛型参数和[运算符重载](https://kaisery.github.io/trpl-zh-cn/appendix-02-operators.html#%E8%BF%90%E7%AE%97%E7%AC%A6)

默认参数语法 `trait Add<RHS=Self> {}`

```rust
    {
        // Rust的运算符重载通过实现std::ops下定义的trait来实现
        // 下面一个例子为 二元操作符 + 例子
        use std::ops::Add;

        #[derive(Debug, PartialEq)]
        struct Point {
            x: i32,
            y: i32,
        }

        impl Add for Point {
            type Output = Point;

            fn add(self, other: Point) -> Point {
                Point {
                    x: self.x + other.x,
                    y: self.y + other.y,
                }
            }
        }

        // Add trait的定义如下
        /*
        trait Add<RHS=Self> { // RHS 为右侧被操作值默认为Self，即实现者的类型
            type Output;

            fn add(self, rhs: RHS) -> Self::Output;
        }
        */
        // RHS 存在目的是不同类型之间的计算
        struct Millimeters(u32);
        struct Meters(u32);

        impl Add<Meters> for Millimeters { // 实现了毫米 + 米的运算
            type Output = Millimeters; // 结果是毫米

            fn add(self, other: Meters) -> Millimeters {
                Millimeters(self.0 + (other.0 * 1000))
            }
        }
    }
```

完全限定语法与消歧义：调用相同名称的方法

语法为：`<Type as Trait>::function(receiver_if_method, next_arg, ...);`

```rust
    {
        // 一个结构体实现多个trait，如何消除歧义
        // 方法通过 TraitName::method(self) 调用
        // 关联函数通过 <StructName as TraitName>::functionName()
        // 一般来说语法为： <Type as Trait>::function(receiver_if_method, next_arg, ...);
        trait Pilot { fn fly(&self); }
        trait Wizard { fn fly(&self); }
        struct Human;
        impl Pilot for Human {
            fn fly(&self) {
                println!("This is your captain speaking.");
            }
        }
        impl Wizard for Human {
            fn fly(&self) {
                println!("Up!");
            }
        }
        impl Human {
            fn fly(&self) {
                println!("*waving arms furiously*");
            }
        }
        let person = Human;
        person.fly(); // 默认调用定义在结构体上的方法
        // 调用特质上的方法
        Pilot::fly(&person);
        Wizard::fly(&person);
        // 关联函数如何调用（第一个参数不是self）
        trait Animal {
            fn baby_name() -> String;
        }
        struct Dog;
        impl Dog {
            fn baby_name() -> String {
                String::from("Spot")
            }
        }
        impl Animal for Dog {
            fn baby_name() -> String {
                String::from("puppy")
            }
        }
        println!("A baby dog is called a {}", Dog::baby_name());
        // println!("A baby dog is called a {}", Animal::baby_name()); // 报错，编译器不知道使用哪一个实现
        println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
    }
```

Trait 继承语法： `trait SubTrait : ParentTrait {}` 或者 `trait SubTrait where Self: ParentTrait`

* 不要理解为继承，而要理解为约束，表示，某结构体要想实现 `SubTrait`，则必须实现 `ParentTrait`

示例

```rust
    {
        // 父 trait 用于在另一个 trait 中使用某 trait 的功能
        // 语法如下： trait SubTrait : ParentTrait {}
        use std::fmt;
        trait OutlinePrint: fmt::Display {
            fn outline_print(&self) {
                let output = self.to_string();
                let len = output.len();
                println!("{}", "*".repeat(len + 4));
                println!("*{}*", " ".repeat(len + 2));
                println!("* {} *", output);
                println!("*{}*", " ".repeat(len + 2));
                println!("{}", "*".repeat(len + 4));
            }
        }
        struct Point {
            x: i32,
            y: i32,
        }
        impl OutlinePrint for Point {} // 如果没有下方实现将报错 `main::Point` doesn't implement `std::fmt::Display`
        impl fmt::Display for Point {
            fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
                write!(f, "({}, {})", self.x, self.y)
            }
        }
        let p = Point { x:1, y:1 };
        p.outline_print();
    }
```

newtype 模式用以在外部类型上实现外部 trait

* 孤儿规则：只要 trait 或类型对于当前 crate 是本地��话就可以在此类型上实现该 trait （也就是说trait和struct都不在当前crate将不允许impl）
* 解决方案：创建一个新的包裹类型
    * 必须直接在 Wrapper 上实现所有方法，这样就可以代理到 `self.0` 上 —— 这就允许我们完全像 被包裹类型一样 那样对待 Wrapper。
    * 或者为封装类型实现 Deref trait 方法，返回 `self.0`

示例

```rust
    {
        use std::fmt;
        // impl fmt::Display for Vec<T> {} // 报错 only traits defined in the current crate can be implemented for arbitrary types
        struct Wrapper(Vec<String>);
        impl fmt::Display for Wrapper {
            fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
                write!(f, "[{}]", self.0.join(", "))
            }
        }
        let w = Wrapper(vec![String::from("hello"), String::from("world")]);
        println!("w = {}", w);
    }
```

### 3、高级类型

为了类型安全和抽象而使用 newtype 模式

参见 十二、最佳实践.2

**类型别名用来创建类型同义词**

```rust
    {
        // 类型别名
        type Kilometers = i32;
        let x: i32 = 5;
        let y: Kilometers = 5;
        println!("x + y = {}", x + y);
        // 减少重复
        /*
        Box<dyn Fn() + Send + 'static>
        let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));
        fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
            // --snip--
        }
        fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
            // --snip--
        }
        type Thunk = Box<dyn Fn() + Send + 'static>; // 类型别名减少重复
        let f: Thunk = Box::new(|| println!("hi"));
        fn takes_long_type(f: Thunk) {
            // --snip--
        }
        fn returns_long_type() -> Thunk {
            // --snip--
        }
        */
        // type Result<T> = std::result::Result<T, std::io::Error>;
    }
```

**从不返回的 never type**

```rust
    {
        // never type 或者 empty type rust 中用 ! 表示
        fn bar() -> ! {
            unimplemented!()
            // 或者
            // loop {
            // }
        }
        // 表示 函数 bar 从不返回（可能有两种情况：死循环、panic），所以：
        // panic! 返回 ! 类型
        // continue 返回 ! 类型
        // loop 返回 ! 类型值（没有break情况）
        // 因此 ! 类型可以强转为 任意类型
    }
```

**动态大小类型和 Sized trait**

```rust
    {
        // let s1: str = "Hello there!"; // 报错，因为不能直接引用str类型，因为str是动态大小类型，编译期不可知
        let s2: &'static str = "Hello there!"; // &str 则是 两个 值：str 的地址和其长度
        // 为了处理 DST，Rust 有一个特定的 trait 来决定一个类型的大小是否在编译时可知：这就是 Sized trait。
        fn generic<T>(t: T) {
            // --snip--
        }
        // 等价于
        fn generic2<T: Sized>(t: T) {
            // --snip--
        }
        // 如果允许接收动态大小类型，显示 `T: ?Sized`
        fn generic3<T: ?Sized>(t: &T) {
            // --snip--
        }
        // 另外： Trait 也是动态类型，不能直接返回，需要使用Box<dyn> 包裹
    }
```

### 4、高阶函数和闭包

高阶函数

```rust
    {
        fn add_one(x: i32) -> i32 {
            x + 1
        }
        // fn 类型接受函数指针作为参数
        // fn 是一个类型而不是一个 trait
        // 函数指针实现了所有三个闭包 trait（Fn、FnMut 和 FnOnce）
        fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
            f(arg) + f(arg)
        }
        let answer = do_twice(add_one, 5);
        println!("The answer is: {}", answer);
        let list_of_numbers = vec![1, 2, 3];
        // 传递闭包
        let list_of_strings = list_of_numbers
            .iter()
            .map(|i| i.to_string())
            .collect::<Vec<String>>();
        // 传递函数指针
        let list_of_strings: Vec<String> = list_of_numbers
            .iter()
            .map(ToString::to_string)
            .collect();
        // 传递初始化函数
        enum Status {
            Value(u32),
            Stop,
        }
        let list_of_statuses: Vec<Status> =
            (0u32..20)
            .map(Status::Value)
            .collect();
    }
```

返回闭包

```rust
    {
        // fn returns_closure() -> Fn(i32) -> i32 { // 报错：因为Trait为尺寸运行时不可知
        //     |x| x + 1
        // }
        fn returns_closure() -> Box<dyn Fn(i32) -> i32> { // 使用box封装
            Box::new(|x| x + 1)
        }
        println!("returns_closure result = {}", returns_closure()(1));
    }
```

### 5、宏

所谓的元编程，rust 提供了

* 一种声明宏 `macro_rules!`
* 三种过程宏
    * 自定义 `#[derive]` 宏在结构体和枚举上指定通过 derive 属性添加的代码
    * 类属性（Attribute）宏定义可用于任意项的自定义属性（应该理解为像属性一样的宏）
    * 类函数宏看起来像函数不过作用于作为参数传递的 token（应该理解为像函数一样的宏）

**宏核函数的区别**

* 宏是编写生成代码的代码
* 宏可以减少代码量，可以做到函数无法做到事情
* 函数必须声明函数参数个数和类型
* 宏只接受一个可变参数
* 宏在编译前将被展开
* 函数在运行时被调用
* 宏代码更加难以理解与维护
* 宏和函数的最后一个重要的区别是：在调用宏 之前 必须定义并将其引入作用域，而函数则可以在任何地方定义和调用。

**使用 `macro_rules!` 的声明宏用于通用元编程**

更多参考 https://danielkeep.github.io/tlborm/book/index.html

```rust
    {
        // 使用 macro_rules! 声明宏
        // 本例中模拟了 vec! 实现
        // 注意：标准库中实际定义的 vec! 包括预分配适当量的内存的代码。这部分为代码优化，为了让示例简化，此处并没有包含在内。
        #[macro_export] // #[macro_export] 注解说明宏应该是可用的。 如果没有该注解，这个宏不能被引入作用域。
        // 定义时宏名不带感叹号
        macro_rules! my_vec { // 开始宏体
            // 本质上是模式匹配 (模式) => {}
            // $(  ) 表示一个模式
            // $x:expr 表示匹配一个表达式，并命名为x
            // ,* 表示以逗号分隔，匹配0个或者多个
            ( $( $x:expr ),* ) => { // 这是一个代码块
                {
                    let mut temp_vec = Vec::new();
                    $( // 表示内部会使用捕获的变量
                        temp_vec.push($x); // $x 引用捕获的变量
                    )*
                    temp_vec
                }
            };
        }
        let v = vec!(1,2,3);
        let v2 = vec![1,2,3];
        let v3 = vec!{1,2,3};
        println!("{:?}", v);
        println!("{:?}", v2);
        println!("{:?}", v3);
        let v4 = my_vec!(1,2,3);
        let v5 = my_vec![1,2,3];
        // 展开后如下
        let v6 = {
            let mut temp_vec = Vec::new();
            temp_vec.push(1);
            temp_vec.push(2);
            temp_vec.push(3);
            temp_vec
        };
        println!("{:?}", v4);
        println!("{:?}", v5);
        println!("{:?}", v6);
    }
```

`macro_rules!` 中有一些奇怪的地方。在将来，会有第二种采用 `macro` 关键字的声明宏，其工作方式类似但修复了这些极端情况。在此之后，macro_rules! 实际上就过时（deprecated）了。

**如何编写自定义 derive 宏**

[标准库提供的可派生trait](https://kaisery.github.io/trpl-zh-cn/appendix-03-derivable-traits.html)

* Debug
* PartialEq 和 Eq
* 次序比较的 PartialOrd 和 Ord
* 复制值的 Clone 和 Copy
* 固定大小的值到值映射的 Hash
* 默认值的 Default

rust 提供了一种为结构体或者枚举自动生成代码的宏，并与一个Trait绑定

trait定义者提供两个库，分别声明trait的库和声明过程宏的库

实现步骤

* 库编写者
    * 实现 声明trait的库
    * 实现 过程宏的库
* 库使用者
    * 引入以上两个库
    * 声明自己的结构体/枚举
    * 在结构体/枚举上方添加 `#[derive(TraitName)]` 注解

创建 声明trait的库 `cargo new hello_macro --lib` 并编写 `hello_macro/src/lib.rs`

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

在hello_macro项目下创建 `cargo new hello_macro_derive --lib`

添加依赖 `hello_macro/hello_macro_derive/Cargo.toml`

```toml
[lib]
proc-macro = true

[dependencies]
syn = "0.14.4"
quote = "0.6.3"
```

编写 `hello_macro/hello_macro_derive/src/lib.rs`

```rust
extern crate proc_macro; // 声明外部的crate （rust自带 编译器用来读取和操作我们 Rust 代码的 API）

use crate::proc_macro::TokenStream;
use quote::quote; // quote 则将 syn 解析的数据结构反过来传入到 Rust 代码中
use syn; // syn crate 将字符串中的 Rust 代码解析成为一个可以操作的数据结构。

#[proc_macro_derive(HelloMacro)] // 为 HelloMacro 特质实现一个默认的过程宏
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // 构建 Rust 代码所代表的语法树
    // 以便可以进行操作
    let ast = syn::parse(input).unwrap(); // 这里选择用 unwrap 来简化了这个例子；在生产代码中，则应该通过 panic! 或 expect 来提供关于发生何种错误的更加明确的错误信息。
    println!("过程宏代码，编译器执行");
    // 构建 trait 实现
    impl_hello_macro(&ast)
}


fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident; // 获取被注解的结构体名
    let gen = quote! { // 编写生成代码
        impl HelloMacro for #name { // #name 是一种模板机制，使用值来替换 https://docs.rs/quote
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name)); // stringify! 一个内置宏，将内容转换为字符串
            }
        }
    };
    gen.into()
}
```

库使用方

添加依赖 `advanced-features/Cargo.toml`

```toml
[dependencies]
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```

使用 `advanced-features/src/main.rs`

```rust
    {
        use hello_macro::HelloMacro; // 导入特质
        use hello_macro_derive::HelloMacro; //导入过程宏
        // 过程宏
        #[derive(HelloMacro)] // 声明使用过程宏
        struct Pancakes;
        Pancakes::hello_macro();
    }
```

**类属性宏**

编写方式是和自定义派生宏相似

相关实例参见： [rocket框架rocket_codegen模块](https://docs.rs/crate/rocket_codegen/0.4.0/source/src/attribute/route.rs)

使用

```rust
#[route(GET, "/")]
fn index() {
```

过程宏定义

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

**类函数宏**

编写方式是和自定义派生宏相似

使用

```rust
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

过程宏编写

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

### 6、静态派发和动态派发

关于 dyn trait 和 impl trait

* 静态派发 `impl trait` 可以用在函数的参数和返回值上，即编译期根据类型生成多份代码
    * 用在 函数返回值 上 `fn foo() -> impl Adder`，返回的类型为 `impl Adder`
        * 在编译上 会编译为 `fn foo() -> 具体类型` 一份，
        * 和直接写区别不大，不会减少代码量，设计这个语法的原因在于：封装，比如只想暴露接口的情况
    * 用在 函数参数 上 `fn bar(a: impl Adder)`，基本等价于 `fn bar<T: Adder>(a: T)`
        * 有了泛型为什么还要这个语法？为了语言语法的对称性
* 动态派发 `Box<dyn trait>`
    * rust 支持 `Box<Struct impl trait>` 到 `Box<dyn trait>` 的转换

例子如下

```rust
trait Adder {
    fn add(&self, a: i32, b:i32) -> i32;
}

struct A;
impl Adder for A {
    fn add(&self, a: i32, b:i32) -> i32 {
        a+b
    }
}

fn main() {

    fn foo() -> impl Adder {
        A{}
    }

    fn foo2() -> Box<dyn Adder> {
        Box::new(A{})
    }
    fn foo3() -> Box<str> {
        Box::from("123")
    }

    fn bar(a: impl Adder) {
        a.add(1,2);
    }

    fn bar2<T: Adder>(a: T) {
        a.add(1,2);
    }

    fn bar3(a: &dyn Adder) {
        a.add(1,2);
    }
    let c = foo();
    let d = foo();
    let f = &foo();
    let g: &dyn Adder = &foo();
    let h: &dyn Adder = &A{};
    let s = "xxx";
    struct S {
        s: str
    }
    let s2 = Box::new("");

    bar(A{});
    bar(c);
    bar2(A{});
    bar2(d);
}
```

### 7、动态尺寸类型 与 静态尺寸类型

https://zhuanlan.zhihu.com/p/21820917

rust 共有两种类型：

* 静态尺寸类型
    * 在编译期，大小可知的类型，可以在栈中创建，因此可以作为函数参数和返回值
    * 其引用对应的指针为瘦指针，大小为 8 字节（64位）
    * 常见的 静态尺寸类型 类型包括
        * 不包含 DST 的结构体
        * 基本数据类型
        * 所有引用、指针类型
* 动态尺寸类型 DST
    * 在编译期间，大小不可知的类型，只能在堆中创建，因此无法直接作为函数参数（因此不是一等公民）
    * 仅有的几种 DST 类型
        * trait 类型 `dyn trait`
        * str 或者 切片类型 类型 `str`
        * 数组类型 `[type]`
        * 最后一个字段为 DST 的 结构体
    * 目前无法创建 原子类型 的 DST 类型
    * 其引用对应的指针为胖指针，大小大于 8 字节（64位）
* 零尺寸类型 ZST
