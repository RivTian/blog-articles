# Jemallocator: Rust jemalloc 内存分配器

> [!TIP]
>
> 默认分配器，有时会出现不能及时释放内存，推荐使用jemallocator

> [!NOTE]  
> Rust 内存分配器替代品.


jemallocator 是一个与 jemalloc 内存分配器链接的库，提供了 `Jemalloc` 单元类型，该类型实现了分配器 API 并可以设置为 `#[global_allocator]`。

`tikv-jemallocator` 是 `jemallocator` 的后继项目。这两个 crate 除了名称不同外完全相同。对于新项目，建议使用 `tikv-xxx` 版本。

## jemalloc 生态系统

jemalloc 支持生态系统由以下 crate 组成：

- **tikv-jemalloc-sys**：构建并链接到 jemalloc，暴露对它的原始 C 绑定。
- **tikv-jemallocator**：提供实现了 `GlobalAlloc` 和 `Alloc` trait 的 `Jemalloc` 类型。
- **tikv-jemalloc-ctl**：jemalloc 控制和自省 API 的高级封装（`mallctl*()` 函数族和 `MALLCTL NAMESPACE`）。

## 使用方法

### 添加依赖

要使用 tikv-jemallocator，将它添加为依赖：

```toml
[dependencies]

[target.'cfg(not(target_env = "msvc"))'.dependencies]
tikv-jemallocator = "0.5"
```

### 设置为全局分配器

要将 `tikv_jemallocator::Jemalloc` 设置为全局分配器，将以下代码添加到您的项目中：

```rust
#[cfg(not(target_env = "msvc"))]
use tikv_jemallocator::Jemalloc;

#[cfg(not(target_env = "msvc"))]
#[global_allocator]
static GLOBAL: Jemalloc = Jemalloc;
```

就这样！一旦定义了这个静态变量，jemalloc 将用于同一程序中 Rust 代码请求的所有内存分配。

## 优势

jemalloc 是一个通用目的的 malloc 实现，专注于：

- 减少内存碎片
- 高并发场景下的可扩展性
- 提供丰富的内省和控制能力

在以下场景中特别有用：
- 长时间运行的应用程序
- 内存密集型工作负载
- 需要细粒度内存管理的高性能服务

## 兼容性说明

jemalloc 不支持 MSVC 目标环境，因此在 Windows 上使用 MSVC 工具链时不可用。这就是为什么在示例代码中包括 `cfg(not(target_env = "msvc"))` 条件的原因。