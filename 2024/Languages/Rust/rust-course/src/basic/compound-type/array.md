# 数组

在日常开发中，使用最广的数据结构之一就是数组，在 Rust 中，最常用的数组有两种，第一种是速度很快但是长度固定的 `array`，第二种是可动态增长的但是有性能损耗的 `Vector`，在本书中，我们称 `array` 为数组，`Vector` 为动态数组。

不知道你们发现没，这两个数组的关系跟 `&str` 与 `String` 的关系很像，前者是长度固定的字符串切片，后者是可动态增长的字符串。其实，在 Rust 中无论是 `String` 还是 `Vector`，它们都是 Rust 的高级类型：集合类型，在后面章节会有详细介绍。

对于本章节，我们的重点还是放在数组 `array` 上。数组的具体定义很简单：将多个类型相同的元素依次组合在一起，就是一个数组。结合上面的内容，可以得出数组的三要素：

- 长度固定
- 元素必须有相同的类型
- 依次线性排列

这里再啰嗦一句，**我们这里说的数组是 Rust 的基本类型，是固定长度的，这点与其他编程语言不同，其它编程语言的数组往往是可变长度的，与 Rust 中的动态数组 `Vector` 类似**，希望读者大大牢记此点。

### 创建数组

在 Rust 中，数组是这样定义的：

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

数组语法跟 JavaScript 很像，也跟大多数编程语言很像。由于它的元素类型大小固定，且长度也是固定，因此**数组 `array` 是存储在栈上**，性能也会非常优秀。与此对应，**动态数组 `Vector` 是存储在堆上**，因此长度可以动态改变。当你不确定是使用数组还是动态数组时，那就应该使用后者，具体见[动态数组 Vector](https://course.rs/basic/collections/vector.html)。

举个例子，在需要知道一年中各个月份名称的程序中，你很可能希望使用的是数组而不是动态数组。因为月份是固定的，它总是只包含 12 个元素：

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

在一些时候，还需要为**数组声明类型**，如下所示：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

这里，数组类型是通过方括号语法声明，`i32` 是元素类型，分号后面的数字 `5` 是数组长度，数组类型也从侧面说明了**数组的元素类型要统一，长度要固定**。

还可以使用下面的语法初始化一个**某个值重复出现 N 次的数组**：

```rust
let a = [3; 5];
```

`a` 数组包含 `5` 个元素，这些元素的初始化值为 `3`，聪明的读者已经发现，这种语法跟数组类型的声明语法其实是保持一致的：`[3; 5]` 和 `[类型; 长度]`。

在元素重复的场景，这种写法要简单的多，否则你就得疯狂敲击键盘：`let a = [3, 3, 3, 3, 3];`，不过老板可能很喜欢你的这种疯狂编程的状态。

### 访问数组元素

因为数组是连续存放元素的，因此可以通过索引的方式来访问存放其中的元素：

```rust
fn main() {
    let a = [9, 8, 7, 6, 5];

    let first = a[0]; // 获取a数组第一个元素
    let second = a[1]; // 获取第二个元素
}
```

与许多语言类似，数组的索引下标是从 0 开始的。此处，`first` 获取到的值是 `9`，`second` 是 `8`。

#### 越界访问

如果使用超出数组范围的索引访问数组元素，会怎么样？下面是一个接收用户的控制台输入，然后将其作为索引访问数组元素的例子：

```rust
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();
    // 读取控制台的输出
    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!(
        "The value of the element at index {} is: {}",
        index, element
    );
}
```

使用 `cargo run` 来运行代码，因为数组只有 5 个元素，如果我们试图输入 `5` 去访问第 6 个元素，则会访问到不存在的数组元素，最终程序会崩溃退出：

```console
Please enter an array index.
5
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 5', src/main.rs:19:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

这就是数组访问越界，访问了数组中不存在的元素，导致 Rust 运行时错误。程序因此退出并显示错误消息，未执行最后的 `println!` 语句。

当你尝试使用索引访问元素时，Rust 将检查你指定的索引是否小于数组长度。如果索引大于或等于数组长度，Rust 会出现 **_panic_**。这种检查只能在运行时进行，比如在上面这种情况下，编译器无法在编译期知道用户运行代码时将输入什么值。

这种就是 Rust 的安全特性之一。在很多系统编程语言中，并不会检查数组越界问题，你会访问到无效的内存地址获取到一个风马牛不相及的值，最终导致在程序逻辑上出现大问题，而且这种问题会非常难以检查。

#### 数组元素为非基础类型

学习了上面的知识，很多朋友肯定觉得已经学会了 Rust 的数组类型，但现实会给我们一记重锤，实际开发中还会碰到一种情况，就是**数组元素是非基本类型**的，这时候大家一定会这样写。

```rust
let array = [String::from("rust is good!"); 8];

println!("{:#?}", array);
```

然后你会惊喜的得到编译错误。

```console
error[E0277]: the trait bound `String: std::marker::Copy` is not satisfied
 --> src/main.rs:7:18
  |
7 |     let array = [String::from("rust is good!"); 8];
  |                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::marker::Copy` is not implemented for `String`
  |
  = note: the `Copy` trait is required because this value will be copied for each element of the array
```

有些还没有看过特征的小伙伴，有可能不太明白这个报错，不过这个目前可以不提，我们就拿之前所学的[所有权](https://course.rs/basic/ownership/ownership.html)知识，就可以思考明白，前面几个例子都是 Rust 的基本类型，而**基本类型在 Rust 中赋值是以 Copy 的形式**，这时候你就懂了吧，`let array=[3;5]`底层就是不断的Copy出来的，但很可惜复杂类型都没有深拷贝，只能一个个创建。

接着就有小伙伴会这样写。

```rust
let array = [String::from("rust is good!"),String::from("rust is good!"),String::from("rust is good!")];

println!("{:#?}", array);
```

作为一个追求极致完美的Rust开发者，怎么能容忍上面这么难看的代码存在！

**正确的写法**，应该调用`std::array::from_fn`

```rust
let array: [String; 8] = std::array::from_fn(|_i| String::from("rust is good!"));

println!("{:#?}", array);
```

## 数组切片

在之前的[章节](https://course.rs/basic/compound-type/string-slice.html#切片slice)，我们有讲到 `切片` 这个概念，它允许你引用集合中的部分连续片段，而不是整个集合，对于数组也是，数组切片允许我们引用数组的一部分：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];

let slice: &[i32] = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

上面的数组切片 `slice` 的类型是`&[i32]`，与之对比，数组的类型是`[i32;5]`，简单总结下切片的特点：

- 切片的长度可以与数组不同，并不是固定的，而是取决于你使用时指定的起始和结束位置
- 创建切片的代价非常小，因为切片只是针对底层数组的一个引用
- 切片类型 [T] 拥有不固定的大小，而切片引用类型 &[T] 则具有固定的大小，因为 Rust 很多时候都需要固定大小数据类型，因此 &[T] 更有用，`&str` 字符串切片也同理

## 总结

最后，让我们以一个综合性使用数组的例子，来结束本章节的学习：

```rust
fn main() {
  // 编译器自动推导出one的类型
  let one             = [1, 2, 3];
  // 显式类型标注
  let two: [u8; 3]    = [1, 2, 3];
  let blank1          = [0; 3];
  let blank2: [u8; 3] = [0; 3];

  // arrays是一个二维数组，其中每一个元素都是一个数组，元素类型是[u8; 3]
  let arrays: [[u8; 3]; 4]  = [one, two, blank1, blank2];

  // 借用arrays的元素用作循环中
  for a in &arrays {
    print!("{:?}: ", a);
    // 将a变成一个迭代器，用于循环
    // 你也可以直接用for n in a {}来进行循环
    for n in a.iter() {
      print!("\t{} + 10 = {}", n, n+10);
    }

    let mut sum = 0;
    // 0..a.len,是一个 Rust 的语法糖，其实就等于一个数组，元素是从0,1,2一直增加到到a.len-1
    for i in 0..a.len() {
      sum += a[i];
    }
    println!("\t({:?} = {})", a, sum);
  }
}
```

做个总结，数组虽然很简单，但是其实还是存在几个要注意的点：

- **数组类型容易跟数组切片混淆**，[T;n] 描述了一个数组的类型，而 [T] 描述了切片的类型， 因为切片是运行期的数据结构，它的长度无法在编译期得知，因此不能用 [T;n] 的形式去描述
- `[u8; 3]`和`[u8; 4]`是不同的类型，数组的长度也是类型的一部分
- **在实际开发中，使用最多的是数组切片[T]**，我们往往通过引用的方式去使用`&[T]`，因为后者有固定的类型大小

至此，关于数据类型部分，我们已经全部学完了，对于 Rust 学习而言，我们也迈出了坚定的第一步，后面将开始更高级特性的学习。未来如果大家有疑惑需要检索知识，一样可以继续回顾过往的章节，因为本书不仅仅是一门 Rust 的教程，还是一本厚重的 Rust 工具书。

## 课后练习

> [Rust By Practice](https://practice-zh.course.rs/compound-types/array.html)，支持代码在线编辑和运行，并提供详细的[习题解答](https://github.com/sunface/rust-by-practice/blob/master/solutions/compound-types/array.md)。
