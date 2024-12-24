#Rust 

Ownership, Borrowing 与 Lifetime 共同成就了 rust 中的内存安全，也是 rust 语言中最精髓的创造，我们就来学习学习它们究竟是什么，为什么要引入这些概念。

## 权力与风险共生

权力与风险往往是一同出现。如果你被授予了制作核弹的权力，那么在你制造它时其实是面临着诸多的风险。

早期的编程语言如 C/C++ 赋予了程序员极高的权力，它们能自由地操作计算机的内存（虚拟内存），程序员们因此可以尽情地挥洒着自己的创造力来达到更强大的性能。

然而这份权力也带来了许多风险，例如一个常见的问题是内存泄漏，即忘记 `free` 自己 `malloc` 出来的内存，程序不断运行最终导致内存耗尽，C++ 通过引入析构函数防止程序员忘记释放内存。但另一个常见问题依旧无法避免，即访问已经释放的内存，或者尝试释放已经释放的内存。

人们认识到，内存管理存在的风险已经远远大于它所赋予的权力带来的好处，Java 语言的便通过引入 GC （垃圾回收器）替程序员管理内存。程序员不再需要关心什么时候释放内存，因为 JVM 会自动处理；也不必害怕会访问已经释放的内存，因为只要内存还有变量使用，JVM 就不会去释放它。而对应的，GC 剥夺了程序员自由操作内存的权力，付出的代价便是额外的性能开销。

## 什么是内存安全

让我们举个例子：

```cpp
void example() {  
    vector<string> vector;  
    ...  
  
    auto& elem = vector[0];  
    vector.push_back(some_string);  
    cout << elem;  
}
```

我们知道，`vector` 内部保存着一个数组，当 `push_back` 被调用时，它会查看该数组还有多少剩余空间，若空间不足，则会开辟新的空间，并将原数组的内容拷贝，如：

```cpp
// this code might not compile, but you got the idea.  
void push_back(string elem) {  
    if (this.size == this.capacity) {  
        string* new_data = new string[this.capacity * 2];  
        memcpy(new_data, this.data, this.size * sizeof(string*));  
  
        delete[] this.data;  // the old array is free  
  
        this.data = data;  
    }  
  
    this.data[this.size++] = elem;  
}
```

即在执行 `vector.push_back` 时，`elem` 指向的内存已经被释放了，造成了“访问已释放内存”的问题。也许程序不会直接崩溃，但极可能得到的错误的结果。

上面的例子中，产生“内存安全”的原因是同时达成了两个因素：

1. 存在别名。即不同的变量（`elem` 和 `vector`）指向了同一块内存区域。
2. 存在修改。即 `push_back` 过程中 `delete` 了该内存。

## Ownership 及 Borrowing

Rust 提出了 Ownership（所有权）及 Borrowing（租借）的概念，做了如下限制：

1. 所有的资源只能有一个主人（owner）。
2. 其它人可以租借这个资源。
3. 但当这个资源被借走时，主人不允许释放或修改该资源。

可以看到，这 3 条规则的目的是防止“存在别名”和“存在修改”同时发生。一个资源如果被共享了，则不允许修改；如果想修改资源，则不允许共享。

想象有一本书（资源），则依照上述 3 个准则，有：

_1_. 它只有一个主人。当然你可以把书“给”其它人，所有权就归其它人。

```rust
fn main() {  
    let a = String::from("book"); // "book" 归 a 所有  
    let b = a;                    // a 将 "book" 转让给 b  
    println!("a = {}", a);        // 出错，a 已经无权使用资源  
}
```

_2_. 允许租借。你可以先把书“给”别人，别人用完后再“给”你。但 rust 中的“借”，则保证了对方不会不把书还你。例如：

```rust
pub fn main() {  
    let a = String::from("book");  
    {  
        let b = a;            // a 将 "book" 转让给 b  
    }                         // b 死了，却没有将 "book" 还给 a  
    println!("a = '{}'", a);  // 出错，"book" 不在 a 手上。  
}
```

你可以将书借给多个人（想象几个人一起看书），前提是它们只想“读”这本书，即 rust 允许有多个不可变的引用 (&T)：

```rust
pub fn main() {  
    let mut a = String::from("book");  
    let b = &a;       // "book" 借给 b 只读  
    let c = &a;       // "book" 同时 借给 c 只读  
    println!("a = '{}'", a);  
    println!("b = '{}'", b);  
    println!("b = '{}'", c);  
}
```

如果有一个人将书借去“写”，则不允许其它人同时“读”，即 rust 只允许有一个可变的引用(&mut T)：

```rust
pub fn main() {  
    let mut a = String::from("book");  
    let b = &mut a;       // "book" 借给 b 写  
    let c = &a;           // 错误，有人借书“写”时，不允许借来“读”  
}
```

_3_. 如果有人还借着书（无论读写），不允许主人修改或销毁书。

```rust
pub fn main() {  
    let b;  
    {  
        let a = String::from("book");  
        b = &a;         // "book" 借给 b  
    }                   // 错误，a 死亡，需要销毁书，但 b 还借着书  
}
```

最后，当拥有者死亡时，rust 会销毁它拥有的资源，由于一份资源只有一个拥有者，因此并不会造成销毁多次的情况。

这三条规则一起，保证了“存在别名”和“存在修改”不会同时发生，最终保证了内存安全，同时防止了多线程的数据竞争。

## Lifetime

我们再回顾上节关于 Ownership 的三条规则，以便分析：

1. 所有的资源只能有一个主人（owner）。
2. 其它人可以租借这个资源。
    1. 同时可以有多个不可变引用(&T)。
    2. 同时只可以有一个可变引用(&mut T)。
3. 但当这个资源被借走时，主人不允许释放或修改该资源。

rust 需要在编译期间就要保证我们的代码不会违反上面三条限制，这样做最大的优点就是不需要 runtime ，也就是不会增加额外的运行时开销。那么编译器又是如何通过静态分析来保证上述限制呢？

一个很直接的想法（不代表实际实现）是：为每个变量维护一个集合，集合里记录该变量的引用（Reference，也就是租借），那么编译器在分析时就能确保规则 #1, #2.1, #2.2。而为了确保规则 #3，rust 编译器需要确保一个资源的 reference 的存在时间小于比资源的 Owner 的存在时间。

Lifetime （生命周期）是 rust 编译器用于对比资源 owner 的存在时间与资源 reference 的存在时间的工具。Lifetime 可以理解为变量的作用域，例如：

```rust
pub fn main() {  
    let mut a = String::from("book");  
    let x = &a;  
    a.push('A');    // 违反 #3 存在 a 的引用，不允许修改  
}
```

上例中，

```rust
{    a    x    *    }  
所有者 a:        |______________|  
借用者 x:             |_________|   x = &a  
  修改 a:                  |        失败：存在 a 的引用 x 违反 #3
```

而下例中

```rust
pub fn main() {  
    let mut a = String::from("book");  
    {  
        let x = &a;  
    }                // x 作用域结束  
    a.push('A');     // 成功：所有对 a  的引用已经结束  
}
```

对应是作用域为：

```rust
{    a    {    x    }    *    }  
所有者 a:        |________________________|  
借用者 x:                  |____|            x = &a  
  修改 a:                            |       成功：对 a 的引用已经结束
```

可以看到，通过对作用域的分析，rust 编译器就能够保证资源的 owner 存活时间比资源的引用更长。

### 人为标注生命周期

上面的例子较为简单，编译器可以做一些自动的分析来判断代码是否佥，但还有一些情况下，编译器并没有办法知道生命周期的是否合法，如：

```rust
fn foo(x: &str, y: &str) -> &str {  
    if random() % 2 == 0 { x } else { y }  
}  
  
fn main() {  
    let x = String::from("X");  
    let z;  
    {  
        let y = String::from("Y");  
        z = foo(&x, &y);  
    }                       // ①  
    println!("z = {}", z);  
}
```

上述例子中，如果 `foo` 返回了 `x` 的值，由于变量 `z` 生命周期小于 `x` 因此不会产生内存安全问题；但当 `foo` 返回 `y` 时，①处 `y` 作用域结束，但 `z` 依旧持有 `y` 的引用，因此存在内存安全问题。

这里的问题是单凭静态分析本身并没有办法确定所有的生命周期，因此需要一定的人工介入，人为地给编译器一些提示：

```rust
fn foo<'a>(x: &'a str, y: &'a str) -> &'a str {  
    if random() % 2 == 0 { x } else { y }  
}
```

上述标识的含义是，函数 `foo` 的返回值的生命周期，要小于任意参数的生命周期。有了这个提示，编译器就很容易知道下例中的代码违反了这个约定。

```rust
pub fn main() {  
    let x = String::from("X");  
    let z;  
    let y = String::from("Y");  
    z = foo(&x, &y);  
    println!("z = {}", z);  
}
```

给出作用域如下：

```rust
{    x    z    y     *    }  
x:        |____________________|  
z:             |_______________|  
y:                  |__________|  
foo 的要求： Lifetime(z) <= Lifetime(x) & Lifetime(y) // 不成立
```

### 作用域作为生命周期的不足

上小节的例子说明了为什么 rust 需要引入 Lifetime 的概念，以及为什么在一些情况下需要人为指定 Lifetime。只是使用变量的作用域作为生命周期会有“误判”，即某些并没有违反规则 #3 的情形也会被 rust 认为是非法的。例如：

```rust
pub fn main() {  
    let mut a = String::from("book1");  
    let mut b = String::from("book2");  
    let mut c = &mut a;  
    c = &mut b;  
    a.push('C');  // ① rust 报错：已存在对 a 的可变引用  
}
```

上述代码中，rust 认为①处存在对变量 `a` 的引用，原因是变量 `c` 是对 `a`的引用，并且在 ①处 `c` 的作用域还未结束，因此认为依旧存在对 `a` 的引用。但实际上 `c` 对 `a` 的引用已经结束。这也是直接用作用域作为生命周期的不足，在 rust 中可以通过如下方案绕过：

```rust
pub fn main() {  
    let mut a = String::from("book1");  
    let mut b = String::from("book2");  
    {  
        let mut c = &mut a;  
        c = &mut b;  
    }  
    a.push('C');  
}
```

## 如何指定 Lifetime

虽然理论上，我们可以指定各种复杂的 Lifetime 规则，但由于我们指定的规则是作用在编译期的静态分析，所以我们指定的规则有一定的要求，具体如下：

```rust
fn foo<'X, 'Y, 'Z>(x: &'X str, x: &'Y str, x: &'Z str) -> &'R str {  
    ...  
}
```

_Lifetime 推导公式_ ： 当输出值 `R` 依赖输入值 `X` `Y` `Z` …，当且仅当输出值的 Lifetime 为所有输入值的 Lifetime 交集的子集时，生命周期合法。

$$Lifetime(R) ⊆ ( Lifetime(X) ∩ Lifetime(Y) ∩ Lifetime(Z) ∩ Lifetime(...) )$$

因此，下例中指定的 Lifetime 是非法的。

```rust
fn foo<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {  
    if true { x } else { y }  
}
```

因为

$$Lifetime(返回值) ⊆ ( Lifetime(x) ∩ Lifetime(y) )  $$
  
即：  
  
$$'a ⊆ ('a ∩ 'b)  // 'b 可能小于 'a ，因此不总是成立$$

上面的规则本质上就是要求函数返回值的 Lifetime 要小于任意一个参数的 Lifetime。为什么需要这样的规则呢？我们重用上节用到的一个例子，如下：

```rust
fn foo<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {  
    if random() % 2 == 0 { x } else { y }  
}  
  
// 下面的代码编译不过，用来演示如果没有 lifetime 分析可能造成的“不安全”  
fn main() {  
    let x = String::from("X");  
    let z;  
    {  
        let y = String::from("Y");  
        z = foo(&x, &y);  // ①  
    }  
    println!("z = {}", z);  
}
```

由于 rust 做的是静态分析，因此在 ① 处分析时，`z` 的 Lifetime 为函数 `foo` 返回值的 Lifetime `'a`，它小于变量 `x` 的生命周期，因此如果 rust 不强制执行 Lifetime 的推导规则，则上述代码能通过静态分析，但若运行时函数 `foo` 返回了 `y`，则又产生了内存安全的问题。上例中的 random 函数修复如下：

```rust
fn foo<'a, 'b: 'a>(x: &'a str, y: &'b str) -> &'a str
```

`'b: 'a` 表示 Lifetime `'b` 比 `'a` 活得长 (outlive)。因此可以通过 Lifetime 规则。

如上，即使 rust 需要我们人工指定一些生命周期，它对指定的内容也是有要求的，要求就是函数返回值的生命周期要小于任意一个参数的生命周期，这样静态分析的结果才能保证运行时的正确性。

## 小结

内存安全、数据竞争等问题的根源是“共享可变数据”，C/C++ 语言将这些问题完全交结程序员，由程序员保证不出错；Java 采用 GC 解决内存回收问题，但依旧面临着数据竞争等问题，需要程序员处理；一些函数式语言，诸如 Haskell, Clojure 针对“共享可变数据”中的“可变”，强制要求数据是“不可变”的，以解决上述问题；而 rust 另辟蹊径处理了“共享”的问题，来达到同样的效果。

当然，在编程语言降低我们出错风险的同时，也剥夺了我们的“自由”与“权力”。有些语言让我们付出的代价是性能，而 rust 需要的则是程序员付出更多的学习时间。

## 参考资料

- [Stanford Seminar - The Rust Programming Language](https://www.youtube.com/watch?v=O5vzLKg7y-k) Aaron Turon 的演讲，对理解 rust 的共享模型很有帮助。
- [rustprimer lifetime](https://wayslog.gitbooks.io/rustprimer/content/ownership-system/lifetime.html) 通俗易懂的 Lifetime 讲解，文中多个例子来源于此。