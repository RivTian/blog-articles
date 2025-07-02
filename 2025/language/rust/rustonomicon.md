参考文档

* [官方英文版](https://doc.rust-lang.org/nomicon/)
* [中文翻译](https://learnku.com/docs/nomicon/2018)

## 一、安全代码

[原文](https://learnku.com/docs/nomicon/2018/10-initial-knowledge-of-security-and-non-security-codes/4703)

原则上开发者希望屏蔽底层细节（一个空的元组占用多少内存）。但是现实是，有些情况人们不得不为了性能或者和底层硬件打交道时关注这些。此时程序员通常有三种选择：

* 修改代码让编译器或者运行时环境做相关优化
* 采取某些古怪、繁琐的奇技淫巧以实现功能需求
* 使用另一种可以处理底层细节的语言重写代码（C语言）

然而，C 在使用中往往过于不安全（虽然有时是出于合理的原因）。尤其是在与其他语言交互的过程中，这种不安全性还会被放大。C 和与其交互的语言必须时刻小心地确认对方的行为，以防踩到舞伴的脚趾头。

不同于 C，Rust 是一种安全的编程语言。
和 C 相同的是，Rust 是一种非安全的编程语言。
准确的说，Rust 是一种同时包含安全和非安全特性的编程语言。

Rust 可以被看作两种编程语言的结合体：安全 Rust 和非安全 Rust。

安全 Rust 是一种真正的安全编程语言。如果你所有的代码都是用安全 Rust 写的，你永远也无需担心类型安全和内存安全，无需费神处理悬垂指针、释放后引用 (use-after-free)，或者其他各种未定义的行为。

非安全 Rust 和安全 Rust 的语法规则完全相同，只不过它允许你做一些另外的不安全的行为

分离安全与非安全 Rust 的价值在于，我们既可以享受像 C 那样的非安全语言的好处 —— 也就是对底层实现细节的控制，又不用处理 C 与其他安全语言集成时遇到的种种问题。

不过还是会遇到一些问题。最明显的。我们必须非常了解类型系统的全部默认要求，并在每次与非安全代码交互的时候检查它们。

### 1、安全与非安全代码的交互方式

使用 `unsafe` 关键字

unsafe 关键字有两层含义：

* 声明代码中存在编译器无法检查的安全规范
* 同时声明开发者会自觉遵守相关规范而不会主动破坏它。

使用关键字 unsafe 表明函数和 trait 的声明中存在不受编译器检查的规范。

* 对于函数，unsafe 意味着调用函数的开发者必须查阅函数的文档以确保他们的用法符合函数的安全要求。
    * unsafe 代码块/函数，表示所有代码都已经人工检查过符合相关规范。比如，传递给 `slice::get_unchecked` 的索引值都没有越界。
* 而对于 trait 的声明，unsafe 意味着实现 trait 的开发者必须查阅 trait 的文档以确保 trait 的实现符合其安全要求。

标准库也有一些非安全函数，包括：

* `slice::get_unchecked` 可接受不受检查的索引值，也就是存在内存安全机制被破坏的可能
* `mem::transmute` 将值重新解析成另一种类型，即允许随意绕过类型安全机制的限制（详情参考类型转换）
* 所有指向确定大小类型 (sized type) 的裸指针都有 offset 方法，当传入的偏移量越界时将导致未定义行为 (Undefined Behavior)。
* 所有 FFI（Foreign Function Interface）函数都是 unsafe 的，因为其他的语言可以做各种的操作而 Rust 编译器无法检查它。

在 Rust 1.0 中，有两个非安全的 trait：

* `Send` 是一个标志 trait（即没有任何方法的 trait），承诺所有的实现都可以安全地发送（move）到另一个线程。
* `Sync` 也是一个标志 trait，承诺线程可以通过共享的引用共享它的实现。

关于这两个特质

* Send 和 Sync 是会被各种类型自动实现的，只要这种实现可以被证明是安全的。
* 如果一种类型其所有的值的类型都实现了 Send，它本身就会自动实现 Send；
* 如果一种类型其所有的值的类型都实现了 Sync，它本身就会自动实现 Sync。
* 将它们设为 unsafe 实际减少了非安全代码的滥用。

许多 Rust 标准库其实内部也使用了非安全 Rust。这些库的实现方法都经过了严苛的人工检查，所以这些基于非安全 Rust 实现的安全 Rust 接口依然可以认为是安全的。

这种代码 隔离的存在说明了安全 Rust 的一个基本特征：

* 无论如何，安全 Rust 代码都不能导致未定义行为

安全和非安全的 Rust 之间存在一种不对称的信任关系：

* 安全 Rust 必须无条件信任非安全 Rust，假定所有与之打交道的非安全代码都是正确的。（因为非安全的 Rust 的实现者承诺了该不安全代码块是安全的，并为此负责，所以安全代码开发者应该无条件信任）
* 反过来，非安全 Rust 却要谨慎对待安全 Rust 的代码。（因为拥有不受控制的黑魔法，安全的东西也不是安全的，因此要小心）

一个比类比：

* unsafe就是国家暴力机关，拥有普通人不曾拥有的能力枪支，
* 而安全代码是普通人
* 系统环境就是社会
* 普通人应该充分信任国家暴力机关，因为，国家暴力机关承诺了正确使用枪支，不会破会社会稳定。
* 但是国家暴力机关无法完全信任普通人，因为普通人在拥有枪支能力后可能破会社会稳定，因此需要特别小心，克制的使用能力。

### 2、非安全 Rust 能做什么

非安全 Rust 比安全 Rust 可以多做的事情只有以下几个：

* 解引用裸指针
* 调用非安全函数（包括 C 语言函数，编译器内联函数，还有直接内存分配等）
* 实现非安全 trait
* 访问或修改可变静态变量

Rust 充分限制了可能出现的未定义行为的种类。语言核心只需要防止这几种行为：

* 解引用 null 指针，悬垂指针，或者未赋值的指针
* 读取未初始化的内存
* 破坏指针混淆规则
* 创建非法的基本类型：
    * 悬垂引用与 null 引用
    * 空的 fn 指针
    * 0 和 1 以外的 bool 类型值
    * 未定义的枚举类型的项
    * 在 [0x0,0xD&FF] 和 [0xE000, 0x10FFFF] 以外的 char 类型值
    * 非 utf-8 编码的 str
* 不谨慎地调用其他语言
* 数据竞争

Rust 对于一些模糊的操作则通常比较宽容。Rust 会认为下列操作是安全的：

* 死锁
* 竞争条件
* 内存泄漏
* 调用析构函数失败
* 整型值溢出
* 终止程序
* 删除产品数据库

当然，有以上行为的程序极有可能就是错误的。Rust 提供了一系列的工具减少这种事情的发生，但是完全地杜绝它们其实是不现实的。

### 3、编写非安全代码

* Rust 安全代码具有非本地性，即非安全代码的稳定性其实依赖于另一些 “安全” 代码的状态
* 将对安全代码的检测也需要放到 Unsafe 块中，也就是说：导致不安全代码不安全的安全代码也应该在不安全的代码块中（尽量）
* 非安全代码必须信任一部分安全代码，但是不应该信任所有的安全代码：因此私有成员的限制对于非安全代码很重要（将不信任的安全代码私有化）

## 二、内存布局

### 1、Rust 默认对齐规则

Rust 结构体内存布局按照一定规则进行对齐，默认规则如下：

* 首先确定对齐属性 `align`，计算规则如下
    * 遍历结构体树的所有基础数据类型，
    * 则 align = max(size_of(基础数据类型))
    * 基础数据类型的 size 如下 `size_of`
    * 特别说明
        * 在 x86 平台上 u64 和 f64 都是按照 32 位对齐的。
        * usize和isize的 size 是 目标平台地址大小。例如，在32位平台，4个字节，在64位平台，8个字节。
* 确定 `align` 后
    * 针对结构体：Rust 会对结构体成员进行排序，以提高内存利用率，让内存更紧凑，减少填充
    * 针对枚举：
        * 一般情况下编译 `size_of` 为内部结构体中 `max(size_of(内部成员)) + sizeof(u8)`
        * 但是针对类似与 `Option` 的结构则 `size_of::<Option<&T>>() == size_of::<&T>()` （空指针优化）

| Type              | `size_of::<Type>()`|
|--                 |--                  |
| `bool`            | 1                  |
| `u8` / `i8`       | 1                  |
| `u16` / `i16`     | 2                  |
| `u32` / `i32`     | 4                  |
| `u64` / `i64`     | 8                  |
| `u128` / `i128`   | 16                 |
| `f32`             | 4                  |
| `f64`             | 8                  |
| `char`            | 4                  |

```rust
// A  size_of = 8, align_of = 4
struct A {
    b: u32,
    c:u16,
    a: u8,
}

// A2 size_of = 8, align_of = 4
struct A2 {
    a: u8,
    b: u32,
    c:u16,
}

// A3 size_of = 8, align_of = 4
struct A3 {
    a: u8,
    b: u32,
    c: u16,
    _pad: [u8; 1], // 保证整体类型尺寸是4的倍数
                    // （译注：原文就是“4的倍数”，但似乎“32的倍数”才对）
}

// B  size_of = 2, align_of = 1
struct B {
    a: bool,
    b: i8,
}

// C  size_of = 12, align_of = 4
struct C {
    a: A,
    b: bool,
}

// D<u16, u32>  size_of = 8, align_of = 4
// D<u32, u16>  size_of = 8, align_of = 4
struct D<T, U> {
    count: u16,
    data1: T,
    data2: U,
}

// E  size_of = 0, align_of = 1
struct E {}

// 编译成 struct F { tag: u8}
// F  size_of = 1, align_of = 1
enum F {
    A,
    B,
    C,
}

// 枚举 null 指针优化（优化掉 tag）
// G<&i32>  size_of = 8, align_of = 8
// &i32  size_of = 8, align_of = 8
enum G<T> {
    Some(T),
    None,
}
```

### 2、特殊类型的内存布局

#### （1）动态尺寸类型（DST）

> [关于 DST / Sized / FatPointer 胖指针](https://zhuanlan.zhihu.com/p/21820917)

动态尺寸类型（DST），表现上为未实现 `Sized` 特质，即实现了 `?Sized`

* 内置的动态尺寸类型
    * `str` （本质上是 字符串切片）
    * 切片类型 `[Type]`
    * trait 对象
* 一个结构体最后一个字段包含了 DST（一个结构体只能包含一个），则该结构也将变成 DST
* 动态尺寸类型本质上是一个胖指针

胖指针和瘦指针

* 瘦指针，size_of 等于 `size_of(uszie)`，仅仅包含执行真实数据的指针值
* 胖指针，size_of 等于 `2*size_of(uszie)`，除了包含指针值外，还包含其他信息比如长度
    * 针对切片类型（以数组切片为例）
        * 数组的 `size_of` 为 `len*size_of(元素类型)`，长度确定
        * 数组的引用的 `size_of` 为 `size_of(usize)`，即为瘦指针
        * 数组切片的 `size_of` 编译期 不可知
        * 数组切片的引用 `size_of` 可知，为 `size_of(usize)*2`，即为胖指针，因为该指针多存储了一个长度字段
    * Trait Object
        * `dyc trait` 类型编译期间 不可知
        * `&dyc trait` 类型（即trait 对象的指针）编译期间可知，为 `size_of(usize)*2`，即为胖指针，包含两个指针
            * `&self` 指向结构体
            * `&vtable` 指向虚函数表
        * 多重trait派发内存布局参见 [文章](https://www.oschina.net/translate/exploring-dynamic-dispatch-in-rust?lang=chs&p=1)

#### （2）零尺寸类型 (ZST, Zero Sized Type)

* 无成员结构体 `struct J {}`
* 空元祖 `()`
* 尺寸为0的数组 `[i32;0]`

#### （3）空类型

* 例如 `enum Void {}`

#### （4）本节代码

```rust
use std::mem::{size_of, align_of};


trait I {}

struct J {}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch02() {

        // println!("H  size_of = {}, align_of = {}", size_of::<H>(), align_of::<H>());
        println!("[i32; 3]  size_of = {}, align_of = {}", size_of::<[i32; 3]>(), align_of::<[i32; 3]>());
        println!("&[i32; 3]  size_of = {}, align_of = {}", size_of::<&[i32; 3]>(), align_of::<&[i32; 3]>());
        // println!("[i32]  size_of = {}, align_of = {}", size_of::<[i32]>(), align_of::<[i32]>());
        println!("&[i32]  size_of = {}, align_of = {}", size_of::<&[i32]>(), align_of::<&[i32]>());

        println!("dyn I  size_of = {}, align_of = {}", size_of::<&dyn I>(), align_of::<&dyn I>());

        println!("J  size_of = {}, align_of = {}", size_of::<J>(), align_of::<J>());
        println!("()  size_of = {}, align_of = {}", size_of::<()>(), align_of::<()>());
        println!("[i32;0]  size_of = {}, align_of = {}", size_of::<[i32;0]>(), align_of::<[i32;0]>());
    }
}
```

### 3、其他 内存布局方式

* `repr(C)` 这是最重要的一种 repr。它的目的很简单，就是和 C 保持一致。数据的顺序、大小、对齐方式都和你在 C 或 C++ 中见到的一摸一样。所有你需要通过 FFI 交互的类型都应该有 repr(C)，因为 C 是程序设计领域的世界语。而且如果我们要在数据布局方面玩一些花活的话，比如把数据重新解析成另一种类型，repr(C) 也是很有必要的。

更多参见 [文章](https://learnku.com/docs/nomicon/2018/23-other-reprs/4710)

## 三、所有权

### 1、引用

引用本质上是加上了约束和一些规则的指针，是受限指针。

有两种引用的类型：

* 共享指针：`&`
* 可变指针：`&mut`

它们遵守以下的规则：

* 引用的生命周期不能超过被引用内容
* 可变引用不能存在别名 (alias)，这种限制类似于读写锁
    * 针对一个变量，当存在读写针的时候，则所有的读指针均失效，且只允许存在一个写指针
    * 允许存在多个读指针

很不幸，Rust 实际上没有定义别名模型。

### 2、别名

遵循了上述的别名规则，编译器可以大胆的进行编译优化

```rust
// 如果不允许出现别名，可以进行优化
fn compute(input: &u32, output: &mut u32) {
    if *input > 10 {
        *output = 1;
    }
    if *input > 5 {
        *output *= 2;
    }
}

// 前提条件是不允许出现如下可能性
// compute(&x, &mut x)
fn compute2(input: &u32, output: &mut u32) {
    let cached_input = *input; // 将*input放入缓存
    if cached_input > 10 {
        *output = 1*2; // x > 5 则必然 x > 5，所以直接加倍并立即退出
    } else if cached_input > 5 {
        *output *= 2;
    }
}
```

### 3、生命周期

Rust 在整个生命周期里强制执行生命周期的规则。

生命周期说白了就是**作用域**的名字。每一个**引用**以及**包含引用的数据结构**，都要有一个生命周期来指定它保持有效的作用域。

在函数体内（比如main函数内），一般不需要显式地给生命周期起名字。因为在本地上下文里，一般没有必要关注生命周期。Rust 知道代码的全部信息，从而可以完美地执行各种操作。它可能会引入许多匿名或者临时的作用域让程序顺利执行。

但是如果你要跨出函数的边界，就需要关心生命周期了。生命周期用这样的符号表示：`'a`, `'static`。为了更清晰地了解生命周期，我们假设我们可以为生命周期打标签，去掉本章所有例子的语法糖。

每一个 let 表达式都隐式引入了一个作用域

已加有几个例子的生命周期分析

```rust

fn life(){
    // 例子1
    {
        let x = 0;
        let y = &x;
        let z= &y;
    }
    // 例子1 生命周期展开
    // 'a: {
    //     let x: i32 = 0;
    //     'b: {
    //         // 生命周期是'b，因为这就足够了
    //         let y: &'b i32 = &'b x;
    //         'c: {
    //             // 'c也一样
    //             let z: &'c &'b i32 = &'c y;
    //         }
    //     }
    // }

    // 例子2
    {
        let x = 0;
        let z;
        let y = &x;
        z = y;
    }
    // // 例子2 生命周期展开
    // 'a: {
    //     let x: i32 = 0;
    //     'b: {
    //         let z: &'b i32;
    //         'c: {
    //             // 必须使用'b，因为引用被传递到了'b的作用域
    //             let y: &'b i32 = &'b x;
    //             z = y;
    //         }
    //     }
    // }
}

// fn as_str(data: &u32) -> &str {
//     let s = format!("{}", data);
//     &s // 生命周期错误
// }

// // 生命周期展开分析
// fn as_str<'a>(data: &'a u32) -> &'a str {
//     'b: {
//         let s = format!("{}", data);
//         return &'a s; // 'a > 'b  与现实相悖
//     }
// }

fn life_mut() {
    {
        let mut data = vec![1, 2,3];
        // 以下三句等价：因此在此创建了 data 的不可变引用
        let x = &data[0];
        // let x = Index::index(& data, 0);
        // let data_ref = & data; let x = Index::index(data_ref, 0);

        // 以下两句等价
        // data.push(4); // 如果该语句存在，则在此作用域及父作用域内同时存在一个变量的共享指针和可变指针，所以违反规则
        // let data_mut_ref = &mut data; Vec::push(data_mut_ref, 4);
        println!("{}", x);
    }
    // 生命周期展开分析
    // {
    //     'a: {
    //         let mut data: Vec<i32> = vec![1, 2, 3];
    //         'b: {
    //             // 对于这个借用来说，'b已经足够大了
    //             // （借用只需要在println!中生效即可）
    //             let x: &'b i32 = Index::index::<'b>(&'b data, 0);
    //             'c: {
    //                 // 引入一个临时作用域，因为&mut不需要存在更长时间
    //                 // 在此作用域及父作用域内同时存在一个变量的共享指针和可变指针，所以违反规则
    //                 Vec::push(&'c mut data, e);
    //             }
    //             println!("{}", x);
    //         }
    //     }
    // }

}
```

### 4、省略生命周期

为了让语言的表达方式更人性化，Rust 允许函数的签名中省略生命周期。

省略的规则如下（输入指函数参数生命周期、输出值返回值生命周期注解）：

* 每一个在输入位置省略的生命周期都对应一个唯一的生命周期参数。
* 如果只有一个输入的生命周期位置（无论省略还是没省略），那个生命周期会赋给所有省略了的输出生命周期。
* 如果有多个输入生命周期位置，而其中一个是 &self 或者 &mut self，那么 self 的生命周期会赋给所有省略了的输出生命周期。
* 除了上述情况，其他省略生命周期的情况都是错误的。

例子如下

```rust
fn print(s: &str);                                      // 省略的
fn print<'a>(s: &'a str);                               // 完整的

fn debug(lvl: usize, s: &str);                          // 省略的
fn debug<'a>(lvl: usize, s: &'a str);                   // 完整的

fn substr(s: &str, until: usize) -> &str;               // 省略的
fn substr<'a>(s: &'a str, until: usize) -> &'a str;     // 完整的

fn get_str() -> &str;                                   // 错误

fn frob(s: &str, t: &str) -> &str;                      // 错误

fn get_mut(&mut self) -> &mut T;                        // 省略的
fn get_mut<'a>(&'a mut self) -> &'a mut T;              // 完整的

fn args<T: ToCStr>(&mut self, args: &[T]) -> &mut Command                  // 省略的
fn args<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command;// 完整的

fn new(buf: &mut [u8]) -> BufWriter;                    // 省略的
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a>;         // 完整的

fn f<'a>(a: &'a i32, b: &i32, c: &i32, ) -> &'a i32 {
    a
}
```

### 5、未绑定的（无界）生命周期

> https://juejin.im/entry/5942159ca0bb9f006b8325a8

形如下所示的生命周期，为未绑定的生命周期

* 带有声明周期泛型且参数中没有使用，且返回值中使用的情况
* 返回的引用的生命周期无限大（比 'static 还大）

```rust
fn foo2<'a>() -> &'a u32
```

该函数声明在 static 变量情况下可以表现正常

```rust
    static global_int: u32 = 0;
    fn foo2<'a>() -> &'a u32 {
        return &global_int;         // ok, globals live longer than everything
    }
    let i = foo2();
    let i: &'static u32 = foo2();
```

但是当使用 Unsafe 时将可能破坏 Rust 生命周期 (常见的场景就是解裸指针)

```rust
    fn foo<'a>(input: *const u32) -> &'a u32 {
        unsafe {
            return &*input
        }
    }
    let x;
    {
        let y = 7;
        x = foo(&y);
    }
    println!("hello: {}", x);
```

因此建议

* 尽早确认生命周期边界
* 使用 省略生命周期 的方式编写涉及引用的函数

如果非要解裸指针建议将 最好整个函数声明为 unsafe

### 6、高阶 trait 边界（HRTB）

当一个 结构体 包含一个泛型参数，且要该泛型参数实现为`Fn`时，可能需要使用 高阶 trait 边界（HRTB, Higher-Rank Trait Bounds），基本语法为：`where for<'a> F: Fn(&'a (u8, u16)) -> &'a u8,`

```rust
// 高阶函数的生命周期
struct Closure<F> {
    data: (u8, u16),
    func: F
}

impl<F> Closure<F>
    where F: Fn(&(u8, u16)) -> &u8,
{
    fn call(&self) -> &u8 {
        (self.func)(&self.data)
    }
}

// 以上等价于
struct Closure2<F> {
    data: (u8, u16),
    func: F
}

impl<F> Closure2<F>
    where for<'a> F: Fn(&'a (u8, u16)) -> &'a u8, // 高阶 trait 边界（HRTB, Higher-Rank Trait Bounds）
{
    fn call<'a>(&'a self) -> &'a u8 {
        (self.func)(&self.data)
    }
}

fn do_it(data: &(u8, u16)) -> &u8 { &data.0 }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch03() {
        let clo = Closure { data: (0, 1), func: do_it };
        println!("{}", clo.call());
        let clo = Closure2 { data: (0, 1), func: do_it };
        println!("{}", clo.call());
    }
}
```

### 7、子类型和变性

> 本节建议看原文 https://doc.rust-lang.org/nomicon/subtyping.html 不要看译文
> 本节的例子是伪代码，无法运行，且涉及类型系统，难以理解。大多数规则是符合直觉的，可以遇到问题再看。
> 这些规则一般在泛型具象化的时候才会应用

#### 生命周期类型化

生命周期子类型（例子参见：[serde 反序列化](https://github.com/rectcircle/rust-serde-learn/blob/master/src/ch05_02_deserializer.rs)）

`'big: 'small` （读作 big 包含 small，或者 big 比 small 长寿），那么 `'big` 就是 `'small` 的子类型。原因如下：

* 需要 `'small` 的地方，给了 `'big` 当然也是可以的（因为 `'big` 比 `'small` 长寿）
* 类比继承 `Dog: Animal`，需要 `Animal`  的地方给 `Dog` 也是当然也是可以的

因此 `'static` 是所有生命周期的子类型

> 子类型理解：当需要用到 `B` 的 地方 用户给了 `A`，则可以称 `A` 是 `B` 的子类型记做 `A:B`

#### 变性

构造函数 F 的变性表示了它的输入的子类型如何影响它输出的子类型。Rust 中有三种变性：

* 如果当 T 是 U 的子类型时，`F<T>` 也是 `F<U>` 的子类型，则 `F` 对于 `T` 是协变的
* 如果当 T 是 U 的子类型时，`F<U>` 是 `F<T>` 的子类型，则 `F` 对于 `T` 是逆变的
* 其他情况（即子类型之间没有关系），则 F 对于 T 是不变的

Rust 中的情况

* `&'a T` 对于 `'a` 和 `T` 是协变的
    * 需要 `'a` 的地方，用户可以给 `'a` 的子类型（即生命周期大于 `'a` 的类型）
    * 需要 `T` 的地方，用户可以给 `T` 的子类型
* `&'a mut T` 对于 `'a` 是协变的，对于 `T` 是不变的
    * 需要 `'a` 的地方，用户可以给 `'a` 的子类型（即生命周期大于 `'a` 的类型）
    * 需要 `T` 的地方，用户可以给 `T` 的子类型，因为是T mut 引用，很有可能会被赋值导致类型错乱
* `fn(T) -> U` 对于 `T` 是逆变的，对于 `U` 是协变的（以下例子考虑为：一个函数需要传递一个函数作为参数）
    * 需要 `fn get_animal() -> Animal` 的地方，用户可以传递 `fn get_animal() -> Cat`
    * 需要 `fn handle_animal(Animal)` 的地方，用户不可以传递 `fn handle_animal(Cat)`
        * 原因，调用的时候，需要将 `Animal` 转化为 `Cat` 显然不行
    * 需要 `fn handle_animal(Cat)` 的地方，用户可以传递 `fn handle_animal(Animal)`
        * 原因，调用的时候，需要将 `Cat` 转化为 `Animal` 显然不行
* `Box`、`Vec` 以及所有的集合类对于它们保存的类型都是协变的
* `UnsafeCell<T>`、`Cell<T>` 、`RefCell<T>`、`Mutex<T>` 和其他的内部可变类型对于 `T` 都是不变的

总结

* 只有函数参数是 逆变 的
* 针对 `'a` 生命周期都是 协变的
* 针对引用/指针
    * 不可变意味着 协变
    * 可变意味着 不变

### 8、Drop检查

> 参考： https://learnku.com/docs/nomicon/2018/39-drop-examination/4720

针对 `'a: 'b` 的情况，并不是 `'a > 'b`，而是 `'a >= 'b`，是非严格大于。

但是针对 `Drop` 复杂情况，编译器要求，严格大于，即

**一个安全地实现 Drop 的类型，它的泛型参数生命周期必须严格地长于它本身**

### 9、`PhantomData` 👻幽灵数据

#### 用例1：包装未使用的生命周期

PhantomData的最常见用例可能是具有未使用的生存期参数的结构，通常作为某些不安全代码的一部分。例如，这是一个结构切片，它具有两个*const T类型的指针，大概指向数组的某个地方：

```rust
struct Slice<'a, T> {
    start: *const T,
    end: *const T,
}
```

`'a` 的作用是，该结构体的生命周期必须小于 `'a` 的生命周期，因为没有使用生命周期 `'a`，因此尚不清楚它适用于什么数据。我们可以通过告诉编译器将其作为`Slice`结构包含引用`＆'a T`来进行纠正：

```rust
use std::marker::PhantomData;

struct Slice<'a, T: 'a> {
    start: *const T,
    end: *const T,
    phantom: PhantomData<&'a T>,
}
```

使用上

```rust
fn borrow_vec<T>(vec: &Vec<T>) -> Slice<'_, T> {
    let ptr = vec.as_ptr();
    Slice {
        start: ptr,
        end: unsafe { ptr.add(vec.len()) },
        phantom: PhantomData,
    }
}
```

#### 用例2：Unused type parameters

有时，您会遇到未使用的类型参数，这些参数指示结构“绑定”到的数据类型，即使实际上并没有在结构本身中找到该数据。这是FFI出现的示例。外部接口使用 `*mut()` 类型的句柄来引用不同类型的Rust值。我们使用包裹外部句柄的struct ExternalResource结构上的幻像类型参数来跟踪Rust类型。

```rust
use std::marker::PhantomData;
use std::mem;

struct ExternalResource<R> {
   resource_handle: *mut (),
   resource_type: PhantomData<R>,
}

impl<R: ResType> ExternalResource<R> {
    fn new() -> ExternalResource<R> {
        let size_of_res = mem::size_of::<R>();
        ExternalResource {
            resource_handle: foreign_lib::new(size_of_res),
            resource_type: PhantomData,
        }
    }

    fn do_stuff(&self, param: ParamType) {
        let foreign_params = convert_params(param);
        foreign_lib::do_stuff(self.resource_handle, foreign_params);
    }
}
```

#### 用例3：所有权与Drop检查

参见用例1，编译器会进行借用检查和所有权检查

* `PhantomData<T>` 结构体会拥有 `T` 的所有权
* `PhantomData<&'a T>` 结构体会检查 `T` 引用的生命周期范围

### 10、引用分解(Split)

将 一个 引用分解成两个引用，例如：安全获取一个数据的前一部分和后一部分的切片，针对这种情况，rust 使用非安全的 代码实现，基于此方法，用户代码可以做到安全。

更多参见 https://doc.rust-lang.org/nomicon/borrow-splitting.html
