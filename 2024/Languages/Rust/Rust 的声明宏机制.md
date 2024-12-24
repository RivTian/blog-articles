#Rust

## 背景

Rust 宏编程是这门语言比较有趣但又难以掌握的知识点，而且在大多数项目中使用频度并不算高。本文**尝试性**地总结 **Rust 声明宏**的原理和使用，目的是为了能更好地看懂一些项目中 `macro_rules!` 的逻辑。

所谓宏编程，我理解本质上就是[元编程](https://en.wikipedia.org/wiki/Metaprogramming)（据说最早源自 LISP 的「Code is Data, Data is Code」的提法）。元编程可以让开发者将原生语言写的代码作为数据输入，经过自定义的逻辑，重新输出为新的代码并作为整体代码的一部分。这个过程一般在**编译时期**完成（对于编译型语言来说），所以让人觉得这是一种神奇的 “黑魔法”。

其他编程语言常见的元编程方式有：

- **Go 的 [ast](https://pkg.go.dev/go/ast) 包和 go generate 机制**：Go 没有显示提供元编程的相应机制，转而提供了一些相对不那么优雅的机制来实现类似于元编程的效果。比如 ast 包可以暴露 Go 程序的语法树，从而让开发者可在编译时期对源代码进行修改或者根据模版生成其他类型代码。
- **C++ 的 Template 编程**：据说 C++ 的 Template 编程是图灵完备的，可在编译时期完成很多让人瞠目结舌的逻辑。由于 C++ 的 Template 编程非常复杂且难以掌握，所以易用性非常差。
- **C 语言的宏**：这估计是大多数程序员对于宏的最初体验。个人觉得， C 语言中的宏本质上是发生在**预处理过程**的文本替换，是一种非常简单原始的元编程机制。而正是这种原始能力，导致 C 语言的宏结合编译器的各种扩展充满了各种奇技淫巧，可读性和可调试性都非常差，而且稍不小心就很容易写出错误的宏。

不同的编程语言对元编程的支持不太一样，有些可能有显式宏的概念（比如 C 和 Rust），有些可能只是提供一些工具包（比如 Go）让用户自行组装逻辑，但是其核心思想还是元编程的范畴。

由于 Rust 提供了更加智能的宏机制，所以 Rust 可以实现更高级的元编程。我们可以在编译时期将代码当成数据一样去改写，所以我们等于**变相地扩展了 Rust 标准语言编译器的能力**，从而去支持一些原本编译器还不支持的语言特性，比如 Rust 的异步机制在未以 `asyc/await` 等关键词引入语言标准时，社区就以 Rust 宏的形式提供对异步能力的支持。这其实是一种非常好的语言扩展手段。Rust 的语言开发者可以先用宏来扩展语言特性，验证某种特性是否可行，以期获得社区用户的反馈，从而再决定是否纳入到语言标准中。

## Rust 中宏的分类

目前 Rust 支持两种类型的宏：

1. **声明宏**（declarative macro）：可认为就是 `macro_rules!`，也是**本文叙述的重点**。
2. **过程宏**（procedural macro）：过程宏又可以继续分为**函数宏**、**属性宏**和**派生宏** 3 种，形式上不太一样，但是本质上都是将宏的编写变成了一个类似于编写函数的过程，**将输入的 TokenStream 处理转化为输出的 TokenStream。**

**Rust 的宏本质上就是一种[语法扩展](https://zjp-cn.github.io/tlborm/syntax-extensions/ast.html)**，从形式上看，Rust 的宏有以下 4 种形式：

1. `#[ $arg ]`：比如 `#[derive(Clone)]`，`#[no_mangle]` 等；
2. `#! [ $arg ]`：比如 `#![allow(dead_code)]` 等；
3. `$name ! $arg`：比如 `println!("Hi")`，`concat!("a", "b")` 等；
4. `$name ! $arg0 $arg1`：比如 `macro_rules! dummy { () => {}; }`，其实就只有 `macro_rules!`；

1 和 2 可认为是属性宏，即过程宏的一种，而 3 和 4 可理解为同一种类。采用**第 3 种形式**，Rust 还提供了不少内置宏，比如 `include!`、`file!`、`line!` 等。内置宏是**硬编码**在 rustc 中实现的。

## 快速入门体验

### 一个最简单的 `my_vec!`

按照大多数文章的做法，我们也来一步步实现一下我们的 [`vec!`](https://doc.rust-lang.org/stable/std/macro.vec.html)，此时我们将其命名为 `my_vec!`。整个项目的目录如下所示（使用 2018 版本）：'

```sh
.
├── Cargo.lock
├── Cargo.toml
├── examples
│   └── main.rs
├── src
│   ├── lib.rs
│   └── my_vec.rs
```

我们将在 `my_vec.rs` 中实现一个最简单的宏。**`#[macro_export]` 注解意味着宏将被引入作用域可见**，缺少了这个注解宏则不能被引入作用域。这个宏最终会转化成一句 `println!`，相当于我们在宏里调用其他宏：

```rust
#[macro_export]
macro_rules! my_vec {
    () => {
        println!("Hello, 'my_vec!'");
    };
}
```

然后在 `examples/main.rs` 中使用我们的 `my_vec!`。我们之所以用以下这 3 种形式，其实是想表达 `my_vec!` 之后可以跟 `[]`/`{}`/`()`。

```rust
fn main() {
    my_vec![];
    my_vec! {};
    my_vec!();
}
```

编译构建运行：

```fallback
$ cargo run --example main
Hello, 'my_vec!'
Hello, 'my_vec!'
Hello, 'my_vec!'
```

一切正常！看起来非常好。可是，我们调用 `my_vec!` 到底发生了什么？宏一个非常重要的概念就是**展开**，本质上就是在对应调用点**展开成真正的 Rust 代码**。Rust 的宏将在 Rust 程序的**语法分析之后**进行递归展开（如果宏里面调用其他宏，将会被进一步展开），最终的效果是生成合法的 Rust 代码。

为了能更方便地观察到宏展开的过程，我们可以使用一个 Cargo 插件 [cargo-expand](https://github.com/dtolnay/cargo-expand) 工具：

```sh
# 直接安装 cargo-expand 插件
$ cargo install cargo-expand
```

安装完成后，我们执行：

```sh
$ cargo expand --example main
```

我们可以看到如下输出（省去一些 `use` 语句）:

```rust
fn main() {
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["Hello, \'my_vec!\'\n"],
            &[],
        ));
    };
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["Hello, \'my_vec!\'\n"],
            &[],
        ));
    };
    {
        ::std::io::_print(::core::fmt::Arguments::new_v1(
            &["Hello, \'my_vec!\'\n"],
            &[],
        ));
    };
}
```

很明显，`my_vec!` 被展开成了一个 `{...}` 语句，而 `{...}` 中通过调用 `std::io::_print()` 实现了标准输出的打印功能。expand 不仅展开了 `my_vec!`，也把 `println!` 展开了。

cargo-expand 其实只是将对 `rustc` 的调用包装成了一个 Cargo 插件：假如我们有一个 Rust 文件，想对其进行宏展开，最原始简单的方式是：

```sh
$ rustc +nightly -Zunpretty=expanded use_macro.rs
```

虽然 Rust 还提供了其他一些调试宏的方式，但是展开宏是**最直观简单**的方式。

### `macro_rules!` 的基本结构

`macro_rules!` 最基本的结构如下所示：

```rust
macro_rules! $name {
    $rule0 ;
    $rule1 ;
    //...
    $ruleN ;
}
```

**至少得有一条规则**，且最后一条规则后面的分号可被省略。

而每一条 `rule` 其实就是**模式匹配**和**代码扩展生成**：

```sh
( $matcher ) => { $expansion };
```

当一个宏被调用时，将根据 `macro_rules!` 写的模式逐一进行匹配（其中 `$expansion` 最外层用 `()` 也可以），一旦匹配，将扩展生成 `{...}` 的代码。

回到上面的例子，我们的 `my_vec!` 只有一条匹配 rule，那就是：

```rust
() => { println!("Hello, 'my_vec!'"); };
```

`()` 表示为**匹配空输出**。因此，当我们使用 `my_vec![]` 时，由于是一个空输入，所以对应的宏调用将被展开成 print 操作。

了解到这里，我们其实就能基本知道 `macro_rules!` 的基本结构：**写一系列匹配规则来转化 Rust 代码**。

由于当 `vec![]` 为空输入时，则表示创建一个新的 vector，所以我们可将我们 `my_vec!` 改写为：

```rust
#[macro_export]
macro_rules! my_vec {
    () => {
        std::vec::Vec::new()
    };
}
```

稍微注意一点：**对 crate 的使用最好带上完整的路径**。因为宏将在调用之处被展开，所以我们**无法感知**当前调用环境是否已经有了相关的 use 语句。

### 捕获元变量

`$matcher` 可以基于某种语法类别匹配输入，并将结果**捕获到元变量**（metavariable ）中使用。比如，当我们想实现类似 `vec![0; 10]` 的功能时，如何写这个 `$matcher` ？

我们仔细观察调用参数： `0; 10` ，其中 `;` 左边是元素初始值 `0`，`;` 右边是个数 `10`。那么匹配似乎可以为：

```rust
($elem ; $n) => { ... }
```

但是这种描述是不精确的，我们还需要加上捕获方式，即捕获的是一个表达式：

```rust
($elem:expr ; $n:expr) => { ... }
```

所以此时整个 `$matcher` 含义为：匹配到以 `;` 分隔的两个**表达式**，`;` 左边的表达式的值将被捕获匹配到 `$elem`，`;` 右边的表达式的值将被捕获匹配到 `$n`。后续的 `$expansion` 中将可以直接使用 `$elem` 和 `$n`，如下所示：

```rust
( $elem: expr ; $n: expr ) => {
    std::vec::from_elem($elem, $n)
};
```

`expr` 可称为片段类型，更详细的分类可参考[文档](https://zjp-cn.github.io/tlborm/decl-macros/minutiae/fragment-specifiers.html)。

### 重复

标准的 `vec!` 一般是如下方式使用：

```rust
let v = vec![1, 2, 3];
```

如果我们用人脑模拟一下，`vec![1, 2, 3]` 应该被展开为：

```rust
let v = {
    let mut v = std::vec::Vec::new();
    v.push(1);
    v.push(2);
    v.push(3);
    v
};
```

这段语句如何用宏模式来表达呢 ？首先我们先看 `$matcher` 部分，即 `1, 2, 3`。像这种需要匹配一系列 token 的模式，我们需要**使用宏里的重复匹配模式**。比如要想匹配 `1,2,3`，可以写成：

```rust
( $( $elem:expr ),*) => { ... }
```

即 `$(...),*` 模式，而 `(...)` 则是和上节中变量捕获的方式是一样的，即 `$elem:expr`。`,` 表示为分隔符，`*` 表示匹配 0 或者多次。这样一来，`$( $elem:expr ),*` 表示为：**匹配 0 或者多次以逗号分隔的表达式，并将变量捕获到 `$elem` 中**。

其实，**重复捕获的一般形式为 `$ ( ... ) sep rep`**，这里：

- `(...)` 就是反复匹配的模式；

- `sep` 是可选的分隔标记，常见的有 `,` 和 `;`；

- ```
  rep
  ```

   

  是必须的重复操作符，可以为：

  - `?`：最多一次重复；
  - `*`：0 次或多次重复；
  - `+`：1 次或多次重复；

解决了 `$matcher`，这时候我们可以用类似的方式写 `$expansion`。重复匹配模式其实隐含提供了一种循环模式，可以生成多条符合某种模式的语句，比如我们可以用：

```rust
$( v.push($elem); )*
```

这句代码将会根据匹配到的 `$elem` 来生成 0 句或者多句 vector 的 push 语句。

因此完整的匹配逻辑可为：

```rust
// 匹配类似于 vec![1, 2, 3] 的输入
( $( $elem:expr ),* ) => {
    // 由于我们将生成多条语句，因此必须再用 {} 包起来
    {
        let mut v = std::vec::Vec::new();
        $( v.push($elem); )*
        v
    }
};
```

但是，如果末尾还有 `,`，比如 `my_vec![1,2,3,]` 这种情况，我们的匹配模式还能符合吗 ？

很遗憾，`$( $elem:expr ),*` 只能匹配以 `,` 分隔的表达式，无法匹配结尾还有一个 `,` 的场景。但其实，我们只要稍微基于这个例子再增加一个匹配模式即可：

```rust
// 匹配类似于 vec![1, 2, 3] 的输入
( $( $elem: expr ),* ) => {
    // 由于我们将生成多条语句，因此必须再用 {} 包起来
    {
        let mut v = std::vec::Vec::new();
        $( v.push($elem); )*
        v
    }
};
// 匹配类似于 vec![1, 2, 3, ] 的输入
( $( $elem: expr, )* ) => {
    // 由于我们将生成多条语句，因此必须再用 {} 包起来
    {
        let mut v = std::vec::Vec::new();
        $( v.push($elem); )*
        v
    }
};
```

我们将 `$( $elem:expr ),*` 修改为了 `$( $elem:expr, )*`，从而支持尾部带 `,`。可是看上面的例子，会感觉代码重复度比较高。由于宏可以**递归调用自己**，所以我们可以在最后一个匹配模式中自己再次调用 `my_vec!`，比如：

```rust
// 匹配类似于 vec![1, 2, 3] 的输入
( $( $elem: expr ),* ) => {
    // 由于我们将生成多条语句，因此必须再用 {} 包起来
    {
        let mut v = std::vec::Vec::new();
        $( v.push($elem); )*
        v
    }
};

// 匹配类似于 vec![1, 2, 3, ] 的输入
( $( $elem: expr, )* ) => {
    // 递归调用
    my_vec![ $( $elem ),* ]
};
```

### 完整的例子

让我们将上述的几个例子拼在一起，就可以得到完整的 `my_vec!`：

```rust
#[macro_export]
macro_rules! my_vec {
    // 匹配空输入，创建一个新的 vector
    () => {
        std::vec::Vec::new()
    };

    // 匹配类似于 vec![0; 10] 的输入
    ( $elem: expr ; $n: expr ) => {
        std::vec::from_elem($elem, $n)
    };

    // 匹配类似于 vec![1, 2, 3] 的输入
    ( $( $elem: expr ),* ) => {
        // 由于我们将生成多条语句，因此必须再用 {} 包起来
        {
            let mut v = std::vec::Vec::new();
            $( v.push($elem); )*
            v
        }
    };

    // 匹配类似于 vec![1, 2, 3, ] 的输入
    ( $( $elem: expr, )* ) => {
        // 递归调用
        my_vec![ $( $elem ),* ]
    };
}
```

此时我们可以像使用 `vec!` 一样地使用 `my_vec!`。

此时我们也许会好奇，真实的 `vec!` 是怎么实现的呢 ？我们可以从[代码](https://doc.rust-lang.org/stable/src/alloc/macros.rs.html#59-70)中一窥究竟：

```rust
macro_rules! vec {
    () => (
        $crate::vec::Vec::new()
    );
    ($elem:expr; $n:expr) => (
        $crate::vec::from_elem($elem, $n)
    );
    ($($x:expr),*) => (
        $crate::slice::into_vec(box [$($x),*])
    );
    ($($x:expr,)*) => (vec![$($x),*])
}
```

其中 `$crate` 是一个**特殊的元变量**，用来指代当前 crate。

通过对比实现，我们其实可以发现二者的主体逻辑几乎都是一致的。但对于 `vec![1,2,3]` 这种类型的匹配，`vec!` 采用了**更高效**的 `into_vec()` 方法，将一个 `[T]` 切片转化为相应的 vector，而不是简单的像 `my_vec!` 这样先创建 vector 后多次进行 push 动作。

## 什么时候使用声明宏

在大多数时候，我们可以将一些冗余的代码用声明宏的形式进行逻辑上的封装。从实现功能的角度来看，`macro_rules!` 与普通函数在某些方面是比较类似的。如果遇到既可以用普通函数又可以用 `macro_rules!`宏进行逻辑抽象的场景，个人觉得应该**优先**使用普通函数，因为这样**可读性和可维护性更强**。但是，由于 `macro_rules!` 可以接触到代码语法分析后的**标记树**，因此可以实现不少用普通函数无法实现的逻辑，一个典型的例子就是在 `?` 还未引入 Rust 标准中时，社区就提供了一个 [`try!`](https://doc.rust-lang.org/std/macro.try.html) 来实现类似的能力。让我们来看看 `try!` 想实现一个什么样的功能，以及为什么用普通函数无法实现。

在 Rust 的 API 设计中，我们大多会使用 `Result<T, E>` 来表示一个**函数返回值**：

> 如果返回值是 `Ok(T)`，则表示函数调用成功；如果返回值是 `Err(E)`，则表示调用失败，其中 `E` 大多数时候是实现了[ `std::error::Error` ](https://doc.rust-lang.org/std/error/trait.Error.html)trait 的对象。

在一个函数中，如果我们**连续调用**多个这样的 API，并且希望如果返回值为 `Ok(T)` 时**取出 `T`** 并继续调用下一个 API；如果某个 API 返回 `Err(E)`，则**立即返回错误**并结束当前函数的调用。比如：

```rust
use std::fs::File;
use std::io::{self, Read};

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut f = {
        match File::open("hello.txt") {
            Ok(file) => file,
            Err(err) => return Err(From::from(err)),
        };
    };

    let mut s = String::new();
    let num_read = match f.read_to_string(&mut s) {
        Ok(num) => num,
        Err(err) => return Err(From::from(err)),
    };

    println!("read {} bytes", num_read);

    Ok(())
}
```

很明显，这里的错误处理存在一个 pattern：

```rust
match Result {
    Ok(val) => val,
    Err(err) => return Err(err)
}
```

当我们想用**普通函数**（称之为 `try()` ）去实现这个 pattern 的时候，会发现竟然实现不了：

- 当函数输入是 `Ok(T)` 时，返回 `T`；当函数输入是 `Err(err)`，返回 `Err(err)`，但这二者却不是同一个类型，Rust 此时没法正常地推断返回值的具体类型；
- `try()` 的 `return` 仅结束了对 `try()` 的调用，而我们是希望从调用 `try()` 的那个父亲函数中直接返回错误，这在普通函数中是无法办到的；

这时候就是 `macro_rules!` 的用武之地了，让我们写一个 `my_try!`：

```rust
#[macro_export]
macro_rules! my_try {
    ($result:expr) => {
        match $result {
            Ok(v) => v,
            Err(e) => {
                return std::result::Result::Err(std::convert::From::from(e));
            }
        }
    };
}
```

利用 `my_try!`，则上面的代码就可变为：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut f = my_try!(File::open("hello.txt"));

    let mut s = String::new();
    let num_read = my_try!(f.read_to_string(&mut s));

    println!("read {} bytes", num_read);

    Ok(())
}
```

是不是感觉顿时**清爽**了很多！这便是宏的魅力！社区对 `try!` 的尝试获得了成功，于是就添加了 `?` 操作符来表达与 `try!` 类似的语义，即：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut f = File::open("hello.txt")?;

    let mut s = String::new();
    let num_read = f.read_to_string(&mut s)?;

    println!("read {} bytes", num_read);

    Ok(())
}
```

这里不禁又要 “嘲笑” 一下 Go 的错误处理。由于 Go 只提供了基于返回值的 `error`，既没有类似于 `Result<T, E>` 的 sum type，也没有类似于 `?` 的错误处理简化写法，所以类似的代码在 Go 里必须用一堆 `if err != nil {...}` 来实现，比如：

```go
func main() {
	file, err := os.OpenFile("hello.txt", os.O_RDONLY, 0644)
	if err != nil {
		panic(err)
	}

	buf := make([]byte, 1024)
	numRead, err := file.Read(buf)
	if err != nil {
		panic(err)
	}

	fmt.Printf("read %d bytes\n", numRead)
}
```

一旦函数相互间调用复杂度上升，则 Go 代码就会充斥着非常多的**啰嗦**的错误返回值判断，所以一直有人诟病 Go 代码中有一半是这类返回值判断语句。Go 薄弱的元编程能力也没法让开发者很好地包装出一个类似于 Rust 的 `try!` 宏。这其实也可以从侧面看出两门语言设计上的哲学差异：Go 比较追求简单务实，而 Rust 更追求高效和零成本抽象，但也因此带来了比 Go 更为复杂的语言特性。尽管 Go 的错误处理看起来比较啰嗦，但是几乎有效地将 “所有” 的错误处理都做成了统一的模式，从而更便于开发者掌握和使用。

## 声明宏的卫生性

宏的**卫生性**（hygiene）是一个比较奇怪的说法，总让人摸不着头脑（其实 “宏” 这个词同样是一个奇怪的翻译）。有些书（比如参考文档 [2]）则翻译为 “**自净**”。本文沿用参考文档 [1] 的中文翻译，因为这是最直译的做法。

所谓宏的卫生性，其实[说](https://zjp-cn.github.io/tlborm/syntax-extensions/hygiene.html)的就是**宏在上下文工作不影响或不受周围环境的影响**。或者换句话来说，就是宏的调用是没有 side effect。对于 `macro_rules!`，它是[**部分卫生的**](https://zjp-cn.github.io/tlborm/decl-macros/minutiae/hygiene.html)（partially hygienic）。我们目前阶段可以不用太关注 `macro_rules!` 在哪些场景是 “不卫生” 的，而是了解一下 `macro_rules!` 是如何在大多数场景做到 “卫生” 的。

我们来看一看这样一个简单的例子：

```rust
#[macro_export]
macro_rules! make_local {
    () => {
        let local = 0;
    };
}

fn main() {
    let local = 42;
    make_local!();
    assert_eq!(local, 42);
}
```

理论上，`make_local!` 是一个意图有副作用的宏，但实际上，`main()` 中的 `local` 在 `make_local!` 调用前后依然为 42。这时候让我们用 `cargo expand` 展开宏来看一看：

```rust
fn main() {
    let local = 42;
    let local = 0;
    {
        match (&local, &42) {
            // 省略 assert_eq! 的展开
        }
    };
}
```

展开后的代码竟然有两个**同名的 `local`**，而且 `assert_eq!` 比较智能地选择了第一个 `local`。之所以能做到这一点，Rust 其实是为每个标识符都赋予了一个**看不见的句法上下文**（syntax context），或者按照参考文档 [2] 的一种更形象的说法，Rust 会为其“**分别染上不同的颜色**”。在 Rust 看来，`macro_rules` 中的 `local` 和 `main()` 里的 `local` 分别有着**不同的颜色**，所以不会将其混淆。

## 总结

虽然 `macro_rules!` 有相对比较多不那么直接的规则，但整体上结构上还是比较简单的，而且实际项目中，`macro_rules!` 的代码大多比较简短。当我们遇到看不太懂的 `macro_rules!`，一个比较好的方式用工具将其**展开**，一展开基本了无秘密，然后我们再就着手册（比如[《Rust 宏小册》](https://zjp-cn.github.io/tlborm/introduction.html) 就是一本翻译和原作都非常不错的书）和文档读懂**匹配模式**和**转换规则**，一般都可以把 `macro_rules!` 给整明白。

**祝大家玩得愉快** ！

## 参考文档

1. [Rust 宏小册](https://zjp-cn.github.io/tlborm/introduction.html)
2. [Rust 程序设计](https://item.jd.com/12971660.html)