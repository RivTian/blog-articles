#Rust

## 背景

我相信大多数人在学习 Rust 异步编程时都会被 [Future trait](https://doc.rust-lang.org/std/future/trait.Future.html) 中的 `Pin` 指针感到困惑：

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

特别是搜索了一圈文档之后，更会对这个 `Pin` 一头雾水，彷佛自己也被 Pin 住了一样。本文将尝试逐步一点点去解释 `Pin` 的来龙去脉，希望能够提供更容易懂的知识结构。

## 快速体验 Pin

我们可以在只了解以下几个**基本事实**的基础上快速体验 `Pin`，从而从直觉上了解它的作用。

1. `Pin<P>` 是一个泛型数据结构，其中**包裹的 `P` 必须是指针**，或者更准确地来说：**必须是实现了 [Deref trait](https://doc.rust-lang.org/std/ops/trait.Deref.html) 的对象**。我们可以将 `Pin<P>` 理解为 **`Pin<P<T>>`**，其中 `T` 是最原始的数据类型；
2. `Pin<P<T>>` 是指针，对其解引用将获取 `*P` 中的类型，即 `T`，常见的 `P<T>` 可以是 `Box<T>`、`&T`、`&mut T` 等；
3. 给定 `Pin<P<T>>` 类型的数据，**只要 `T` 不满足 Unpin trait**，则 Safe Rust（即**不使用 `unsafe{}` 块**）下无法获得 `&mut T` 和 `T`。换言之，要想 `Pin` 有其效果，则 `T` 必须不满足 Unpin trait；

在理解了以上几条，我们来简单快速尝试一下 `Pin`：

```rust
use std::pin::Pin;

#[derive(Debug)]
struct Foo {
    x: i32,
    y: i32,
}

impl Foo {
    fn new() -> Self {
        Foo { x: 0, y: 1 }
    }
}

fn main() {
    // 先创建数据在堆上的 Box<Foo> 指针，然后在基于 Box<Foo> 创建 Pin 指针
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    
    // 这里必须使用 rerference，否则 borrow checker 将会报错
    // *pin_foo 相当于获取 Foo
    let foo_ref: &Foo = &*pin_foo;
    
    println!("{:?}", foo_ref);
}
```

如上例子所示：

- 我们先创建数据在堆上的 `Box<Foo>` 指针，然后再基于 `Box<Foo>` 创建 Pin 指针，上面这两句代码也可以简化为用 `Box::pin()` 函数：

  ```rust
  let pin_foo: Pin<Box<Foo>> = Box::pin(Foo::new())
  ```

  二者是**等价**的。

- `&*pin_foo` 相当于获取堆上 `Foo` 数据的不可变引用。此处必须使用引用。如果直接使用 `*pin_foo`，相当于我们通过解引用获取所有权，这在 borrow checker 中是不可行的，将触发 [E0507](https://doc.rust-lang.org/error-index.html) 错误；

- `println!` 接收的就是 `&T` 类型参数，所以我们最终可以如预期获得相应的打印：

  ```sh
  Foo { x: 0, y: 1 }
  ```

从逻辑上来看，这段代码在实际堆栈上如下所示：

![](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/bwzqKU.jpg)



我们此时再想想事实 1，即 `Pin<P>` 中 `P` 必须是指针，比如上面例子中的 `Box<Foo>`。如果我随便传入一个非指针值呢？比如：

```rust
let p = Pin::new(2);
```

聪明的 Rust 编译器立即就发现问题：

```sh
let p = Pin::new(2);
                 ^ the trait `Deref` is not implemented for `{integer}`
```

整型的 `2` 并没有实现 Deref trait，不被认为是一个指针。

此时让我们基于这个简单的例子体验一下第 3 个基本事实，即**只要 `T` 不满足 Unpin trait，则 `Pin<P<T>>`无法在 Safe Rust 下获得 `&mut T`** 。

[Unpin trait](https://doc.rust-lang.org/std/marker/trait.Unpin.html) 是一个特殊的 marker trait（可以理解为类似于 Send 或者 Sync trait），它不需要实现什么具体的方法，仅仅只是一个标记，**Rust 默认给所有类型都自动实现了 Unpin trait**，也就是说，对于原生类型或者自定义类型，编译器都自动为其实现了 Unpin trait，也就是说，此时的 `Pin<P<T>>` 可以获取到 `&mut T`，让我们试一下：

```rust
fn main() {
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let mut pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    let _foo_ref: &mut Foo = &mut *pin_foo;
}
```

编译没有问题（可以使用 `cargo check`），这说明自定义的 `Foo{}` 确实不满足基本事实 3。

我们如何显式地不让编译器为我们实现 Unpin trait ? 最简单且通用的方式就是使用 [`PhantomPinned`](https://doc.rust-lang.org/nightly/std/marker/struct.PhantomPinned.html)。`PhantomPinned` 是一个空的结构体，是一个 marker type。只要类型中含有这个 marker 成员，则表示对应类型**默认不要实现 Unpin trait**，此时我们可以将代码改成：

```rust
use std::marker::PhantomPinned;
use std::pin::Pin;

#[derive(Debug)]
struct Foo {
    x: i32,
    y: i32,
    _marker: PhantomPinned,
}

impl Foo {
    fn new() -> Self {
        Foo {
            x: 0,
            y: 1,
            _marker: PhantomPinned,
        }
    }
}

fn main() {
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let mut pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    let _foo_ref: &mut Foo = &mut *pin_foo;
}
```

发现此时无法编译：

```sh
let mut pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
                                 ^^^^^^^^ within `Foo`, the trait `Unpin` is not implemented for `PhantomPinned`
```

为什么会出现这个错误 ？其实还得看 `Pin::new()` 的[签名定义](https://doc.rust-lang.org/src/core/pin.rs.html#475)：

```rust
impl<P: Deref<Target: Unpin>> Pin<P> {
    // ...
    pub const fn new(pointer: P) -> Pin<P> {
        // ...
    }
    // ...
}
```

上面的泛型约束 `P: Deref<Target: Unpin>` 说明了 `Pin::new()` 必须是：

- `Pin<P>` 中 `P` 需满足 Deref trait；
- `Pin<P>` 中 `P` 所包裹的 `Target` 需满足 Unpin trait；

很显然，当 `Foo{}` 被显式不实现 Unpin trait 时，已经无法使用 `Pin::new()`。

这时候，要为不实现 Unpin trait 的类型构造 `Pin`，必须使用 **unsafe** 的 [`Pin::new_unchecked()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.new_unchecked)，即：

```rust
fn main() {
    let box_foo: Box<Foo> = Box::new(Foo::new());
    // let mut pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    let pin_foo: Pin<Box<Foo>> = unsafe { Pin::new_unchecked(box_foo) };
    let _foo_ref: &mut Foo = &mut *pin_foo;
}
```

此时我们再次编译，会发现有如下错误：

```rust
let _foo_ref: &mut Foo = &mut *pin_foo;
                         ^^^^^^^^^^^^^ cannot borrow as mutable
help: trait `DerefMut` is required to modify through a dereference, but it is not implemented for `Pin<Box<Foo>>`
```

我们暂且不要去理解这个错误，可以认为是**编译器阻止**我们创建 `&mut Foo` 类型变量，即基本事实 3 所说的内容。当我们把 `mut` 关键字去掉时，如下：

```rust
fn main() {
    let box_foo: Box<Foo> = Box::new(Foo::new());
    // let mut pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    let pin_foo: Pin<Box<Foo>> = unsafe { Pin::new_unchecked(box_foo) };
    let _foo_ref: &Foo = &*pin_foo;
}
```

则编译通过，说明 `&Foo` 类型的变量还是可以正常创建。

至此，**我们深呼一口气**，对 Pin 的初体验暂且告一段落。

## Pin 想解决的问题

### 核心能力

我们可以从 Pin 模块的[文档](https://doc.rust-lang.org/std/pin/)上提炼出这么几句话：

> …
>
> …`Pin<P>` does not let clients actually obtain a `Box<T>` or `&mut T` to pinned data…
>
> …`Pin<P>` prevents certain *values* (pointed to by pointers wrapped in `Pin<P>`) from being moved by making it impossible to call methods that require `&mut T` on them (like [`mem::swap`](https://doc.rust-lang.org/std/mem/fn.swap.html)).
>
> …

其实，我们的体验篇的基本事实 3 就很好的概括了 Pin 的**核心能力**：**给定 `Pin<P<T>>` 类型的数据，\**只要 `T` 不满足 Unpin trait\**，则 Safe Rust 下无法获得 `&mut T` 和 `T`**，从而让 `T` **不能被 move**。

这样一来，只要用 `Pin<P<T>>` 类型的数据，Safe Rust 下也无法使用依赖于 `&mut T` 的 API（**编译器会阻止**）。

为什么 Rust 要引入 Pin 机制去阻止用户获取 `&mut T` 和 `T`？很大原因是为了解决**自引用结构体**带来的不安全问题。我们接下来就来介绍什么是自引用数据结构及其存在的问题。

### 自引用结构体

自引用结构体（self-referential struct）非常容易理解：**就是结构体成员引用（持有指针）了当前结构体的其他成员**。举个例子，假如我们有一个数据架构 `SelfReferenceFoo`（**注意**：结构体内使用引用需要显式用上生命期注解）：

```rust
struct SelfReferenceFoo<'a> {
    data: String,
    data_ref: &'a String,
}
```

其中：`data` 是一个 `String` 类型的变量，而 `data_ref` 是一个初始化时指向其成员 `data` 的一个引用变量。让我们来看看是否可以成功初始化：

```rust
struct SelfReferenceFoo<'a> {
    data: String,
    data_ref: &'a String,
}

fn main() {
    let s = String::from("hello");
    let _foo = SelfReferenceFoo {
        data: s,
        data_ref: &s,
    };
}
```

编译时发现了错误：

```mysql
let s = String::from("hello");
    - move occurs because `s` has type `String`, which does not implement the `Copy` trait
let _foo = SelfReferenceFoo {
    data: s,
          - value moved here
    data_ref: &s,
              ^^ value borrowed here after move
```

很明显，`s` 的所有权已经 move 给了 `foo.data`，则 `s` 已经失去了对数据的所有权，所以 `&s` 是一个无效的引用，不能赋给 `data_ref`。

那我们是不是可以绕一下 ？由于 Rust 的引用初始化必须被赋值，那么我们是不是可以：先初始化一个 `mut SelfReferenceFoo`，`data_ref` 成员先暂时用一个临时的 `String` 引用变量代替，待 `mut SelfReferenceFoo` 变量创建完了再将其修改成真实的 `data` 引用。比如下面这种做法：

```rust
fn main() {
    let data = String::from("hello");
    let tmp_data_ref = String::from("");

    // 创建一个 mut SelfReferenceFoo 变量，data_ref 用一个临时的 tmp_data_ref 变量替代
    let mut foo = SelfReferenceFoo {
        data,
        data_ref: &tmp_data_ref,
    };

    // 将 data_ref 修改成真实的 data 引用
    foo.data_ref = &foo.data;

    // 分别打印 foo.data 的内存地址和 foo.data_ref 变量的值
    println!(
        "address of data: {:p}, varible of data_ref: {:p}",
        &foo.data, foo.data_ref
    );
}
```

**编译运行成功**！

```sh
address of data: 0x7ffee25a3238, varible of data_ref: 0x7ffee25a3238
```

而且最后输出的内存地址也表明 `foo.data_ref` 确实指向了 `foo.data` 所在的地址。此时逻辑上内存 layout 如下所示：

![](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/rIl9Ib.jpg)



由于 `String` 类型的字符串数据是存放在堆上，所以 `data` 内部的数据结构将指向堆上的一个地址。

看起来 Rust 编译器并没有阻止我们使用自引用结构体嘛。但是，如果我们想将 `foo.data` 修改成其他值：

```rust
let new_data = String::from("world");
foo.data = new_data;
```

此时编译器将会报错：

```rust
foo.data_ref = &foo.data;
               --------- borrow of `foo.data` occurs here
...
    foo.data = new_data;
    ^^^^^^^^
    |
    assignment to borrowed `foo.data` occurs here
    borrow later used here
```

编译器提示我们 `foo.data` 已经被 borrowed，此时无法修改 `foo.data`，否则其先前的借出值 `foo.data_ref` 将可能会成一个悬垂指针。虽然我们持有一个 mutable 的 `SelfReferenceFoo`，**但却无法修改 `data` 字段**（`data_ref` 可以被修改）。这在实际使用中是没有价值的。

上面几个简单的尝试告诉我们，对于自引用结构体，其初始化和使用都受到了 Rust borrow checker 的层层限制，我们几乎没法以符合直觉的方式来安全使用自引用结构体。

### 使用自引用结构体存在的问题

为了进一步绕开 Rust borrow checker 的检查，从而更好地说明自引用结构体存在的问题，我们不得不使用 Rust 中的[裸指针](https://kaisery.github.io/trpl-zh-cn/ch19-01-unsafe-rust.html#解引用裸指针)。我们将 `SelfReferenceFoo` 修改为：

```rust
struct SelfReferenceFoo {
    data: String,
    data_ref: *const String,
}
```

将 `data_ref` 改成对 `String` 的裸指针 `*const String`。裸指针的初始化可以用 `std::ptr::null()` 将其设置成空指针，比如我们为 `SelfReferenceFoo` 构造了以下两个函数：

```rust
impl SelfReferenceFoo {
    fn new(data: String) -> Self {
        SelfReferenceFoo {
            data,
            data_ref: std::ptr::null(),
        }
    }

    // 设置 data_ref，将其指向 data
    fn set_data_ref(&mut self) {
        self.data_ref = &self.data as *const String;
    }
}
```

参考之前的实验，我们可以创建一个变量并输出 `data` 和 `data_ref` 的内存地址：

```rust
fn main() {
    let mut foo = SelfReferenceFoo::new(String::from("hello"));
    foo.set_data_ref();

    println!(
        "address of data: {:p}, varible of data_ref: {:p}",
        &foo.data, foo.data_ref
    );
}
```

这段代码与之前的实验的输出是类似的。当然，也可以对 `foo.data` 重新赋值（用裸指针等于绕开了 borrow checker）：

```rust
fn main() {
    let mut foo = SelfReferenceFoo::new(String::from("hello"));
    foo.set_data_ref();

    foo.data = String::from("bar");
    foo.set_data_ref();
}
```

我们可以在 Safe Rust 里安全地创建一个裸指针，**但是解引用裸指针必须在 `unsafe{}` 中**。比如我们需要用如下方式才能获得对 `data` 的引用：

```rust
let r: &String = unsafe { &*foo.data_ref };
println!("{:?}", r);
```

其中 `*foo.data_ref` 解引用裸指针获得 `String` 变量，再用 `&` 获得 `&String` 类型的引用。

综上，我们可以增加以下两个方法来分别打印 `data` 和 `data_ref` 的数据和地址：

```rust
impl SelfReferenceFoo {
    fn print_data(&self) {
        println!("data: {:?}, memory address: {:p}", self.data, &self.data);
    }

    fn print_data_ref(&self) {
        println!(
            "data that referenced: {:?}, memory address: {:p}",
            unsafe { &*self.data_ref },
            self.data_ref
        );
    }
}
```

使用自引用结构体最大的问题就是：**当对应结构体变量发生 move 动作时，结构体变量内的自引用指针将处于一种不安全的状态**。

比如下面这段简单的代码：

```rust
fn main() {
    let mut foo = SelfReferenceFoo::new(String::from("hello"));
    foo.set_data_ref();

    println!("[before move]");
    foo.print_data();
    foo.print_data_ref();

    // 发生移动，foo 将被回收，所指向的内存区域有可能会被写入其他值
    let new_foo = foo;

    println!("[after move]");
    new_foo.print_data();
    new_foo.print_data_ref();
}
```

代码将打印出：

```fallback
[before move]
data: "hello", memory address: 0x7ffee62ec230
data that referenced: "hello", memory address: 0x7ffee62ec230
[after move]
data: "hello", memory address: 0x7ffee62ec298
data that referenced: "hello", memory address: 0x7ffee62ec230
```

我们很明显可以看到：

- `new_foo` 中 `data` 字段的内存地址已经发生了改变，从原来的 `0x7ffee62ec230` 改变到 `0x7ffee62ec298`；
- `new_foo` 中的 `data_ref` 依然指向旧的 `foo.data`，此时 `new_foo` 中的 `data` 和 `data_ref` 将出现不一致。因为旧的 `foo` 的数据还未被回收覆盖，所以此时 `println()` 操作还可以正常打印；

在 Rust 中，move 动作从语义上是所有权发生了改变，但是底层实现是执行了变量的 **shallow copy**，同时让前一个变量不可用（所有权转移到新变量上），上述代码执行完 move 之后的内存 layout 如下图所示：

![](https://cdn.jsdelivr.net/gh/rivtian/Blogimg@main/uPic/T8Z8lV.jpg)



我们要特别注意上图中的**红色线**：`data_ref` 指向了一个即将被内存回收的区域，因此处于一种**不安全**的状态。将 `foo` move 到 `new_foo`，其实也可以理解成是一个 memcpy 的过程，比如[文档](https://doc.rust-lang.org/std/pin/)中提及调用 [`std::mem::swap()` ](https://doc.rust-lang.org/std/mem/fn.swap.html)也是可让自引用结构体处于不安全状态的一种典型的 API。

### 用 Pin 来解决问题

那么我们如何利用 `Pin` 来解决问题 ？根据我们之前的经验，我们得先显式让 `SelfReferenceFoo` 不实现 Unpin trait，即增加 `PhantomPinned` 类型的 `_marker`：

```rust
struct SelfReferenceFoo {
    data: String,
    data_ref: *const String,
    _marker: PhantomPinned,
}
```

然后我们将 `new()` 改为：

```rust
impl SelfReferenceFoo {
    fn new(data: String) -> Self {
        SelfReferenceFoo {
            data,
            data_ref: std::ptr::null(),
            _marker: PhantomPinned,
        }
    }
}
```

我们使用 `Pin`，是想将 `mut &SelfReferenceFoo` 包裹在 `Pin` 中，即**最终用户只能使用 `Pin<mut &SelfReferenceFoo>`**。结合我们之前的经验，我们可以这样创建 `Pin<mut &SelfReferenceFoo>` 类型变量：

```rust
fn main() {
    let mut original_foo: SelfReferenceFoo = SelfReferenceFoo::new(String::from("hello"));
    let foo: Pin<&mut SelfReferenceFoo> = unsafe { Pin::new_unchecked(&mut original_foo) };
    
    // do stuff with foo...
}
```

由于用户只能使用 `Pin<mut &SelfReferenceFoo>` ，我们同样要调整一下 `set_data_ref()` 的逻辑：

```rust
impl SelfReferenceFoo {
    fn set_data_ref(pinned_data: Pin<&mut Self>) {
        let data_ref: *const String = &pinned_data.data as *const String;

        // 必须使用 unsafe 的 Pin::get_unchecked_mut() 才能获得 Pin<P<T>> 中的 &mut T
        let mut_foo_ref: &mut SelfReferenceFoo = unsafe { pinned_data.get_unchecked_mut() };

        mut_foo_ref.data_ref = data_ref;
    }
}
```

一共三句代码，我们逐条分析：

1. 由于不满足 Unpin trait 的 Pin 也实现了 Deref trait（**注意**：该状态下的 Pin 没有实现 DerefMut trait，所以无法获取 `&mut T`），所以我们可以直接获得 `data` 指针，其实也可以写成：

   ```rust
   let data_ref: *const String = &pinned_data.as_ref().get_ref().data as *const String;
   ```

   - `as_ref()` 是获得 `Pin<&SelfReferenceFoo>`，而 `get_ref()` 是获得 `&SelfReferenceFoo`。

2. 在 `Pin<P<T>>` 中获取 `&mut T` 方式（T 不满足 Unpin trait ）就是**调用 unsafe 的 `get_unchecked_mut()`**。这句代码是我们这个例子中唯一使用到 `&mut T` 的地方，目的是为了修改 `data_ref` 的值，将其指向 `data`。

同理，我们将 `print_data()` 和 `print_data_ref()` 修改为：

```rust
impl SelfReferenceFoo {    
	fn print_data(pinned_data: Pin<&Self>) {
        println!(
            "data: {:?}, memory address: {:p}",
            pinned_data.data, &pinned_data.data
        );
    }

    fn print_data_ref(pinned_data: Pin<&Self>) {
        println!(
            "data that referenced: {:?}, memory address: {:p}",
            unsafe { &*pinned_data.data_ref },
            pinned_data.data_ref
        );
    }
}
```

此时我们可以这样使用新的 `SelfReferenceFoo`：

```rust
fn main() {
    let mut original_foo: SelfReferenceFoo = SelfReferenceFoo::new(String::from("hello"));
    let mut foo: Pin<&mut SelfReferenceFoo> = unsafe { Pin::new_unchecked(&mut original_foo) };
    
    // as_mut() 将返回 Pin<&mut SelfReferenceFoo>
    SelfReferenceFoo::set_data_ref(foo.as_mut());

    // as_ref() 将返回 Pin<&SelfReferenceFoo>
    SelfReferenceFoo::print_data(foo.as_ref());
    SelfReferenceFoo::print_data_ref(foo.as_ref());
}
```

那么我们还可以对 `SelfReferenceFoo` 执行 move 动作吗 ？回答是：在 Safe Rust 中我们没有办法获得 `Pin<&mut SelfReferenceFoo>` 中的 `SelfReferenceFoo` 变量的所有权，所以我们无法执行 move。而且在 Safe Rust 中我们也无法获得 `&mut SelfReferenceFoo`，从而也无法调用 `std::men:swap()` 。所以顾名思义，`SelfReferenceFoo` 变量的区域就相当于**被 Pin 住**一样，只能读，**无法 move 和获得可变引用**。而如果我们 move `Pin<&mut SelfReferenceFoo>`，并不影响 `SelfReferenceFoo` 变量。

综上，Pin 机制就是提供了一种方式，包裹住自引用结构体的指针，从而从编译器层面避免自引用结构体被 move。如果非得 move，则用户自己使用 unsafe API，**自己负责其安全性**。

## Future 中存在的自引用结构体

让我们来重新回过头来看 Future trait 的定义：

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

现在是不是很好理解呢 ？`poll()` 中的 `Pin<&mut Self>` 表示：实现 Future trait 的类型**有可能是一个自引用结构体**，为了安全性，我们需要将其用 `Pin` 包裹起来。

为什么实现 Future trait 的类型有可能是一个自引用结构体 ？那这必须从 Rust 的异步机制说起。不同于 Go 的基于 stack 的 goroutine 机制，Rust 利用 `async {}` 关键字将可被异步化的逻辑封装成一个 Future，而多个 Future 交给各种不同的 Executor 采取不同的调度机制来执行（比如 [future-rs](https://github.com/rust-lang/futures-rs)、[tokio](https://tokio.rs/)、[async-std](https://github.com/async-rs/async-std) 等等）。在编译器的背后，每一个 async 部分的逻辑都将被编译成一个个实现了 Future trait 的结构体，而且结构体内部维护着一个状态机，状态机内部的每一个状态则是 `.await` 点。这样的结构体也可以理解为是一个 coroutine 或者 [task](https://tokio.rs/tokio/tutorial/spawning#tasks)。

如果一个 Future 内部有多个 `.await`，而遇到 `.await` 则说明当前操作需要等一等（很多时候是 IO 操作），可以让当前 CPU 去干点其他事。此时，`.await` 点相当于**让出**（Yield）CPU，让 Executor 去执行另外的 Future。为了让这个状态机可以恢复 `.await` 前后的状态，编译器需要将相应的变量保存起来，形成一个结构体，且这个结构体需要实现 Future trait。在这个过程中，由于这个结构体不仅要保存上一个状态中的一些变量，而且有可能当前状态的 Future 中依赖于上一个状态生成的变量，这就将产生自引用结构体。Rust 这种实现 coroutine 的方式称之为 [stackless coroutines](https://rust-lang.github.io/rfcs/2033-experimental-coroutines.html#state-machines-as-stackless-coroutines)，区别于 Go 的 stackful coroutines（即 goroutine）。

因此，为了保证这个 coroutine 生成过程的安全性，我们用 `Pin` 将实现 Future trait 的潜在自引用结构体给钉住。

## Pin 的核心 API

### 几个关键的 API

**备注**：!Unpin 表示不实现 Unpin trait，可参考最后一节的内容。

参考[文档](https://zyy.rs/post/rust-pin/(https://folyd.com/blog/rust-pin-advanced/))的分类，我们可以重点关注 Pin 模块中的这几个 API（**下面的 API 均在 `std::pin::Pin` 这个模块中**）：

- **构造 `Pin<P<T>>`**

  - 如果 T 是 Unpin，可直接使用安全的 [`new()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.new)，比如：

    ```rust
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let pin_foo: Pin<Box<Foo>> = Pin::new(box_foo);
    ```

  - 如果 T 是 !Unpin，需使用**不安全**的 [`new_unchecked()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.new_unchecked)，比如：

    ```rust
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let pin_foo: Pin<Box<Foo>> = unsafe { Pin::new_unchecked(box_foo) };
    ```

  或者是将变量用 [`Box::pin()`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.pin) Pin 在堆上，此时无需区分使用哪一个 API。

- **获取 `&T`**

  - [`get_ref()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.get_ref)：获取 `&T`，**适用于 T 是 Unpin 和 !Unpin 安全 API**；
  - [`get_muf()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.get_mut)：获取 `&mut T`，**适用于 T 是 Unpin 的安全 API**。对于 T 是 Unpin 的场景，获取 `&mut T` 是一个安全操作；
  - [`get_unchecked_mut()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.get_unchecked_mut)：获取 `&mut T`，**适用于 T 是 !Unpin 的不安全 API**；

- **获取 `P`**

  - [`into_inner()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.into_inner)：输入 `Pin<P>` 变量，获得 `P`，此时将消耗 `Pin<P>`。**适用于 T 是 Unpin 的安全 API**；
  - [`into_inner_unchecked()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.into_inner_unchecked)：输入 `Pin<P>` 变量，获得 `P`，此时将消耗 `Pin<P>`。**适用于 T 是 !Unpin 的不安全 API**；

- **转换 `Pin` 类型**

  - [`as_ref()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.as_ref)：将 `Pin<P<T>>` 转换为 `Pin<&T>`，**适用于 T 是 Unpin 和 !Unpin 的安全 API**；
  - [`as_mut()`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.as_mut)：将 `Pin<P<T>>` 转换为 `Pin<&mut T>`，**适用于 T 是 Unpin 和 !Unpin 的安全 API**；

### 另一种阻止编译器不实现 Unpin 的方法

除了显式在结构体内加入 `PhantomPinned` marker 变量阻止编译器实现 Unpin trait，Rust 还提供了 [negative_impls](https://doc.rust-lang.org/beta/unstable-book/language-features/negative-impls.html) 方式来直接声明不让对于结构体实现 Unpin trait。如下所示：

```rust
// 必须使用对应的编译器 feature
#![feature(negative_impls)]
use std::pin::Pin;

#[derive(Debug)]
struct Foo {
    x: i32,
    y: i32,
}

// 显式表示 Foo 不要实现 Unpin（即 !Unpin）
impl !Unpin for Foo {}

impl Foo {
    fn new() -> Self {
        Foo { x: 0, y: 1 }
    }
}

fn main() {
    let box_foo: Box<Foo> = Box::new(Foo::new());
    let pin_foo: Pin<Box<Foo>> = unsafe { Pin::new_unchecked(box_foo) };
    let _foo_ref: &Foo = &*pin_foo;
}
```

其中：

- `#![feature(negative_impls)]`：告诉编译器需要 negative_impls 这个 feature；
- `impl !Unpin for Foo {}`：`!Unpin` 表示不实现 Unpin trait；

由于 negative_impls 特性目前还没有加入在主干上，我们需要用到 nightly 特性，上面的代码可以用如下方式进行编译：

```sh
$ rustup install nightly
$ cargo +nightly build
```

## 参考文档

1. [Rust 的 Pin 与 Unpin](https://folyd.com/blog/rust-pin-unpin/)
2. [Rust 进阶](https://folyd.com/blog/rust-pin-advanced/)
3. [Async/Await Pinning](https://os.phil-opp.com/async-await/#pinning)
4. [Module std::pin](https://doc.rust-lang.org/std/pin/)