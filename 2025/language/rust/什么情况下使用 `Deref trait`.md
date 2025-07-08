`Deref` trait 是 Rust 中的一个特性，它允许我们 <font color="#f1c17d"><b>重载解引用运算符</b></font>  `*`。 这个特性在 Rust 中非常有用， 因为<font color="#DF718A"><b>它允许我们在自定义类型上使用解引用运算符，而不需要手动调用</b></font> `*`。

在Rust中，`Deref` trait 是一个非常强大的工具， <font color="#DF718A"><b>它允许你通过解引用运算符</b></font>（`*`）<font color="#80AEE7"><b>来访问底层数据</b></font>。

`Deref` trait <font color="#f1c17d"><b>最常见的用途之一</b></font> 是将自定义智能指针类型转换为其内部持有的数据类型。

通过实现 `Deref` trait，你可以使你的自定义类型与标准库中的类型（如引用和智能指针）具有相同的行为。

## 使用 Deref 的情形

- **自定义智能指针**：如果你创建了一个自定义智能指针类型，可以通过实现 Deref trait 来使其像标准指针一样工作。
- **类型转换**：当你希望你的类型在某些上下文中像另一个类型一样工作时，可以使用 Deref 进行隐式转换。
- **函数调用简化**：当你希望你的类型能够自动解引用以便调用底层类型的方法时，Deref 非常有用。

## 情形1: 自定义智能指针

**情形说明**：

- 如果你创建了一个自定义智能指针类型，可以通过实现 `Deref` trait 来使其像标准指针一样工作。

```rust
use std::ops::Deref;

struct MyBox<T> {
  value: T,
}

impl<T> MyBox<T> {
  fn new(value: T) -> MyBox<T> {
    MyBox { value }
  }
}

impl<T> Deref for MyBox<T> {
  type Target = T;

  fn deref(&self) -> &T {
    &self.value
  }
}

fn main() {
  let x = 5;
  let y = MyBox::new(x);

  // 通过 Deref trait 自动解引用
  println!("Value in MyBox: {}", *y);
}
```

## 情形2: 类型转换

**情形说明**：

- 当你希望你的类型在某些上下文中像另一个类型一样工作时，可以使用 `Deref`  <font color="#f1c17d"><b>进行隐式转换</b></font>

```rust
use std::ops::Deref;

struct MyBox<T> {
    value: T,
}

impl<T> MyBox<T> {
    fn new(value: T) -> MyBox<T> {
        MyBox { value }
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.value
    }
}

fn print_value(value: &i32) {
    println!("Value: {}", value);
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    // 通过 Deref trait 自动解引用
    print_value(&y);
}
```

## 情形3： 函数调用简化

**情形说明**：

- 当你希望你的类型能够自动解引用以便调用底层类型的方法时，`Deref` 非常有用

<font color="#DF718A"><b>如果参数传递的是结构体的引用，并且你想要在使用该引用时像操作结构体本身一样使用</b></font>，

<font color="#80AEE7">那么实现 Deref trait 是一个很好的选择</font></font>。

**通过实现 `Deref` trait，你可以让结构体的引用表现得像结构体本身一样，从而简化代码并提高可读性。**

## 总结

`Deref` trait 主要用在智能指针类型和自定义类型上， 它可以帮助我们简化代码、减少重复、提高可读性。

Rust中传参，优先使用引用，而不是值， 实现 `Deref` trait 可以让你在使用引用时像操作结构体本身一样使用， 从而提高代码的可读性和简洁性。