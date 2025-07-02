> serder version: 1.0
> rust version 1.41 (1.31+)

参考

* [Github](https://github.com/serde-rs/serde)
* [官方文档](https://serde.rs/)

创建测试项目 `cargo new serde-learn --lib`

依赖 `Cargo.toml`

```toml
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
```

## 一、总览

### 1、简介

Serde 是 Rust 生态中最主流的 序列化、反序列化框架

设计上，基于 Rust 的静态类型系统和元编程（宏）的能力，使Serde序列化的执行速度与手写序列化器的速度相同。

使用上及其简单

* 用户 为自己的类型 `Serialize` 和 `Deserialize` 特质即可 （大多数情况下使用过 `derive` 宏即可）
* 序列化提供商，提供 `Serializer` 和 `Deserializer` 特质的实现即可。

Serde 及社区提供主流的序列化协议的支持，具体如下

* [JSON](https://github.com/serde-rs/json), the ubiquitous JavaScript Object Notation used by many HTTP APIs.
* [Bincode](https://github.com/TyOverby/bincode), a compact binary format used for IPC within the Servo rendering engine.
* [CBOR](https://github.com/pyfisch/cbor), a Concise Binary Object Representation designed for small message size without the need for version negotiation.
* [YAML](https://github.com/dtolnay/serde-yaml), a popular human-friendly configuration language that ain't markup language.
* [MessagePack](https://github.com/3Hren/msgpack-rust), an efficient binary format that resembles a compact JSON.
* [TOML](https://github.com/alexcrichton/toml-rs), a minimal configuration format used by Cargo.
* [Pickle](https://github.com/birkenfeld/serde-pickle), a format common in the Python world.
* [RON](https://github.com/ron-rs/ron), a Rusty Object Notation.
* [BSON](https://github.com/mongodb/bson-rust), the data storage and network transfer format used by MongoDB.
* [Avro](https://github.com/flavray/avro-rs), a binary format used within Apache Hadoop, with support for schema definition.
* [JSON5](https://github.com/callum-oakley/json5-rs), A superset of JSON including some productions from ES5.
* [Postcard](https://github.com/jamesmunns/postcard), a no_std and embedded-systems friendly compact binary format.
* [URL](https://github.com/nox/serde_urlencoded), the x-www-form-urlencoded format.
* [Envy](https://github.com/softprops/envy), a way to deserialize environment variables into Rust structs. (deserialization only)
* [Envy Store](https://github.com/softprops/envy-store), a way to deserialize AWS Parameter Store parameters into Rust structs. (deserialization only)
* [S-expressions](https://github.com/rotty/lexpr-rs), the textual representation of code and data used by the Lisp language family.

### 2、数据结构

Serde 对常见 Rust 标准库数据结构 提供了开箱即用的实现。例如：`String`, `&str`, `usize`, `Vec<T>`, `HashMap<K,V>`。

对于自定义类型可以通过 派生宏 提供支持

`src/ch01_overview.rs`

```rust
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let point = Point { x: 1, y: 2 };

        // Convert the Point to a JSON string.
        let serialized = serde_json::to_string(&point).unwrap();

        // Prints serialized = {"x":1,"y":2}
        println!("serialized = {}", serialized);

        // Convert the JSON string back to a Point.
        let deserialized: Point = serde_json::from_str(&serialized).unwrap();

        // Prints deserialized = Point { x: 1, y: 2 }
        println!("deserialized = {:?}", deserialized);
    }
}
```

## 二、Serde 数据模型

Serde数据模型是 与 Rust 数据结构和数据格式进行交互的API。您可以将其视为Serde的类型系统，Serde 将 Rust 类型分为 29 种

* 针对需要序列化的类型，用户需要 实现 `Serialize`：根据 Rust 类型调用参数 `Serializer` 上的方法，而 `Serializer` 的实现由序列化提供商提供
* 针对需要反序列化的类型，用户需要 实现 `Deserialize`：根据 Rust 类型调用参数 `Serializer` 上的方法，传递一个 实现了 Visitor 的类型

### 1、29 种类型

* 14 基础类型
    * bool
    * i8, i16, i32, i64, i128
    * u8, u16, u32, u64, u128
    * f32, f64
    * char
* string
    * 有长度标记的UTF-8 字节数据（不是`\0`结尾的形式），可能长度为0
    * 在序列化时，所有类型的字符串被同等处理。在反序列化时，有三种方案：transient, 拥有所有权, 和借用。参见《理解反序列化生命周期》，（Serde使用零拷贝技术）
* byte array - `[u8]`
    * 与字符串相似，在反序列化期间，字节数组可以是 transient, 拥有所有权, 和借用
* option
    * None 或者 Value
* unit
    * Rust 中 `()` 的类型，它表示不包含数据的匿名值
* unit_struct
    * 例如 `struct Unit` 或 `PhantomData<T>`，它表示不包含数据的命名值
* unit_variant
    * 例如 在 `enum E { A, B }` 中的 `E::A` 和 `E::B`
* newtype_struct
    * 例如 `struct Millimeters(u8)`
* newtype_variant
    * 例如 在 `enum E { N(u8) }` 中的 `E::N`
* seq
    * 可变大小的异质序列
    * 例如 `Vec<T>` 或者 `HashSet<T>`
    * 序列化时，长度在遍历之前可能是未知的。在反序列化时，通过 查看数据 可以得知长度
    * 注意，像 ` vec![Value::Bool(true), Value::Char('c')]` 之类的同质Rust集合可以序列化为异构Serde seq，在这种情况下，包含Serde bool和Serde char。
* tuple
    * 大小静态可知的异质序列
    * 例如 `(u8,)` 或 `(String, u64, Vec<T>)` 或 `[u64; 10]`
    * 其长度在反序列化时就已知道，无需查看数据
* tuple_struct
    * 命名元组，例如 `struct Rgb(u8, u8, u8)`
* tuple_variant
    * 例如 在 `enum E { T(u8, u8) }` 中 的 `E::T`
* map
    * 大小可变的异类键值对，例如 `BTreeMap <K, V>`。进行序列化时，在遍历所有条目之前，长度可能未知，也可能未知。反序列化时，通过 查看数据 可以得知长度
* struct
    * 静态大小的异构键值对，其中的键是编译时常量字符串，并且在反序列化时无需查看序列化数据即可知道
    * 例如 `struct S { r: u8, g: u8, b: u8 }`
* struct_variant
    * 例如 在 `enum E { S { r: u8, g: u8, b: u8 } }` 中 的 `E::S`

### 2、映射到数据模型

对于大多数Rust类型，将它们映射到Serde数据模型中非常简单。例如，Rust bool类型对应于Serde的bool类型。Rust元组结构 `Rgb(u8, u8, u8)` 对应于Serde的元组结构类型。

但是不一定就是这种简单一一对应的映射，例如：

`std::ffi::OsString` 类型，在 Windows 和 Unix 下表现不一致。直觉上应该映射为 `string`，但是跨平台无法使用。因此更好的方案是：

```rust
enum OsString {
    Unix(Vec<u8>),
    Windows(Vec<u16>),
    // and other platforms
}
```

## 三、派生宏

> 实例细节参见： https://github.com/serde-rs/serde/blob/master/test_suite/tests/test_gen.rs

### 1、实例

参见：一、总览 2、数据结构

### 2、属性

属性用于使用派生宏的一些配置，主要分为3类：

* 容器属性 Container attributes — 应用在 枚举 和 结构体
* Variant属性 Variant attributes — 应用在 枚举的 Variant 上
* 字段属性 Field attributes — 应用在 结构体 和 枚举 variant 的 字段上

```rust
#[derive(Serialize, Deserialize)]
#[serde(deny_unknown_fields)]  // <-- this is a container attribute
struct S {
    #[serde(default)]  // <-- this is a field attribute
    f: i32,
}

#[derive(Serialize, Deserialize)]
#[serde(rename = "e")]  // <-- this is also a container attribute
enum E {
    #[serde(rename = "a")]  // <-- this is a variant attribute
    A(String),
}
```

#### （1）容器属性

* `#[serde(rename = "name")]`
    * 使用给定的名字而不是其Rust名
    * 同时允许如下写法
    * `#[serde(rename(serialize = "ser_name"))]`
    * `#[serde(rename(deserialize = "de_name"))]`
    * `#[serde(rename(serialize = "ser_name", deserialize = "de_name"))]`
* `#[serde(rename_all = "...")]`
    * 根据给定的大小写约定重命名所有字段（结构）或 variants（枚举）。`"..."` 的可选值为 `"lowercase"`, `"UPPERCASE"`, `"PascalCase"`, `"camelCase"`, `"snake_case"`, `"SCREAMING_SNAKE_CASE"`, `"kebab-case"`, `"SCREAMING-KEBAB-CASE"`
    * 同时允许如下写法
    * `#[serde(rename_all(serialize = "..."))]`
    * `#[serde(rename_all(deserialize = "..."))]`
    * `#[serde(rename_all(serialize = "...", deserialize = "..."))]`
* `#[serde(deny_unknown_fields)]`
    * 指定遇到未知字段时，在反序列化期间始终出错。
    * 默认情况下，对于诸如JSON之类的自描述格式，未知字段将被忽略
* `#[serde(tag = "type")]`
    * ？？
* `#[serde(tag = "t", content = "c")]`
    * ？？
* `#[serde(untagged)]`
    * ？？
* `#[serde(bound = "T: MyTrait")]`
    * 序列化和反序列化的where子句表示。这将替换Serde推断的任何特征范围。
    * 同时允许如下写法
    * `#[serde(bound(serialize = "T: MySerTrait"))]`
    * `#[serde(bound(deserialize = "T: MyDeTrait"))]`
    * `#[serde(bound(serialize = "T: MySerTrait", deserialize = "T: MyDeTrait"))]`
* `#[serde(default)]`
    * 反序列化时，从结构的 Default 实现中 填写所有缺少的字段
    * 仅允许在 struct 中使用
* `#[serde(default = "path")]`
    * 反序列化时，使用给定函数或方法返回的对象中填写所有缺少的字段。该函数声明为 `fn()->T`。
    * 例如，`default="my_default"`将调用 `my_default()`，而 `default = "SomeTrait::some_default"` 将调用`SomeTrait::some_default()`
    * 仅允许在 struct 中使用
* `#[serde(remote = "...")]`
    * ??
* `#[serde(transparent)]`
    * 只允许应用在只有一个字段枚举或者结构体上
    * 表示只序列化内部对象，不要外部支撑，比如

```rust
#[derive(Serialize, Deserialize, Debug)]
#[serde(transparent)]
struct Axis {
    x: i32
}
```

* `#[serde(from = "FromType")]`
    * 反序列化为 `FromType` 后，转换为 该类型
    * 使用条件
        * 此类型必须实现 `From<FromType>`
        * 且 `FromType` 必须实现 `Deserialize`
* `#[serde(try_from = "FromType")]`
    * 反序列化为 `FromType` 后，转换为 该类型
    * 使用条件
        * 此类型必须实现 `TryFrom<FromType>`，其中错误类型必须实现 `Display`
        * 且 `FromType` 必须实现 `Deserialize`
* `#[serde(into = "IntoType")]`
    * 序列化之前，现将该类型转换为 `IntoType` 类型，然后再序列化
    * 使用条件
        * 此类型必须实现 `Clone` 和 `Into<IntoType>`
        * 且 `IntoType` 必须实现 `Serialize`
* `#[serde(crate = "...")]`
    * 指定从生成的代码中，引用 Serde API 时，要使用的 Serde crate实例的路径。
    * 这通常仅适用于从不同板条箱中的公共宏调用重新导出的Serde

#### （2）Variant属性

* `#[serde(rename = "name")]`
    * 使用给定的名字而不是其Rust名
    * 同时允许如下写法
    * `#[serde(rename(serialize = "ser_name"))]`
    * `#[serde(rename(deserialize = "de_name"))]`
    * `#[serde(rename(serialize = "ser_name", deserialize = "de_name"))]`
* `#[serde(alias = "name")]`
    * 反序列化时，对应的别名
    * 允许配置多个
* `#[serde(rename_all = "...")]`
    * 根据给定的大小写约定重命名 struct variant。`"..."` 的可选值为 `"lowercase"`, `"UPPERCASE"`, `"PascalCase"`, `"camelCase"`, `"snake_case"`, `"SCREAMING_SNAKE_CASE"`, `"kebab-case"`, `"SCREAMING-KEBAB-CASE"`
    * 同时允许如下写法
    * `#[serde(rename_all(serialize = "..."))]`
    * `#[serde(rename_all(deserialize = "..."))]`
    * `#[serde(rename_all(serialize = "...", deserialize = "..."))]`
* `#[serde(skip)]`
    * 跳过序列化或反序列化此 variant
    * 尝试序列化时将报错
    * 尝试反序列化时将报错
* `#[serde(skip_serializing)]`
    * 尝试序列化时将报错
* `#[serde(skip_deserializing)]`
    * 尝试反序列化时将报错
* `#[serde(serialize_with = "path")]`
    * 使用 `path` 函数 进行序列化，该序列化函数声明为：`fn<S>(&FIELD0, &FIELD1, ..., S) -> Result<S::Ok, S::Error> where S: Serializer`
* `#[serde(deserialize_with = "path")]`
    * 使用 `path` 函数 进行反序列化，该序列化函数声明为：`fn<'de, D>(D) -> Result<FIELDS, D::Error> where D: Deserializer<'de>`
* `#[serde(with = "module")]`
    * 指定 `#[serde(serialize_with = "path")]` 和 `#[serde(deserialize_with = "path")]` path 所在 module
* `#[serde(bound = "T: MyTrait")]`
    * 序列化和反序列化的where子句表示。这将替换Serde推断的任何特征范围。
    * 同时允许如下写法
    * `#[serde(bound(serialize = "T: MySerTrait"))]`
    * `#[serde(bound(deserialize = "T: MyDeTrait"))]`
    * `#[serde(bound(serialize = "T: MySerTrait", deserialize = "T: MyDeTrait"))]`
* `#[serde(borrow)]` 和 `#[serde(borrow = "'a + 'b + ...")]`
    * ？？
* `#[serde(other)]`
    * ？？为匹配其他类型

#### （3）字段属性

* `#[serde(rename = "name")]`
    * 使用给定的名字而不是其Rust名
    * 同时允许如下写法
    * `#[serde(rename(serialize = "ser_name"))]`
    * `#[serde(rename(deserialize = "de_name"))]`
    * `#[serde(rename(serialize = "ser_name", deserialize = "de_name"))]`
* `#[serde(alias = "name")]`
    * 反序列化时，对应的别名
    * 允许配置多个
* `#[serde(default)]`
    * 如果反序列化时不存在该值，则使用 `Default::default()`
* `#[serde(default = "path")]`
    * 反序列化时，使用给定函数或方法返回的对象中填写所有缺少的字段。该函数声明为 `fn()->T`。
    * 例如，`default="my_default"`将调用 `my_default()`，而 `default = "SomeTrait::some_default"` 将调用`SomeTrait::some_default()`
* `#[serde(flatten)]`
    * 展平该字段，也就是将该字段内部抽到当前结构
* `#[serde(skip)]`
    * 跳过此字段：不序列化或反序列化
    * 反序列化时，Serde将使用 `Default::default()` 或 `default = "..."` 生成该值
* `#[serde(skip_serializing)]`
    * 跳过序列化该字段
* `#[serde(skip_serializing)]`
    * 反序列化时跳过该字段
    * 反序列化时，Serde将使用 `Default::default()` 或 `default = "..."` 生成该值
* `#[serde(skip_serializing_if = "path")]`
    * 调用一个函数以确定是否跳过序列化此字段。
    * 该函数声明为 `fn(＆T)-> bool`。例如 `skip_serializing_if="Option::is_none"`
* `#[serde(serialize_with = "path")]`
    * 使用 `path` 函数 进行序列化，函数声明为：`fn<S>(&T, S) -> Result<S::Ok, S::Error> where S: Serializer`
    * 这样 T 就不需要事先 `Serialize`
* `#[serde(deserialize_with = "path")]`
    * 使用 `path` 函数 进行反序列化，该序列化函数声明为：`fn<'de, D>(D) -> Result<T, D::Error> where D: Deserializer<'de>`
    * 这样 T 就不需要事先 `Serialize`
* `#[serde(with = "module")]`
    * 指定 `#[serde(serialize_with = "path")]` 和 `#[serde(deserialize_with = "path")]` path 所在 module
* `#[serde(borrow)]` 和 `#[serde(borrow = "'a + 'b + ...")]`
    * ？？？
* `#[serde(bound = "T: MyTrait")]`
    * 序列化和反序列化的where子句表示。这将替换Serde推断的任何特征范围。
    * 同时允许如下写法
    * `#[serde(bound(serialize = "T: MySerTrait"))]`
    * `#[serde(bound(deserialize = "T: MyDeTrait"))]`
    * `#[serde(bound(serialize = "T: MySerTrait", deserialize = "T: MyDeTrait"))]`
* `#[serde(getter = "...")]`
    * ???

## 四、自定义序列化/反序列化

Serde的通过 `#[derive(Serialize, Deserialize)]` 派生宏为结构体和枚举提供了合理的默认序列化行为，并且可以使用属性在某种程度上进行自定义。

但是对于特殊需求，Serde可以通过为您的类型手动实现 `Serialize` 和 `Deserialize` 特质来完全自定义序列化行为。这两个特质都有只有一个方法，声明如下

```rust
pub trait Serialize {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer;
}

pub trait Deserialize<'de>: Sized {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>;
}
```

而序列化协议只需要提供 `Serializer` 和 `Deserializer` 的实现即可。

创建测试代码 `src/ch04_custom_serde.rs`

### 1、实现 Serialize

本节参考源码： [`serde/src/ser/impls.rs`](https://github.com/serde-rs/serde/blob/master/serde/src/ser/impls.rs)

特质声明如下

```rust
pub trait Serialize {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer;
}
```

该方法的工作是：

* 通过调用给定 `Serializer` 参数上的一种方法并传递 `&self`，将 `self` 映射到 Serde数据模型中

#### （1）序列化基础数据类型

```rust
impl Serialize for i32 {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        serializer.serialize_i32(*self)
    }
}
```

此处仅为示例，Serde 已为所有基础数据类型实现了类似的实现（以上代码已经在 用户 crate 无法编译，因为不满足孤儿规则）

#### （2）序列化序列或者map

复杂类型序列化一般需要三步：初始化，放置元素，结束。

```rust
use serde::ser::{Serialize, Serializer, SerializeSeq, SerializeMap};

impl<T> Serialize for Vec<T>
where
    T: Serialize,
{
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let mut seq = serializer.serialize_seq(Some(self.len()))?;
        for e in self {
            seq.serialize_element(e)?;
        }
        seq.end()
    }
}

impl<K, V> Serialize for MyMap<K, V>
where
    K: Serialize,
    V: Serialize,
{
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let mut map = serializer.serialize_map(Some(self.len()))?;
        for (k, v) in self {
            map.serialize_entry(k, v)?;
        }
        map.end()
    }
}
```

（以上代码已经在 用户 crate 编译）

#### （3）序列化元组

通过阅读源码可知，Rust 为 长度为 0~32 的数组 和 长度为 0~16 的元组提供了实现 Serialize 实现

参见源码 `serde/src/ser/impls.rs`

#### （4）序列化结构体

```rust
use serde::{Serialize, Serializer};
use serde::ser::{SerializeStruct, SerializeTupleStruct, SerializeStructVariant, SerializeTupleVariant};


// 普通 结构体. 步骤如下:
//   1. serialize_struct
//   2. serialize_field
//   3. end
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

impl Serialize for Color {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let mut state = serializer.serialize_struct("Color", 3)?;
        state.serialize_field("r", &self.r)?;
        state.serialize_field("g", &self.g)?;
        state.serialize_field("b", &self.b)?;
        state.end()
    }
}

// 元组 结构体. 步骤如下:
//   1. serialize_tuple_struct
//   2. serialize_field
//   3. end
struct Point2D(f64, f64);

impl Serialize for Point2D {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        let mut state = serializer.serialize_tuple_struct("Point2D", 2)?;
        state.serialize_field(&self.0)?;
        state.serialize_field(&self.1)?;
        state.end()
    }
}

// newtype struct. 使用 serialize_newtype_struct.
struct Inches(u64);

impl Serialize for Inches {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        serializer.serialize_newtype_struct("Inches", &self.0)
    }
}


// unit struct. 使用 serialize_unit_struct.
struct Instance;


impl Serialize for Instance {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        serializer.serialize_unit_struct("Instance")
    }
}


#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch04() {
        println!("Color = {}", serde_json::to_string(&Color{r:1, g:2, b:3}).unwrap());
        println!("Point2D = {}", serde_json::to_string(&Point2D(1.0, 2.0)).unwrap());
        println!("Inches = {}", serde_json::to_string(&Inches(1)).unwrap());
        println!("Instance = {}", serde_json::to_string(&Instance).unwrap());
    }
}
```

#### （5）序列化枚举

```rust
use serde::{Serialize, Serializer};
use serde::ser::{SerializeStruct, SerializeTupleStruct, SerializeStructVariant, SerializeTupleVariant};


#[allow(unused, dead_code)]
enum E {
    // 如下三步:
    //   1. serialize_struct_variant
    //   2. serialize_field
    //   3. end
    Color { r: u8, g: u8, b: u8 },

    // 如下三步:
    //   1. serialize_tuple_variant
    //   2. serialize_field
    //   3. end
    Point2D(f64, f64),

    // 使用 serialize_newtype_variant.
    Inches(u64),

    // 使用 serialize_unit_variant.
    Instance,
}

impl Serialize for E {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        match self {
            E::Color {r,g,b} => {
                let mut state = serializer.serialize_struct_variant("E", 0, "Color", 3)?;
                state.serialize_field("r", r)?;
                state.serialize_field("g", g)?;
                state.serialize_field("b", b)?;
                state.end()
            }
            E::Point2D(x, y) => {
                let mut state = serializer.serialize_tuple_variant("E", 1, "Point2D",  2)?;
                state.serialize_field(x)?;
                state.serialize_field(y)?;
                state.end()
            }
            E::Inches(i) => serializer.serialize_newtype_variant("E", 2, "Inches", i),
            E::Instance => serializer.serialize_unit_variant("E", 3, "Instance"),
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch04() {
        println!("E::Color = {}", serde_json::to_string(&E::Color{r:1, g:2, b:3}).unwrap());
        println!("E::Point2D = {}", serde_json::to_string(&E::Point2D(1.0, 2.0)).unwrap());
        println!("E::Inches = {}", serde_json::to_string(&E::Inches(1)).unwrap());
        println!("E::Instance = {}", serde_json::to_string(&E::Instance).unwrap());

    }
}
```

#### （6）序列化字节数组

由于 [RFC 1210 specialization](https://github.com/rust-lang/rust/issues/31844) 尚未稳定。

因此 `[u8]` 和 `Vec<u8>` 分别与 `[T]` 和 `Vec<T>` 重复产生定义。

因此 在 `serde` 库中，没有对 `serializer.serialize_bytes(self)` 的使用

因此 Serde 创建了一个专为 字节数组 实现 `Serialize` 类型 的 库 `serde_bytes`，以提高字节流的序列化/反序列化效率。

使用方式如下

`Cargo.toml`

```toml
serde_bytes = "0.11"
```

`src/ch04_custom_serde.rs`

```rust

#[derive(Serialize)]
struct Efficient<'a> {
    #[serde(with = "serde_bytes")]
    bytes: &'a [u8],
    byte_buf: Vec<u8>,
}

struct Efficient2<'a> {
    bytes: &'a [u8],
    byte_buf: Vec<u8>,
}

impl <'a> Serialize for Efficient2<'a> {
    fn serialize<S>(&self, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
            let mut state = serializer.serialize_struct("Efficient2", 2)?;
        state.serialize_field("bytes", {
            // 对 bytes 添加了一层包装，用来代理调用 `serde_bytes::serialize` 方法
            // 这样就能 使用高效的 字节数组序列化
            struct SerializeWith<'__a, 'a: '__a> {
                values: (&'__a &'a [u8],),
                phantom: serde::export::PhantomData<Efficient2<'a>>,
            }
            impl<'__a, 'a: '__a> serde::Serialize for SerializeWith<'__a, 'a> {
                fn serialize<__S>(
                    &self,
                    __s: __S,
                ) -> serde::export::Result<__S::Ok, __S::Error>
                where
                    __S: serde::Serializer,
                {
                    serde_bytes::serialize(self.values.0, __s)
                }
            }
            &SerializeWith {
                values: (&self.bytes,),
                phantom: serde::export::PhantomData::<Efficient2<'a>>,
            }
        })?;
        state.serialize_field("byte_buf", &self.byte_buf)?;
        state.end()
    }
}
```

#### （7）序列化Option

针对 `Option` 枚举 和 `#[derive(Serialize)]` 普通枚举的代码实现逻辑不同：

* 针对 `Some(value)` 使用 `serializer.serialize_some(value)` 进行序列化，在JSON中将返回 value 的序列化
* 针对 `None` 使用 `serializer.serialize_none()` 进行序列化，在JSON中将返回null

### 2、实现 Deserialize

本节参考源码：

* [`serde/src/de/mod.rs`](https://github.com/serde-rs/serde/blob/master/serde/src/de/mod.rs)

特质声明如下

```rust
pub trait Deserialize<'de>: Sized {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>;
}
```

该方法的工作是：

* 通过调用给定 `Deserializer` 参数上的一种方法，并传递一个实现了 `Visitor` 的实例

#### （1）反序列化基础数据类型

```rust
use std::fmt;

use serde::de::{self, Visitor};

struct I32Visitor;

impl<'de> Visitor<'de> for I32Visitor {
    type Value = i32;

    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
        formatter.write_str("an integer between -2^31 and 2^31")
    }

    fn visit_i8<E>(self, value: i8) -> Result<Self::Value, E>
    where
        E: de::Error,
    {
        Ok(i32::from(value))
    }

    fn visit_i32<E>(self, value: i32) -> Result<Self::Value, E>
    where
        E: de::Error,
    {
        Ok(value)
    }

    fn visit_i64<E>(self, value: i64) -> Result<Self::Value, E>
    where
        E: de::Error,
    {
        use std::i32;
        if value >= i64::from(i32::MIN) && value <= i64::from(i32::MAX) {
            Ok(value as i32)
        } else {
            Err(E::custom(format!("i32 out of range: {}", value)))
        }
    }

    // Similar for other methods:
    //   - visit_i16
    //   - visit_u8
    //   - visit_u16
    //   - visit_u32
    //   - visit_u64
}

impl<'de> Deserialize<'de> for i32 {
    fn deserialize<D>(deserializer: D) -> Result<i32, D::Error>
    where
        D: Deserializer<'de>,
    {
        deserializer.deserialize_i32(I32Visitor)
    }
}
```

此处仅为示例，Serde 已为所有基础数据类型实现了类似的实现（以上代码已经在 用户 crate 无法编译，因为不满足孤儿规则）

需要注意的是

* Visitor特质还有许多方法未实现的方法，如果反序列化协议提供商调用这些方法，将返回类型错误。
* 例如，I32Visitor没有实现 `Visitor::visit_map`，因此在输入包含map时尝试反序列化i32是类型错误。
* 反序列化器不一定会跟随类型提示，因此对deserialize_i32的调用不一定意味着反序列化器将调用I32Visitor :: visit_i32。（因为不一定序列化协议都支持如此之多的数据类型）例如，JSON将所有带符号的整数类型都视为相同。JSON反序列化器将为任何带符号的整数调用visit_i64，为任何无符号的整数调用visit_u64，即使提示使用其他类型。

#### （2）反序列化Map

```rust

struct MyMap<K, V>(PhantomData<K>, PhantomData<V>);


impl<K, V> MyMap<K, V> {
    fn with_capacity(c: usize) -> Self {
        println!("build MyMap size = {}", c);
        MyMap(PhantomData, PhantomData)
    }

    fn insert(&mut self, _: K, _: V) {
        println!("call MyMap insert")
    }
}

// Visitor 决定不同类型的数据的如何处理，其方法包含默认为报错，调用哪个方法取决于
// Deserializer 的实现和序列化数据流解析
// 针对 Map 类型的 Visitor 需要提供 <K, V> 的泛型，因此需要使用 Deserializer
// PhantomData 用于在零尺寸类型中使用泛型
struct MyMapVisitor<K, V> {
    marker: PhantomData<fn() -> MyMap<K, V>>
}

impl<K, V> MyMapVisitor<K, V> {
    fn new() -> Self {
        MyMapVisitor {
            marker: PhantomData
        }
    }
}

impl<'de, K, V> Visitor<'de> for MyMapVisitor<K, V>
where
    K: Deserialize<'de>,
    V: Deserialize<'de>,
{
    // 表示 这个 Visitor 的产出类型
    type Value = MyMap<K, V>;

    // 格式化一条消息，说明该访客希望接收哪些数据。
    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
        formatter.write_str("a very special map")
    }

    // 将 Map 数据 MyMap
    fn visit_map<M>(self, mut access: M) -> Result<Self::Value, M::Error>
    where
        M: MapAccess<'de>,
    {
        let mut map = MyMap::with_capacity(access.size_hint().unwrap_or(0));

        // 遍历数据
        while let Some((key, value)) = access.next_entry()? {
            map.insert(key, value);
        }

        Ok(map)
    }
}

// 表示 MyMap 类型可以被反序列化
impl<'de, K, V> Deserialize<'de> for MyMap<K, V>
where
    K: Deserialize<'de>,
    V: Deserialize<'de>,
{
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        // 实例化访问者，并要求反序列化器将其驱动到输入数据上，从而生成MyMap实例。
        deserializer.deserialize_map(MyMapVisitor::new())
    }
}


#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch04() {
        // 反序列化
        let json1 = r#"
        {
            "k1": "v1",
            "k2": "v2"
        }
        "#;
        let _r1: MyMap<String, String> = serde_json::from_str(json1).unwrap();
    }
}
```

#### （3）反序列化结构

```rust
use std::fmt;

use serde::de::{self, Deserialize, Deserializer, Visitor, SeqAccess, MapAccess};

#[derive(Debug)]
struct Duration {
    secs: u64,
    nanos: u32,
}

impl Duration {
    fn new(secs: u64, nanos: u32) -> Duration {
        Duration {
            secs,
            nanos
        }
    }
}

impl<'de> Deserialize<'de> for Duration {
    fn deserialize<D>(deserializer: D) -> Result<Self, D::Error>
    where
        D: Deserializer<'de>,
    {
        enum Field { Secs, Nanos };

        // 这部分也可以如下写法生成，用于解析结构体的Key
        //
        //    #[derive(Deserialize)]
        //    #[serde(field_identifier, rename_all = "lowercase")]
        //    enum Field { Secs, Nanos }
        impl<'de> Deserialize<'de> for Field {
            fn deserialize<D>(deserializer: D) -> Result<Field, D::Error>
            where
                D: Deserializer<'de>,
            {
                struct FieldVisitor;

                impl<'de> Visitor<'de> for FieldVisitor {
                    type Value = Field;

                    fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                        formatter.write_str("`secs` or `nanos`")
                    }

                    fn visit_str<E>(self, value: &str) -> Result<Field, E>
                    where
                        E: de::Error,
                    {
                        match value {
                            "secs" => Ok(Field::Secs),
                            "nanos" => Ok(Field::Nanos),
                            _ => Err(de::Error::unknown_field(value, FIELDS)),
                        }
                    }
                }

                deserializer.deserialize_identifier(FieldVisitor)
            }
        }

        struct DurationVisitor;

        impl<'de> Visitor<'de> for DurationVisitor {
            type Value = Duration;

            fn expecting(&self, formatter: &mut fmt::Formatter) -> fmt::Result {
                formatter.write_str("struct Duration")
            }

            fn visit_seq<V>(self, mut seq: V) -> Result<Duration, V::Error>
            where
                V: SeqAccess<'de>,
            {
                let secs = seq.next_element()?
                    .ok_or_else(|| de::Error::invalid_length(0, &self))?;
                let nanos = seq.next_element()?
                    .ok_or_else(|| de::Error::invalid_length(1, &self))?;
                Ok(Duration::new(secs, nanos))
            }

            fn visit_map<V>(self, mut map: V) -> Result<Duration, V::Error>
            where
                V: MapAccess<'de>,
            {
                let mut secs = None;
                let mut nanos = None;
                // next_key 方法将调用 Field 类型的 deserialize 方法
                while let Some(key) = map.next_key::<Field>()? {
                    match key {
                        Field::Secs => {
                            if secs.is_some() {
                                return Err(de::Error::duplicate_field("secs"));
                            }
                            secs = Some(map.next_value()?);
                        }
                        Field::Nanos => {
                            if nanos.is_some() {
                                return Err(de::Error::duplicate_field("nanos"));
                            }
                            nanos = Some(map.next_value()?);
                        }
                    }
                }
                let secs = secs.ok_or_else(|| de::Error::missing_field("secs"))?;
                let nanos = nanos.ok_or_else(|| de::Error::missing_field("nanos"))?;
                Ok(Duration::new(secs, nanos))
            }
        }

        // 调用 deserialize_struct 传递 Visitor
        const FIELDS: &'static [&'static str] = &["secs", "nanos"];
        deserializer.deserialize_struct("Duration", FIELDS, DurationVisitor)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ch04() {

        // 反序列化
        let json2 = r#"
        {
            "secs": 1,
            "nanos": 2
        }
        "#;
        let r2: Duration = serde_json::from_str(json2).unwrap();
        println!("Duration = {:?}", r2);
    }
}
```

### 3、单元测试

参见 https://serde.rs/unit-testing.html

## 五、实现序列化器/反序列化器（Serializer/Deserializer）

参见：https://serde.rs/data-format.html

## 六、反序列化器生命周期

参见：https://serde.rs/lifetimes.html

## 七、no-std 支持

参见：https://serde.rs/no-std.html

## 八、feature 列表

参见：https://serde.rs/feature-flags.html
