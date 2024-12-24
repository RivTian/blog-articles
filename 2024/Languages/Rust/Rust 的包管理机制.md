#Rust 
## 背景

Rust 作为一门强大的系统编程语言，拥有一个**现代且成熟的**包管理机制。C/C++ 发展多年，迄今也没有特别靠谱好用且生态健全的包管理系统（据说 [Conan](https://conan.io/) 还不错），以至于很多时候都以源代码的形式（比如常见的 `deps/` 目录或者 git submodules 等）来管理依赖。就这点上，Rust 通过语言层面的设计，并辅以 Cargo 包管理器，原生为用户提供了一个体验极佳的包管理系统。

由于系统且详细介绍 Rust 包管理机制的文章比较少，本文尝试做一下梳理，希望能给大多数用户做一个有效参考。

## 基本概念

我们先理解一下 Rust 包管理机制从逻辑上提供的 3 个基本概念：

1. **module**

   语言层面提供了 `mod` 关键字来定义 module。一个 module 类似于一个**命名空间**（与 C++ 的 namespace 比较像），可在 module 内部定义数据类型、变量、函数等 item，且通过相应的 visibility 机制来对外暴露。一个单独的 Rust 源文件可定义多个 module。

2. **crate**

   **可理解为多个 module 构成 crate**。crate 是 Rust 中的**最小编译单元**。一个 crate 将会有相应的 `Cargo.toml` 文件来进行对 Cargo 行为的配置。**从构建产物来看**，crate 可是一个 lib 或者 binary，亦或二者皆可（即有 binary 生成也可作为 lib）。

3. **package**

   **可理解为多个 crate 构成 package**。其实 package 可视为一个大 crate（它也需要一个 `Cargo.toml`），包含了 1 个或多个小 crate。从这点来看，**crate 和 package 都可以叫作包**，因为单个 crate 也可以是一个 package，但是 package 通常**倾向于多个 crate 的集合**。package 只能有一个 lib crate 和多个 binary crate。

   为了能更好理解 package，我们可以看 [Cargo](https://github.com/rust-lang/cargo) 项目的组织形式：

   ```sh
   $ tree -L 1 cargo-0.59.0
   cargo-0.59.0
   ├── CHANGELOG.md
   ├── CONTRIBUTING.md
   ├── Cargo.toml
   ├── LICENSE-APACHE
   ├── LICENSE-MIT
   ├── LICENSE-THIRD-PARTY
   ├── README.md
   ├── benches
   ├── build.rs
   ├── ci
   ├── crates
   ├── publish.py
   ├── src
   ├── tests
   └── triagebot.toml
   ```

   

   Cargo 这个 package 有多个 crates，除了 `src/` 作为入口之外，还有 `crates/` 作为内部依赖库的集合：

   ```sh
   $ tree -L 1 cargo-0.59.0
   cargo-0.59.0
   ├── CHANGELOG.md
   ├── CONTRIBUTING.md
   ├── Cargo.toml
   ├── LICENSE-APACHE
   ├── LICENSE-MIT
   ├── LICENSE-THIRD-PARTY
   ├── README.md
   ├── benches
   ├── build.rs
   ├── ci
   ├── crates
   ├── publish.py
   ├── src
   ├── tests
   └── triagebot.toml
   ```

   

   `crates/` 是多个内部 crate 的集合。

通常来说，我们用 `cargo` 命令可创建一个 package（**默认只有一个 crate**）：

```sh
$ cargo new my-project-bin
$ tree -L 2 my-project-bin
  my-project-bin
  ├── Cargo.toml
  └── src
      └── main.rs
```



不加任何选项将默认创建含有一个 binary crate 的 package ，即只有一个 `src/main.rs`。

如果我们创建过程中加上 `--lib` 选项，则将创建含有一个 lib crate 的 package，如下：

```sh
$ cargo new my-project-lib --lib
$ tree -L 2 my-project-lib
  my-project-bin
  ├── Cargo.toml
  └── src
      └── lib.rs
```



即包含一个 `src/lib.rs`。在一个 crate 中，`src/main.rs` 和 `src/lib.rs` 是可同时存在，此时 `src/main.rs` 默认将是 binary 的根结点编译出相应的 binary，而 `src/lib.rs` 则作为 lib 的**根节点**（也可通过 `Cargo.toml` 的 [`[lib\]` 字段](https://doc.rust-lang.org/cargo/reference/cargo-targets.html#library)来修改这一默认行为）。

我们还可以**创建 `src/bin/` 目录**，并在这个特殊的目录下创建更多的 binary，Cargo 会将该目录下的每一个源文件都编译为 binary（此时 `src/main.rs` 依然可同时存在）。比如：

```sh
# 创建 multi-bin 项目，默认将有一个 src/main.rs
$ cargo new multi-bin

# 编译 multi-bin 项目，将生成 multi-bin 二进制
$ cargo build
$ ./target/debug/multi-bin
Hello, world!

# 创建 src/bin 目录
$ mkdir src/bin

# 创建 3 个二进制文件
$ cp src/main.rs src/bin/helloworld1.rs
$ cp src/main.rs src/bin/helloworld2.rs
$ cp src/main.rs src/bin/helloworld3.rs

# 编译构建
$ cargo build

# 将分别生成 multi-bin、helloworld1、helloworld2 和 helloworld3
$ ./target/debug/helloworld1 ; ./target/debug/helloworld2; ./target/debug/helloworld3
Hello, world!
Hello, world!
Hello, world!
```



我们还可以用下面这个命令来快速运行 `src/bin/` 目录下的某个二进制文件：

```sh
$ cargo run --bin helloworld1
```



当我们理解了 Rust 包管理机制的这三个基本概念后，我们就可以开始进行我们的冒险了。

## module 的基本使用

### 我们的第一个 module

理论上，Rust 的 module 可以定义在任意的文件中。我们接下来的例子很简单：**创建一个 package，接着在 `src/main.rs` 中定义 module，最后使用该 module**：

```sh
$ cargo new my-first-module
```



我们在 `src/main.rs` 定义 module `foo_mod`，并在 module 中定一个非常简单的函数。一个典型的 module 将被包裹在 `mod {}` 中：

```rust
mod foo_mod {
    pub fn foo() {
        println!("foo_mod::foo()");
    }
}
```



这里要特别注意 `pub` 关键字（我们将在下一节提到），此处可认为是将 `foo_mod` 中的 `foo()` 对外公开暴露（类似于 Go 里将某个函数头字母大写表示为公共函数）。

我们接下来在 `main()` 中使用这个 module：

```rust
fn main() {
    crate::foo_mod::foo();
}
```



其中 `crate::foo_mod::foo()` 表示了该 module 的一个完整的绝对路径（将在下一节提及），我们这里只需要知道这是为了使用 module 的公共函数 `foo()` 即可。

最后，Run：

```sh
$ cargo run
foo_mod::foo()
```



冒险很成功 ！

从这个例子我们可以知道，使用 module 内定义的 item 只需要满足两个条件：

1. **对应 item 的引用路径要写对**：比如下文将要提及的绝对路径和相对路径；
2. **使用 module 要符合 Rust 的 visibility 机制**：即我们调用的是公共的 item 还是私有的 item ；

只要符合这两个条件，**module 之间**就可以相互使用彼此定义的 item 。

### module 的引用路径

Rust 采用了一种类似于**文件系统目录树**（module tree）的形式来管理 module 的引用路径。module 之间通过使用其路径来使用 module（比如上面例子中的 `crate::foo_mod::bar()`）。为了简单起见，下面的所有例子都**显式加上了 `pub`**，即默认都是公共 item。

类比于文件系统，module 的引用路径可分为**绝对路径**和**相对路径**。让我们先来认识绝对路径。

绝对路径的**起点**可以有两种：

1. **名为 `crate` 的根节点**

   `src/main.rs` 和 `src/lib.rs` 可**分别**形成 `crate` 根。比如上面第一节的例子，module tree 即为：

   ```sh
   crate (src/main.rs)
   ├── foo_mod
       ├── foo()
   ```

   

   所以一个完整的 `foo()` 的引用路径为：`crate::foo_mod::foo()`。在 `src/lib.rs` 定义的 module 也可以此类推。

2. **crate 名**

   当我们同时有 `src/main.rs` 和 `src/lib.rs` 的时候，比如我们再在上面的例子中创建一个 `src/lib.rs`：

   ```sh
   pub mod bar_mod {
       pub fn bar() {
           println!("bar_mod::bar()");
       }
   
       pub fn call_bar() {
           crate::bar_mod::bar();
       }
   }
   ```

   

   我们在 `src/lib.rs` 中创建一个公共的 module `bar_mod`，并定义了公共函数 `bar()` 和 `call_bar()`，其中 `call_bar()` 中用绝对路径调用 `bar()`。虽然我们同时有 `src/main.rs` 和 `src/lib.rs`，但是这两个文件都各自有其**同名**根节点 `crate`，所以 `call_bar()` 在 `src/lib.rs` 中的用法没有问题（编译可以通过）。

   当我们在 `src/main.rs` 中使用 `bar()`，此时发现 `crate::bar_mod::bar()` 是不行的（编译报错），因为在 `src/main.rs` 中 `crate` 根节点指的并不是 `src/lib.rs`，而是 `src/main.rs`，所以此时将无法解析这个引用路径（错误信息为：` failed to resolve: unresolved import`）。**这种情况就只能用 crate 名**，即 `my_first_module`（如果原来 package 命名中带有 `-`，将会默认转换为下划线 `_`），如下所示：

   ```rust
   fn main() {
       crate::foo_mod::foo();
       // ERROR!
       // crate::bar_mod::bar();
       my_first_module::bar_mod::bar();
   }
   ```

   此时的 crate 名其实也是 package 名。由于一个 package 只能有一个 lib crate，所以此时 package 名指向的就是其 lib（即 `lib.rs`），不会产生二义性。

   除此之外，当我们在其他 crate 中使用这个 lib crate 时候，也**必须使用 crate 名**（因为 `crate::` 根仅对当前 crate 有效），也就是说 **crate 名是 lib crate 对外的根节点**。

**module 可以多层嵌套**，这是一个**极为重要**的用法。比如我们在 `src/lib.rs` 定义这么一个 module：

```rust
pub mod bar_mod {
    pub mod deeply {
        pub mod nested {
            pub fn hello() {}
        }
    }
}
```



则 `hello()` 在 `src/lib.rs` 的绝对路径为：`crate::bar_mod::deeply::nested::hello()`。假如我们将 `crate` 根节点类比为 `/` ，则这这个绝对路径就可以为 `/bar_mod/deeply/nested/hello()`，这和文件目录树是非常类似的。

绝对路径是一个非常好理解且容易掌握的方法，比如在 Go 中，我们引用某一个 package就**只能用绝对路径**。绝对路径的缺点就是一旦父级 module 发生了移动，则 module 内部中的引用路径将会**失效**。为解决这个问题我们需要引入**相对路径**，比如下面这个例子：

```rust
pub mod bar_mod {
    pub fn bar() {
        println!("bar_mod::bar()");
    }

    pub fn call_bar() {
        crate::bar_mod::bar();
    }

    pub mod deeply {
        pub mod nested {
            pub fn hello() {}
        }

        pub fn call_hello() {
            // 绝对路径
            crate::bar_mod::deeply::nested::hello();

            // 相对路径
            nested::hello();
        }
    }
}
```



在 `call_hello()` 中，我们分别用了绝对路径和相对路径来调用 `nested` module 中定义的 `hello()`。由于 `call_hello()` 与 `nested` module 处于同一层，可以理解是同一个目录下，因此 `call_hello()` 可以**看见**`nested`，可直接使用 `nested::hello()`。

假设 `bar_mod` 或者 `deeply` module 发生了移动，则 `crate::bar_mod::deeply::nested::hello()` 这个绝对路径将可能不再有效，而相对路径 `nested::hello()` 则依旧有效，**从而保障了 module 内部引用的稳定性**。

类比于文件目录树中的 `.` 和 `..` 这两个特殊目录，Rust 提供了 **`self`** 和 **`super`** 这两个特殊的关键字来分别表示**当前 module** 和**上一层 module**，比如下面这段代码中的 `use_self_and_super()`：

```rust
pub mod bar_mod {
    pub fn bar() {
        println!("bar_mod::bar()");
    }

    pub fn call_bar() {
        crate::bar_mod::bar();
    }

    pub mod deeply {
        pub mod nested {
            pub fn hello() {}
        }

        pub fn call_hello() {
            // 绝对路径
            crate::bar_mod::deeply::nested::hello();

            // 相对路径
            nested::hello();
        }

        pub fn use_self_and_super() {
            // self 即表示 deeply module
            self::nested::hello();

            // super 即表示 bar_mod
            super::call_bar();
        }
    }
}
```



其中：

- `self::nested::hello()`：由于 `use_self_and_super()` 定义在 `deeply` module 中且 `self` 表示当前 module，所以此时 `self` 是 `deeply` module。
- `super::call_bar()`：`super` 表示为上一层 module，即 `deeply` 的上一层 module，则为 `bar_mod`，所以这句代码为调用 `bar_mod` 的 `call_bar()` 函数。

### 公共还是私有

对于 module 本身及其 module 内定义的 item，Rust 的 visibility 机制将其划分为**公共**和**私有**（不像 C++ 还有一个 `protected`）。其实可理解为：**处于某个 module 中的用户是否能看得见（使用）其他 module 中的 item**。

让我们先了解几个基本的规则：

1. 一个 module 内的所有 item 默认为**私有**，除非显式加上 `pub` 关键字；
2. module 之外的用户只能看见 `pub` module 的 `pub` item ；
3. 同一个 module 内部同级 item 相互可以看见（无论是否私有）；
4. **父级 module 不能看见子级 module 的私有 item ，而子级 module 可看见所有祖先 module 的 item（无论是否私有）**；

规则 1 和 2 比较容易理解，规则 3 和 4 我们可以通过下面的小例子就可以理解：

```rust
mod parent_module {
    fn foo() {}

    // 父级 module 不能使用子级 module 的私有 item 
    fn use_child_module_private_function() {
        // ERROR!
        // child_module::call_foo_from_child();
    }

    // module 内部的同级 item 相互可以使用
    fn call_foo() {
        foo();
    }

    mod child_module {
        // 子级 module 可使用所有祖先 module 的 item 
        fn call_foo_from_child() {
            super::foo();
        }
    }
}
```



对于结构体和枚举，我们还需要注意：

- 结构体定义前使用 `pub` 可让结构体本身变成公共的结构体，**但是内部的字段依旧保持私有**，我们可以**逐一决定**是否将某个字段变成公共；

  ```rust
  mod parent_module {
      pub struct Breakfast {
          pub toast: String,
          fruit: String,
      }
  }
  ```

  如上，`Breakfast` 这个结构体是公共的，且 `toast` 字段也是公共的，而 `fruit` 字段则是私有的；

- 将枚举定义为公共的时，则整个类型都是公共的，比如：

  ```rust
  mod parent_module {
     pub enum Appetizer {
          Soup,
          Salad,
      }
  }
  ```

  

  则此时不仅 `Appetizer` 是公共的，`Soup` 和 `Salad` 也是公共的。

  枚举只有其所有类型都是公共的才能实现最大的功效。

  **类似的规则在 trait 的定义上也适用**，当 trait 被定义为公共时，则整个 trait 也为公共的。

### 使用 `use` 来导入 module

类似于 Go 或者 Java 的 `import`，Rust 也可以使用 `use` 语句来导入 module。比如上面的例子中，我们可以：

```rust
mod bar_mod {
    pub mod deeply {
        pub mod nested {
            pub fn hello() {}
        }
    }
}

// 使用绝对路径引入
use crate::bar_mod::deeply::nested;

fn main() {
    // 我们就可以直接使用 module 名
    nested::bar();
}
```



对于**外部的 crate**，我们必须显式地使用 `use` 语句来声明其使用。比如我们想使用 [random](https://crates.io/crates/random) 时，则必须在使用之前导入 random：

```rust
use random;
```



在 Rust 2018 之前，我们可能还会看到使用 `extern` 语句，比如：

```rust
extern crate random;
```



二者的效果是等价的。

`use` 语句有几种常见的使用方式，比如：

- **使用嵌套路径来消除大量的 `use` 行**

  **普通版本为**：

  ```rust
  use std::cmp::Ordering;
  use std::io;
  ```

  

  使用嵌套路径后：

  ```rust
  use std::{cmp::Ordering, io};
  ```

  

  可将多个带有相同前缀的项引入作用域。

  还可以使用 `self` 来处理如下场景：

  普通版本为：

  ```rust
  use std::io;
  use std::io::Write;
  ```

  

  使用 self 的嵌套路径为：

  ```rust
  use std::io::{self, Write};
  ```

  

- **通过 glob 运算符将所有的公有定义引入作用域**

  可以用 `*` 将**一个路径下所有公有项引入作用域**：

  ```rust
  use std::collections::*;
  ```

  

  常用于测试模块 `tests` 中，有时也用于 prelude 模式。

**我们还可以通过 `as` 关键字重命名引入作用域的类型**，如下所示：

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {}

fn function2() -> IoResult<()> {}
```



## module 的几种常见的组织形式

### `src/lib.rs` 总是根入口

当 package 或者 crate 作为 lib 发布时，其根入口总是 `src/lib.rs`。我们将会在 `lib.rs` 中定义或者声明其他 module。所以，当我们使用某一个 lib 的时候，可以优先先阅读 `src/lib.rs`，也就是说，**一个 lib 对外暴露的 item 总是在 `src/lib.rs` 中**。

### 常见的几种组织方式

1. **将所有逻辑都放在 `src/lib.rs` 中**

   这是最简单的一种组织形式，比如我们上面几个小节的例子都是采用了这种方式。我们可以把希望对外暴露的 item 都用 `pub` 关键字进行暴露，这样当其他项目引用 crate 的时候就可以直接使用。此时 `lib.rs` 可以有 1 个或多个 module。

2. **将不同的 module 放在 `src/` 的不同文件中，此时对应的文件名即是 module 名，并在 `src/lib.rs` 做相应的声明**

   比如我们分别可以创建 `src/foo.rs` 和 `src/bar.rs`:

   ```rust
   // src/foo.rs
   
   pub fn hello_foo() {
       println!("foo::hello_foo()")
   }
   ```

   

   在 `src/bar.rs` 中，则为：

   ```rust
   // src/bar.rs
   
   pub fn hello_bar() {
       println!("bar::hello_bar()")
   }
   ```

   

   如上所示，我们分别在 `foo.rs` 和 `bar.rs` 中创建了相应的公共函数。我们接着在 `lib.rs` 进行 module 的声明和使用：

   ```rust
   // src/lib.rs
   
   pub mod bar;
   pub mod foo;
   
   fn use_foo_and_bar() {
       crate::foo::hello_foo();
       crate::bar::hello_bar();
   }
   ```

   

   此时，**Rust 将会默认将对应文件名作为 module 名**，而无需显式使用 `mod {}` 语句。这时候，文件的组织形式为：

   

   ```sh
   src
   ├── bar.rs
   ├── foo.rs
   └── lib.rs
   
   ```

   

3. **将不同的 module 放在 `src/` 的不同文件目录中，此时对应的文件目录名即是 module 名，每个文件目录必须有一个 `mod.rs` 来声明对应文件目录下的 module，最后在 `src/lib.rs` 做声明**

   参考着上面的例子，我们可以创建 `src/foo/`，并创建 `src/foo/foo_mod1.rs` 和 `src/foo/foo_mod2.rs`:

   ```rust
   // src/foo/foo_mod1.rs
   
   pub fn hello_foo_mod1() {
       println!("foo::foo_mod1::hello_foo_mod1()")
   }
   ```

   

   在 `src/foo/foo_mod2.rs` 中，则为：

   ```rust
   // src/foo/foo_mod2.rs
   
   pub fn hello_foo_mod2() {
       println!("foo::foo_mod2::hello_foo_mod2()")
   }
   ```

   

   此时我们还需要创建 `src/foo/mod.rs` 做声明，此时 `mod.rs` 就相当于 `foo/` 下的 `lib.rs`：

   ```rust
   // src/foo/mod.rs
   
   pub mod foo_mod1;
   pub mod foo_mod2;
   ```

   

   我们最后再在 `src/lib.rs` 做声明并使用：

   ```rust
   // src/lib.rs
   
   pub mod foo;
   
   fn use_foo() {
       crate::foo::foo_mod1::hello_foo_mod1();
       crate::foo::foo_mod2::hello_foo_mod2();
   }
   ```

   

   这时候，`pub mod foo` 声明的 `foo` module 中其实是包含了 `foo_mod1` 和 `foo_mod2` 两个 module。此时对应的文件目录为：

   ```sh
   src
   ├── foo
   │   ├── foo_mod1.rs
   │   ├── foo_mod2.rs
   │   └── mod.rs
   └── lib.rs
   ```

   

## re-export 机制

Rust 支持采用 `pub use` 进行 re-export（重导出）。所谓的 re-export 可以简单理解为如下**行为**：

**如果 module A 定义了一个 item `foo`，而此时 module B 可访问到 `foo`，则可在 module B 通过使用 `pub use A::foo`（路径可以更为复杂），从而将 `foo` 这个 item 导出为 `B::foo`**。

这么做的好处是**可以建立一套与内部具体实现不一样的对外结构**，从而可以更好地屏蔽内部实现。

就拿上一节中最后一个例子，当我们需要在其他 crate 调用 `hello_foo_mod1()` 或者 `hello_foo_mod2()`时，我们需要写这么一串很长的路径（假设 crate 名为 `testlib`）：

```rust
fn main() {
    testlib::foo::foo_mod1::hello_foo_mod1();
}
```



绝对路径中的 `foo::foo_mod1` 部分有可能仅仅是内部实现，对外用户无需了解，此时我们可以在 `src/lib.rs` 重导出一个新的结构：

```rust
pub use foo::foo_mod1::hello_foo_mod1();
```



此时用户只需要使用更短更简洁的引用路径：

```rust
fn main() {
    testlib::hello_foo_mod1();
}
```



除此之外，`pub use` 还可以将对应 module 的私有 item 再重导出变为公共 item ，从而可有选择性地改变原有的私有属性，这在某些场景下比较有用。

## 更细粒度的 `pub` 控制

默认地，当某一个 module 被设置为 `pub` 时，则该 module 下的所有 `pub` item 都可直接被其他祖先 module 或者 module 之外的用户所使用，**可见范围比较大**。如果开发者只想针对某一个范围内实行 `pub` 可见性 ，此时 Rust 提供了另一种**更细粒度的控制方式**：

- **`pub(in path)`**

  让对应 item 只在指定 `path` 可见。`path` 必须是 item 的祖先 module。从 Rust 2018 开始，`path` 必须以 `crate`、`self` 或者 `super` 开头。

- **`pub(crate)`**

  item 可在当前 crate 内可见，也就是说出了 crate 就无法可见，当对应 crate 被其他 crate 引用时，无法看见这类 item。

- **`pub(super)`**

  item 只在父级 module 可见，等价于 `pub(in super)`；

- **`pub(self)`**

  item 只在当前 module 可见，等价于 `pub(in self)` 或者不使用 `pub`；

我们可以从下面这个例子（来源于参考文档 [1]）来看对应的用法：

```rust
// src/main.rs

pub mod outer_mod {
    pub mod inner_mod {
        // outer_mod_visible_fn 只能在 outer_mod 内可见
        pub(in crate::outer_mod) fn outer_mod_visible_fn() {}

        // 只在对应 crate 内可见
        pub(crate) fn create_visible_fn() {}

        // 只在 outer_mod 内可见
        pub(super) fn super_mod_visible_fn() {}

        // 只在 inner_mod 内可见，效果内与不加 pub 是一样的
        pub(self) fn inner_mod_visible_fn() {}
    }

    pub fn foo() {
        inner_mod::outer_mod_visible_fn();
        inner_mod::create_visible_fn();
        inner_mod::super_mod_visible_fn();

        // ERROR!
        // inner_mod_visible_fn() 只在 inner_mod 可见
        // inner_mod::inner_mod_visible_fn();
    }
}

fn bar() {
    outer_mod::inner_mod::create_visible_fn();

    // ERROR!
    // super_mod_visible_fn() 只在 outer_mod 内可见
    // outer_mod::inner_mod::super_mod_visible_fn();

    // ERROR!
    // outer_mod_visible_fn() 只在 outer_mod 内可见
    // outer_mod::inner_mod::outer_mod_visible_fn();

    outer_mod::foo();
}

fn main() {
    bar();
}
```



总结来说就是：**默认 `pub` 是一种可见范围较大的属性，而 Rust 支持在 `pub` 基础之上通过添加相应限制语句来缩小可见范围**。而在 Go 中，则没有这种细粒度控制可见范围的手段（可用特殊的 `internal/` 目录将可见范围缩小在 `internal` package 中），所以灵活性相对要差一些。

## Cargo 的 Workspace 机制

当一个项目足够大的时候，我们必须将 package 划分成多个 crate，此时就可以使用 Cargo 的 Workspace 机制。所谓 Workspace，就是**多个 crate 共享同一个 `target/` 和 `Cargo.lock`**。这样**可以节省编译时间和磁盘空间**，否则每一个 crate 单独维护一个 `target/`，也无法共享任何编译后的代码。

用 Cargo Workspace 来组织的项目，其项目最顶层的 `Cargo.toml` 会声明相应的 `members`，比如我们来看一看 [wasmer](https://github.com/wasmerio/wasmer/blob/master/Cargo.toml)（一个用 Rust 写的 wasm runtime）：

```toml
[workspace]
members = [
    "lib/api",
    "lib/cache",
    "lib/c-api",
    "lib/cli",
    "lib/compiler",
    "lib/compiler-cranelift",
    "lib/compiler-singlepass",
    "lib/compiler-llvm",
    "lib/derive",
    "lib/emscripten",
    "lib/engine",
    "lib/engine-universal",
    "lib/engine-dylib",
    "lib/engine-staticlib",
    "lib/object",
    "lib/vfs",
    "lib/vm",
    "lib/wasi",
    "lib/wasi-types",
    "lib/wasi-experimental-io-devices",
    "lib/types",
    "tests/wasi-wast",
    "tests/lib/wast",
    "tests/lib/compiler-test-derive",
    "tests/integration/cli",
    "tests/integration/ios",
    "fuzz",
]
```



`members` 的每一个成员都是对应目录下的文件夹且都是 crate。**此时我们无论在哪一个 crate 运行 `cargo build`，都只会在最外层 `target/` 生成构建数据**。

## Cargo 的依赖管理

### 如何引入第三方库

1. **直接使用 crates.io 的 package**

   这是最常见的一种方式。Rust 有一个生态非常健全的 crate registry：[crates.io](https://crates.io/)。我们只需要在对应的 `Cargo.toml` 中的 `dependencies` 中声明所使用的包名和版本即可，比如当我们想使用 [random](https://crates.io/crates/random)，只需要：

   

   ```toml
   [dependencies]
   random = "0.12.2"
   ```

   

   则 Cargo 在构建阶段就会从 crates.io 中下载相应版本的 package 到本地。

2. **使用本地 crate**

   还有一种常见的方式就是引用本地 crate。比如在 Cargo workspace 下 crate 之间相互引用的场景。此时可用 `path` 字段，比如：

   ```toml
   [dependencies]
   foo_lib = { path = "../foo" }
   ```

   

   `path` 指定了对应 crate 的路径。

3. **使用 git 仓库**

   还有一种情况，比如我们想使用的 package 并没有上传到 crates.io，且也不是同一个代码仓库下的项目（比如内部私有项目或者 GitHub fork 的版本），此时可以直接使用 git 仓库，比如：

   | `1 2 ` | `[dependencies] foo-apis = { git = "https://github.com/foo/foo-apis.git", branch = "master"} ` |
   | ------ | ------------------------------------------------------------ |
   |        |                                                              |

### 依赖版本决策和 `Cargo.lock`

不同于 Go Module 的依赖版本决策所使用的 [MVS 算法](https://zyy.dev/post/go-modules-in-10-minutes/)，Cargo 所使用的依赖版本决策算法要相对简单得多。Cargo 对于 crate 的版本也是使用语义化版本管理，同时也定义了一些**显式控制版本范围**的方式（可参考文档 [2]）。

Cargo 采用如下几个规则来进行依赖版本决策：

1. **当多个 package 共用同一个语义化版本兼容的依赖时，Cargo 将会选择符合语义化版本范围内的上游最新的版本**

   比如：

   ```toml
   [dependencies]
   foo-apis = { git = "https://github.com/foo/foo-apis.git", branch = "master"}
   ```

   

   若此时 bitflags 最新发布版为 1.2.1，则 **Cargo 将会使用 1.2.1**，因为从语义化版本的角度来看，1.2.1 可兼容 1.0 和 1.1。

2. **当多个 package 共用同一个语义化版本不兼容的依赖时，Cargo 将会各自使用其不兼容依赖版本上游最新版的拷贝**

   比如：

   ```toml
   # Package A
   [dependencies]
   bitflags = "1.0"
   
   # Package B
   [dependencies]
   bitflags = "1.1"
   ```

   

   从语义化版本来看，0.7 和 0.6 版本并不兼容，且此时 rand 上游分别有最新版的 0.7.3 和 0.6.5，则 Cargo 最终将会分别使用 0.7.3 和 0.6.5 这两个版本的拷贝。

**为了保证构建的稳定性**，Cargo 会将每个包确切的版本号都记录在 `Cargo.lock` 中，后续构建都将参考这个文件。**除非我们修改 `Cargo.toml` 中的依赖版本或者运行了 `cargo update`**。

与 Go Module 的 MVS 算法相比，Rust **总是使用最新版本的依赖**，而非满足全局版本约束条件下的最够用的版本，正是有这个差异，才导致 Rust 总是需要 `Cargo.lock` 来维护构建的稳定性，而 Go 则并不需要，因为它所选择的版本总是我们所声明的某个版本而非上游的某个新版本。

## 总结

相比于 Go Module，Rust 的包管理机制要更为复杂，但正是这种复杂性，带来了更大的灵活性。开发者可在这种灵活性之上更好地进行代码架构的抽象。

## 参考

1. [The Rust Reference: Visibility and Privacy](https://doc.rust-lang.org/reference/visibility-and-privacy.html)
2. [Dependency Resolution](https://doc.rust-lang.org/cargo/reference/resolver.html#dependency-resolution)
