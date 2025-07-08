# Chrono: Rust 日期和时间库

Chrono旨在提供在公历格里高利历中执行正确日期和时间操作所需的所有功能：

- `DateTime`类型默认是时区感知的，同时提供单独的时区无关类型。
- 可能产生无效或模糊日期和时间的操作会返回`Option`或`MappedLocalTime`。
- 可配置的解析和格式化，具有受strftime启发的日期和时间格式化语法。
- `Local`时区可以与操作系统的当前时区一起工作。
- 类型和操作的实现考虑了合理的效率。
- 为了限制二进制大小，Chrono默认不附带时区数据。使用配套的crate `Chrono-TZ`或`tzfile`获取完整的时区支持。

## 功能特性

Chrono支持各种运行时环境和操作系统，并具有几个可启用或禁用的功能。

### 默认功能：

- `alloc`：启用依赖于内存分配的功能（主要是字符串格式化）。
- `std`：启用依赖于标准库的功能。这是`alloc`的超集，增加了与标准库类型和特征的互操作。
- `clock`：启用读取本地时区（`Local`）的功能。这是`now`的超集。
- `now`：启用读取系统时间（`now`）的功能。
- `wasmbind`：为wasm32目标提供与JS Date API的接口。

### 可选功能：

- `serde`：通过serde启用序列化/反序列化。
- `rkyv`：已弃用，使用`rkyv-*`功能。
- `rkyv-16`、`rkyv-32`、`rkyv-64`：通过rkyv启用序列化/反序列化，分别使用16位、32位或64位整数。
- `rkyv-validation`：使用bytecheck启用rkyv验证支持。
- `arbitrary`：使用Arbitrary crate构造类型的任意实例。
- `unstable-locales`：启用本地化。这添加了带有`_localized`后缀的各种方法。

## 概述

### 时间差/持续时间

Chrono提供了`TimeDelta`类型来表示时间跨度的大小。这是一个以秒和纳秒表示的"精确"持续时间，不表示如天或月等"名义"组件。

`TimeDelta`类型以前命名为`Duration`（仍作为类型别名提供）。与类似的`core::time::Duration`的一个显著区别是它是有符号值而不是无符号值。

### 日期和时间

Chrono提供了`DateTime`类型来表示时区中的日期和时间。

`DateTime`是时区感知的，必须从`TimeZone`对象构造，该对象定义了本地日期如何转换为UTC日期并返回。有三个众所周知的`TimeZone`实现：

- `Utc`指定UTC时区。它最高效。
- `Local`指定系统本地时区。
- `FixedOffset`指定任意固定时区，如UTC+09:00或UTC-10:30。

不同`TimeZone`类型的`DateTime`是不同的，不能混用，但可以使用`DateTime::with_timezone`方法相互转换。

您可以在UTC时区（`Utc::now()`）或本地时区（`Local::now()`）获取当前日期和时间。

```rust
use chrono::prelude::*;

let utc: DateTime<Utc> = Utc::now(); // 例如 `2014-11-28T12:45:59.324310806Z`
let local: DateTime<Local> = Local::now(); // 例如 `2014-11-28T21:45:59.324310806+09:00`
```

此外，您还可以创建自己的日期和时间：

```rust
use chrono::offset::MappedLocalTime;
use chrono::prelude::*;

let dt = Utc.with_ymd_and_hms(2014, 7, 8, 9, 10, 11).unwrap(); // `2014-07-08T09:10:11Z`
```

### 格式化和解析

格式化通过`format`方法完成，其格式等同于熟悉的strftime格式。

默认的`to_string`方法和`{:?}`说明符也提供了合理的表示。Chrono还提供了`to_rfc2822`和`to_rfc3339`方法用于常见格式。

Chrono现在还提供几乎任何语言的日期格式化功能，无需额外的C库。此功能在`unstable-locales`特性下提供：

```rust
use chrono::prelude::*;

let dt = Utc.with_ymd_and_hms(2014, 11, 28, 12, 0, 9).unwrap();
assert_eq!(dt.format("%Y-%m-%d %H:%M:%S").to_string(), "2014-11-28 12:00:09");
assert_eq!(dt.format_localized("%A %e %B %Y, %T", Locale::fr_BE).to_string(), 
           "vendredi 28 novembre 2014, 12:00:09");
```

解析可以通过两种方法完成：

1. 标准的`FromStr`特性（以及字符串上的`parse`方法）可用于解析`DateTime<FixedOffset>`、`DateTime<Utc>`和`DateTime<Local>`值。
2. `DateTime::parse_from_str`解析带偏移量的日期和时间，并返回`DateTime<FixedOffset>`。

```rust
use chrono::prelude::*;

let dt = "2014-11-28T12:00:09Z".parse::<DateTime<Utc>>().unwrap();
let fixed_dt = DateTime::parse_from_str("2014-11-28 21:00:09 +09:00", "%Y-%m-%d %H:%M:%S %z").unwrap();
```

### 与EPOCH时间戳的转换

使用`DateTime::from_timestamp(seconds, nanoseconds)`从UNIX时间戳构建`DateTime<Utc>`。

使用`DateTime.timestamp`从`DateTime`获取时间戳（以秒为单位）。此外，您可以使用`DateTime.timestamp_subsec_nanos`获取额外的纳秒数。

```rust
use chrono::{DateTime, Utc};

// 从epoch构建datetime：
let dt: DateTime<Utc> = DateTime::from_timestamp(1_500_000_000, 0).unwrap();
assert_eq!(dt.to_rfc2822(), "Fri, 14 Jul 2017 02:40:00 +0000");

// 从datetime获取epoch值：
let dt = DateTime::parse_from_rfc2822("Fri, 14 Jul 2017 02:40:00 +0000").unwrap();
assert_eq!(dt.timestamp(), 1_500_000_000);
```

## 限制

- 仅支持向前推算的格里高利历（即扩展以支持更早的日期）。
- 日期类型限制为从公元纪元前后约262,000年。
- 时间类型限制为纳秒精度。
- 可以表示闰秒，但Chrono不完全支持它们。