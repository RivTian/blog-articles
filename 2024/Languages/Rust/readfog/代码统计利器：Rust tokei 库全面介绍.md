#Rust 

## 引言

作为程序员，我们常常需要统计项目中的代码行数，以了解项目规模和进度。市面上有很多代码统计工具，但不少工具存在统计不准、语言支持不全、性能不佳等问题。今天给大家介绍一个 Rust 生态中的代码统计利器：tokei。tokei 通过语法分析准确统计代码行数，目前已支持 200+ 种语言，而且性能卓越，堪称业界良心之作。

## 创作背景

笔者最近在学习 Rust 语言，接触到了 tokei 这个库。该库的设计非常优雅，代码也很简洁，给笔者留下了深刻印象。因此希望通过此文，将 tokei 介绍给更多的 Rust 开发者和程序员，希望大家能从中获益。

## 主要特性

tokei 主要有以下特性：

- 支持 200 多种语言，覆盖主流语言
    
- 通过语法分析准确统计代码、注释、空行数
    
- 支持排除指定目录，如 node_modules 等
    
- 支持输出 JSON、YAML、CBOR 等结构化格式
    
- 支持输出到终端，并且支持颜色高亮和排序
    
- 拥有命令行工具和 Rust API 两种使用方式
    
- 统计性能优异，可在数秒内分析百万行代码库
    

## 快速上手

tokei 可通过 Cargo 一键安装：

```sh
cargo install tokei
```

然后就可以在命令行使用了：

统计当前目录下 Rust 代码行数

```sh
tokei ./src
```

统计指定语言，输出 JSON

```sh
tokei ./src --type=Rust,Markdown --output json
```

排除指定的 `Kan.md` 文件

```sh
tokei ./src -e kan.md
```

除了命令行，tokei 也可以直接在 Rust 代码中使用。

首先需要导入相关的 crate 到 `Cargo.toml` 文件中：

```toml
[dependencies]  
tokei="13.0.0-alpha.0"
```

编写我们的代码：

```rust
use tokei::Config;  
use tokei::Languages;  
  
fn main() {  
    // 待统计的目录  
    let paths = &["./src"];  
    // 要排除的目录  
    let excludes = &["./target"];  
    // tokei 配置  
    let config = Config::default();  
  
    // 执行统计  
    let mut languages = Languages::new();  
    languages.get_statistics(paths, excludes, &config);  
  
    // 打印结果  
    for (name, language) in &languages {  
        println!("{}: {} lines", name, language.code);  
    }  
}
```

输出：

```sh
Rust: 1621 lines  
Markdown: 432 lines  
TOML: 37 lines
```

## 总结

本文介绍了 Rust 生态中优秀的代码统计库 tokei，它通过语法分析实现了准确统计，支持 200 多种语言，拥有命令行工具和 API 两种使用方式。tokei 的实现非常优雅，代码简洁，也是学习 Rust 编程的很好示例。感兴趣的读者可以阅读 tokei 源码，必有收获。

## 参考文章

- tokei 文档：https://docs.rs/tokei/latest/tokei/
    
- tokei Github：https://github.com/XAMPPRocky/tokei