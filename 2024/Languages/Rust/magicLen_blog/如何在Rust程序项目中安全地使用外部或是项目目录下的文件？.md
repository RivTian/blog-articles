#Rust 

在开发程序的时候难免会需要在程序中引入外部的文件，为了方便管理我们也常会将这些文件放置在程序项目目录下。然而在代码撰写程序路径于运行阶段读取文件时，文件路径的正确性需要等到运行阶段的时候才会知道，就算写错了而找不到这个文件，程序项目也是能成功通过编译，这就会使得程序在运行阶段有出现问题的可能。

Rust内置的`include`、`include_str`、`include_bytes`宏会在编译阶段去检查文件是否存在，这是因为在编译阶段的时候就要把文件引用进来。但有时候我们会希望文件是在程序运行阶段才引入，所以就要有个机制是让Rust能够在编译项目的时候，去检查文件路径是否存在。

### Manifest Dir Macros

「Manifest Dir Macros」是笔者开发的套件，提供了一些类似函数的宏，能够在编译阶段检查或是操作相对于`CARGO_MANIFEST_DIR`的文件路径。`CARGO_MANIFEST_DIR`就是Cargo程序项目的`Cargo.toml`的所在目录，也就是程序项目的根目录啦！

Crates.io

[https://crates.io/crates/manifest-dir-macros](https://crates.io/crates/manifest-dir-macros)

Cargo.toml

```toml
manifest-dir-macros = "0.1.18"
```

#### 使用方法

`manifest_dir_macros`这个crate提供了以下几个宏：

- `absolute_path`：只允许传入的路径为绝对路径。回传绝对路径。
- `directory_absolute_path`：只允许传入的路径为绝对路径，且是个目录(必须存在)。回传绝对路径。
- `directory_path`：允许传入的路径可为绝对路径或相对路径。如果是相对路径，会使其相对于`CARGO_MANIFEST_DIR`。且是个目录(必须存在)。回传绝对路径。
- `directory_relative_path`：只允许传入的路径为相对路径，会使其相对于`CARGO_MANIFEST_DIR`，且是个目录(必须存在)。回传绝对路径。
- `exist_absolute_path`：只允许传入的路径为绝对路径，且必须存在。回传绝对路径。
- `exist_path`：允许传入的路径可为绝对路径或相对路径。如果是相对路径，会使其相对于`CARGO_MANIFEST_DIR`。且必须存在。回传绝对路径。
- `exist_relative_path`：只允许传入的路径为相对路径，会使其相对于`CARGO_MANIFEST_DIR`，且必须存在。回传绝对路径。
- `file_absolute_path`：只允许传入的路径为绝对路径，且是个文件(必须存在)。回传绝对路径。
- `file_path`：允许传入的路径可为绝对路径或相对路径。如果是相对路径，会使其相对于`CARGO_MANIFEST_DIR`。且是个文件(必须存在)。回传绝对路径。
- `file_relative_path`：只允许传入的路径为相对路径，会使其相对于`CARGO_MANIFEST_DIR`，且是个文件(必须存在)。回传绝对路径。
- `get_extension`：回传传入的路径的扩展名。如果没有指定默认值，路径中的扩展名不存在的话会编译失败。
- `get_file_name`：回传传入的路径的文件名称。如果没有指定默认值，路径中的文件名称不存在的话会编译失败。
- `get_file_stem`：回传传入的路径的不包含扩展名的文件名称(主文件名)。如果没有指定默认值，路径中的主文件名不存在的话会编译失败。
- `get_parent`：回传传入的路径的上一层文件路径。如果没有指定默认值，路径已经是最上一层的话会编译失败。
- `mime_guess`：须激活`mime_guess`特色才能使用。借由文件路径(或者准确的说，路径中的扩展名)，来猜测并回传文件的MIME Type，如果没有指定默认值，且无法猜到文件的MIME Type的话会编译失败。
- `not_directory_absolute_path`：只允许传入的路径为绝对路径，且不是个目录(必须存在)。回传绝对路径。
- `not_directory_path`：允许传入的路径可为绝对路径或相对路径。如果是相对路径，会使其相对于`CARGO_MANIFEST_DIR`。且不是个目录(必须存在)。回传绝对路径。
- `not_directory_relative_path`：只允许传入的路径为相对路径，会使其相对于`CARGO_MANIFEST_DIR`，且不是个目录(必须存在)。回传绝对路径。
- `path`：允许传入的路径可为绝对路径或相对路径。如果是相对路径，会使其相对于`CARGO_MANIFEST_DIR`。回传绝对路径。
- `relative_path`：只允许传入的路径为相对路径，会使其相对于`CARGO_MANIFEST_DIR`。回传绝对路径。

传给宏的文件路径可以是一个字符串定数，也可以是多个用逗号隔开的文件名(字符串定数)，宏会根据编译环境的操作系统将这些文件名其加上适当的斜线或反斜线，串接在一起成一个路径。

如果有激活`tuple`特色，还可以用嵌套的「元组(tuple)」，来串接多个文件名(字符串定数)，这个用法可以与`macro_rule`宏搭配使用。

范例代码如下：

```rust
#[macro_use] extern crate manifest_dir_macros;
 
println!(path!("Cargo.toml"));
println!(path!("src/lib.rs"));
println!(path!("src", "lib.rs"));
println!(path!("src", "lib.rs", "/bin"));
println!(path!("/usr"));
 
println!(exist_path!("Cargo.toml"));
println!(directory_path!("src"));
println!(not_directory_path!("Cargo.toml"));
println!(file_path!("Cargo.toml"));
 
println!(relative_path!("Cargo.toml"));
println!(directory_relative_path!("src"));
println!(not_directory_relative_path!("Cargo.toml"));
println!(file_relative_path!("Cargo.toml"));
 
println!(get_file_name!("src/lib.rs"));
println!(get_file_name!(default = "main.rs", "/"));
println!(get_file_stem!("src/lib.rs"));
println!(get_file_stem!(default = "lib", "/"));
println!(get_extension!("src/lib.rs"));
println!(get_extension!(default = "rs", "src/lib"));
println!(get_parent!("src/lib.rs"));
println!(get_parent!(default = "/home", "/"));
 
// enable the `mime_guess` feature
{
    println!(mime_guess!("src/lib.rs"));
    println!(mime_guess!(default = "application/octet-stream", "Cargo.lock"));
}
 
// The `tuple` feature lets these macros above support to input nested literal string tuples, which is useful when you want to use these macros inside a `macro_rule!` macro and concatenate with other literal strings.
// `$x:expr` matchers can be used in these macros thus.
{
    println!(path!(("foo",)));
    println!(path!(("foo", "bar")));
    println!(path!("a", ("foo", "bar")));
    println!(path!(("foo", "bar"), "a"));
    println!(path!(("foo", "bar"), ("a", "b")));
    println!(path!(("foo", "bar", ("a", "b")), ("c", "d")));
}
```