#Rust 

本文展示了如何在Rust中使用PhantomType消除重复代码；

源代码：

- [https://github.com/JasonkayZK/rust-learn/tree/feature-phantom](https://github.com/JasonkayZK/rust-learn/tree/feature-phantom)

# **Rust中的PhantomType**

## **引出问题**

如果我们有一个表示距离的类型`Meter`：

examples/0_normal_type.rs

```rust
#[derive(Debug, Copy, Clone)]
struct Meter {
    value: f64,
}

impl Meter {
    fn new(value: f64) -> Self {
        Self { value }
    }
}
```

此时我们需要其支持加法和减法运算，因此我们为其实现了`Add`和`Sub` Trait：

```rust
impl Add for Meter {
    type Output = Meter;

    fn add(self, rhs: Self) -> Self::Output {
        let value = self.value + rhs.value;
        Meter { value }
    }
}

impl Sub for Meter {
    type Output = Meter;

    fn sub(self, rhs: Self) -> Self::Output {
        let value = self.value - rhs.value;
        Meter { value }
    }
}

```

测试一下：

```rust
fn main() {
    let one = Meter::new(1.0);
    let three = Meter::new(3.3);

    let four = one + three;
    dbg!(&four);

    let two = three - one;
    dbg!(&two);
}
```

输出如下：

```sh
[src/main.rs:37:5] &four = Meter {
    value: 4.3,
}
[src/main.rs:40:5] &two = Meter {
    value: 2.3,
}
```

如果此时，我们还需要为`Kilogram`、`Liter`等类型实现相同的逻辑，则需要重复的实现多个`Add`和`Sub` Trait；

> 虽然我们可以使用过程宏（Macro）实现；

实际上，在Rust中可以使用一个`PhantomType`来实现，即虚拟的类型！

## 使用 PhantomType

`PhantomType`指的是那些在Runtime中实际上并不存在的类型，但是可以帮助代码在编译器对类型做一些限制；

> **概念上有些类似于Go中泛型的实现；**

我们可以定义一个通用类型：`Unit<T>`并为其实现`Add`和`Sub` Trait；

examples/1_phantom_type.rs

```rust
#[derive(Debug, Copy, Clone)]
pub struct Unit<T> {
    value: f64,
}

#[derive(Debug, Copy, Clone)]
pub struct MeterType;

pub type Meter = Unit<MeterType>;

#[derive(Debug, Copy, Clone)]
pub struct KilogramType;
pub type Kilogram = Unit<KilogramType>;
```

上面同时声明了Meter和Kilogram类型，`MeterType`和`KilogramType`的大小都是0；

同时，得益于Rust的`零抽象成本`，声明的多个类型在运行时没有其他额外开销！

编译上面的代码会报错：

```sh
error[E0392]: parameter `T` is never used
  --> src/main.rs:38:13
   |
38 | struct Unit<T> {
   |             ^ unused parameter
   = help: consider removing `T`, referring to it in a field, or using a marker such as `PhantomData`
   = help: if you intended `T` to be a const parameter, use `const T: usize` instead
```

编译器要求泛型必须要被使用，因此我们修改`Unit`声明：

```rust
#[derive(Debug, Copy, Clone)]
pub struct Unit<T> {
    value: f64,
    unit_type: T,
}
```

并增加构造函数：

```rust
impl<T: Default> Unit<T> {
    pub fn new(value: f64) -> Self {
        Self {
            value,
            unit_type: T::default(),
        }
    }
}
```

这里使用的构造函数是在类型T中默认的构造函数，因此避免了此时由于T的类型未确定而无法初始化的问题；

为Unit实现`Add`和`Sub` Trait：

```rust
impl<T: Default> Add for Unit<T> {
    type Output = Unit<T>;

    fn add(self, rhs: Self) -> Self::Output {
        let new_value = self.value + rhs.value;
        Unit::new(new_value)
    }
}

impl<T: Default> Sub for Unit<T> {
    type Output = Unit<T>;

    fn sub(self, rhs: Self) -> Self::Output {
        let new_value = self.value - rhs.value;
        Unit::new(new_value)
    }
}
```

最后，为我们的`MeterType`和`LiterType`类型实现默认构造函数：

```rust
impl Default for MeterType {
    fn default() -> Self {
        MeterType
    }
}

impl Default for KilogramType {
    fn default() -> Self {
        KilogramType
    }
}
```

下面，测试我们的代码：

```rust
mod types;

#[cfg(test)]
mod tests {
    use tests::types::{Kilogram, Meter};

    use super::*;

    #[test]
    fn test() {
        let one = Meter::new(1.1);
        let three = Meter::new(3.3);

        let four = one + three;
        println!("{:?}", four);
        dbg!(&four);

        let one = Kilogram::new(1.1);
        let three = Kilogram::new(3.3);
        let four = one + three;
        println!("{:?}", four);
        dbg!(&four);
    }
}

```

可以看到，`Meter`和`Kilogram`类型都已经实现了加法；

而对于`Meter`和`Kilogram`相互之间，是无法操作的，完美！

  

## **使用PhantomData**

其实在Rust标准库中已经提供了PhantomData类型，用于作为某个结构体数据类型的占位：[PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html)；

实际上Rust的编译器已经提醒了我们：

```sh
help: consider removing `T`, referring to it in a field, or using a marker such as `PhantomData`
```

> 官方文档中对PhantomData的说明：
> 
> Zero-sized type used to mark things that “act like” they own a T.
> 
> 一个零大小的类型，用于标志他们对一个T类型的所有权；
> 
>   
> 
> 补充PhantomType说明：
> 
> Though they both have scary names, PhantomData and ‘phantom types’ are related, but not identical. 
> 
> A phantom type parameter is simply a type parameter which is never used. In Rust, this often causes the compiler to complain, and the solution is to add a “dummy” use by way of PhantomData.
> 
> PhantomData和PhantomType是不同的；
> 
> PhantomType仅仅被用于声明一个永远不会被用到的参数，在Rust中经常会导致编译器告警，因此经常会使用PhantomData代替；

下面的代码使用了PhantomData：

examples/2_phantom_data.rs

```rust
use std::marker::PhantomData;
use std::ops::{Add, Sub};

#[derive(Debug, Copy, Clone)]
struct Unit<T> {
    value: f64,
    unit_type: PhantomData<T>,
}

impl<T> Unit<T> {
    fn new(value: f64) -> Self {
        Self {
            value,
            unit_type: PhantomData,
        }
    }
}

#[derive(Debug, Copy, Clone)]
struct MeterType;
type Meter = Unit<MeterType>;

#[derive(Debug, Copy, Clone)]
struct KilogramType;
type Kilogram = Unit<KilogramType>;

impl<T> Add for Unit<T> {
    type Output = Unit<T>;

    fn add(self, another: Unit<T>) -> Self::Output {
        let new_value = self.value + another.value;
        Unit::new(new_value)
    }
}

impl<T> Sub for Unit<T> {
    type Output = Unit<T>;

    fn sub(self, another: Unit<T>) -> Self::Output {
        let new_value = self.value - another.value;
        Unit::new(new_value)
    }
}

fn main() {
    let one = Meter::new(1.1);
    let three = Meter::new(3.3);

    let four = one + three;
    dbg!(&four);

    let one = Kilogram::new(1.1);
    let three = Kilogram::new(3.3);

    let four = one + three;
    dbg!(&four);

    // Compiling err!

    // let one = Meter::new(1.1);
    // let three = Kilogram::new(3.3);

    // let four = one + three;
    // dbg!(&four);
}
```

从上面的代码可以看到，得益于PhantomData：

- 我们直接使用PhantomData对unit_type进行了初始化，而无需为各种类型实现`Default` Trait；
- 同时，使用 `PhantomData<T>`，我们可以将类型参数的用途明确地在代码中传达；