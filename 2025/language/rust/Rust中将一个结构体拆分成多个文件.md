官方文档将[一个模块拆分成多个文件](https://kaisery.github.io/trpl-zh-cn/ch07-05-separating-modules-into-different-files.html)时，介绍的是将原来多个模块写在同一个文件中，拆分成了每个模块一个文件。不过还有一种情况没有提到，如果一个模块中的某个 struct 实现代码过多时，仍写在同一个模块文件的话，维护成本就显的比较高了，这时我们可能还需要对这个 struct 的实现按某种粒度拆分成多个文件来实现。

```sh
$ tree
.
├── main.rs
├── model
│   ├── article.rs // 文章相关
│   └── user.rs // 用户相关
└── model.rs
```

这里是按官方教程拆分后的样子

- `article.rs` 是文件模块相关实现 -
- `user.rs` 是与用户相关的实现
- `model.rs` 公开模块



model.rs

```rust
// src/model.rs

pub mod article;
pub mod user;
```

`pub` 关键字表示该模块是公开的，可以被其他模块访问。

`mod article` 声明了一个名为 `article` 的模块，并且 Rust 编译器会在同文件名的目录下( `src/model/` )找到一个名为 `article.rs` 或者 `src/article/mod.rs` 的文件来实现这个模块。

文章模块`article.rs`

```rust
// src/model/article.rs
pub struct Article {
    id: u32,
    title: String,
}

impl Article {
    pub fn new(id: u32, title: String) -> Self {
        Article { id, title }
    }

    pub fn info(&self) {
        println!("Article info [id: {}, title: {}]", self.id, self.title);
    }
}
```

用户模块 `user.rs`

```rust
// src/model/user.rs

// Student
pub struct Student {}

impl Student {
    pub fn print() {
        println!("this is student")
    }
}
```

主函数 `main.rs`

```rust
// src/main.rs
use crate::model::article; // 导入模块 (先声明，后引入)
use crate::model::user;

mod model; // 声明模块

fn main() {
    // article info
    let art = article::Article::new(10, "How to read a book".to_string());
    art.info();

    // user print
    user::Student::print();
}
```

在主函数中，先通过 `mod model` 声明模块，然后再通过 `use` 引入模块。这里它们作用域是相同的，所以顺序无所谓，不过一般是先通过 mod 声明，然后在需要的时候再 use 引入。

以上是多数项目的结构。

但是，当随着业务的增加，Student 添加的方法越来越多，如果全部写在`user.rs`一个文件里，后期维护起来可能很不方便。那能否按一定的功能将结构体的实现拆分成多个文件里呢？如对 db 的操作单独在 `user_db.rs`文件实现，而搜索单独在 `user_search.rs`实现呢？答案是肯定的。

# 方法拆分

这里以 `user_db.rs` 为例，目录结构

```sh
$ tree model
model
├── article.rs
├── user.rs
├── user_db.rs
```

`user_db.rs`文件

```rust
// src/model/user_db.rs

use super::user::Student; // 引入模块中的结构体Student

impl Student {
    pub fn query() {
        println!("query");
    }
    pub fn insert() {
        println!("insert");
    }
}
```

首先通过 `use super::user::Student;` 引入结构体，这里`super` 关键字用于指向当前模块的父模块 `model`，通过 `::`依次引入 `user`模块中的 `Student` 结构体。

接着就可以通过 `impl` 来实现结构体的相关方法了。

# 模块方法暴露

不过当前 user_db 模块里对结构体添加的 `query()` 和 `insert()` 方法还没有办法被其它地方调用，因此还需要在 `model.rs` 文件中暴露出去。

在 `src/model.rs`文件末尾添加一行

```rust
pub mod user_db;
```

通过 pub 关键字表示此 user_db 模块(user_db.rs) 里的内容向外公开，这样才可以被其它地方调用。

好了，现在改行工作已经结束，下面让我们测试一下结果。

# 验证

在 `main.rs` 里添加调用数据库方法代码

```rust
// src/main.rs
use crate::model::article; // 导入模块 (先声明，后引入)
use crate::model::user;

mod model; // 声明模块

fn main() {
    // article info
    let art = article::Article::new(10, "How to read a book".to_string());
    art.info();

    // user print
    user::Student::print();

    // 调用数据库方法
    user::Student::query();
    user::Student::insert();
}
```

如果不出意外的话，程序执行一切正常。

# 总结

本文主要介绍将一个结构体的实现如何拆分成多个文件，以便减少当一个结构体实现代码量过大导致的维护成本过高的问题。

当在一个新文件实现已存在 struct 时，首先需要将这个 struct 引入到当前文件，然后才可以通过 impl 编写相关代码，最后还需要记得将这个文件通过 `pub mod `公开出去，不然此模块方法没有办法被调用。

每当创建一个新文件时，这个文件即被视为一个新模块，所以需要通过 `pub mod `关键字来公开此模块内容，这样才可以在其它地方调用此模块里的内容。

本例中 user_db 模块是对在其它模块 user 中声明结构体 Student 的继续实现，因此调用时，需要指定 struct 声明时所在的模块 user，而结构体对数据库操作的方法实现在 user_db 模块中，因此还需要将 user_db 模块通过 `pub mod user_db` 进行分开因此调用时使用 `user::Student::query()`, 而不是 `user_db::student::query()`。

对于调用语句也可以拆分成两部分理解，通过这个`user::Student` 找到要调用的结构体，而对于 `query()` 方法已经通过在 `src/model.rs` 文件中通过 `pub mod user_db` 公开，因此可以直接调用 `user::Student::query()`，如果不公开 user_db 的话，则系统将找不到这个方法。

# 参考资料

- [将模块拆分成多个文件](https://kaisery.github.io/trpl-zh-cn/ch07-05-separating-modules-into-different-files.html#将模块拆分成多个文件)