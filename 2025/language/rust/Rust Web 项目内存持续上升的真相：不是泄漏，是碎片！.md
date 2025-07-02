最近，有开发者在使用 Axum 框架构建 Web 服务时遇到这样一个问题：服务运行一段时间后，内存使用不断上升，最后 OOM 被系统杀死，重启后依旧如此，形成了循环。

起初怀疑是代码中出现了内存泄漏，经过多次排查和 profiling 分析，未发现任何明显问题。使用的都是 Rust 安全代码，没有 `Rc` 循环，也未使用 `mem::forget()` 等容易导致泄漏的操作。

进一步调查后发现，问题可能并非出在业务逻辑，而是在于：**默认的内存分配器可能不适合这个场景**

## Rust 的默认分配器适合哪些程序？

Rust 默认使用的是“系统自带”的内存分配器，比如在 Linux 上就是 **glibc 的 malloc**。这种分配器具有以下特点：

- 体积小、依赖少
- 适用于生命周期短、结构简单的程序
- 不适合长时间运行、频繁分配释放、多线程的场景

而现代 Web 服务通常具备以下特征：

- 使用 Tokio 实现异步 + 多线程
- 长时间运行，7x24 不间断
- 每次请求涉及大量堆内存分配与释放
- 运行在容器或 Kubernetes 中，有内存上限

此时继续使用系统分配器，容易导致性能下降或内存异常增长。



## 是内存泄漏吗？可能是“碎片”

这类问题很可能不是程序真的“泄漏”内存，而是出现了**内存碎片（fragmentation）** 。表现是内存没有被释放，但实际上程序仍然能访问，只是系统无法有效重用它。

比如：

- 多次分配释放不同大小的内存块
- 导致大块空闲内存被分散，无法再次分配
- 分配器无法整理这些碎片，久而久之内存持续上升

这就像抽屉里塞满了形状各异的物品，虽然有空隙，但再也塞不下新东西。

## 换个分配器能解决问题吗？

许多开发者建议，在这种场景下换用专为高性能设计的内存分配器，比如 mimalloc 或 jemalloc，能带来明显改善。

实际案例显示：

- 一台拥有 **192GB RAM** 的服务器在使用默认分配器时依然 OOM
- 切换到 mimalloc 后，内存使用下降至 **18GB**
- 在某些应用中，mimalloc 不仅降低内存占用，还能 **成倍提升性能**

------

## 如何切换分配器？一行代码搞定

以 mimalloc 为例：

```toml
# Cargo.toml
[dependencies]
mimalloc = "0.1"

# main.rs
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

jemalloc 则类似：

```toml
# Cargo.toml
jemallocator = "0.3"

# main.rs
#[global_allocator]
static GLOBAL: jemallocator::Jemalloc = jemallocator::Jemalloc;
```

切换后，整个程序所有的堆内存分配操作都会使用新的分配器。

## mimalloc 和 jemalloc 选哪个？

| 分配器   | 特点                                                 |
| -------- | ---------------------------------------------------- |
| mimalloc | 微软出品，轻量级，低碎片率，适合高并发服务           |
| jemalloc | 老牌稳定，性能好，最坏情况也能控制内存增长，体积稍大 |

如果没有特别要求，建议优先选择 **mimalloc**，更容易上手，表现稳定。



## 一些建议

除了更换分配器，还可以考虑以下方式优化内存使用：

- 启用 leak sanitizer 检查真正的内存泄漏
  RUSTFLAGS="-Z sanitizer=leak -Zexport-executable-symbols" cargo test --target x86_64-unknown-linux-gnu
- 使用 对象池（object pool）重用内存，避免重复分配大对象，例如复用 Vec<u8>
- 如果仍使用系统分配器，可尝试 `malloc_trim(0)` 主动释放空闲内存（仅适用于 glibc）
- 避免 `tracing-subscriber` 等日志库中缓存无限增长



## 粽结一下吧

在如下场景中，默认分配器可能会成为瓶颈：

- Tokio 驱动的多线程 Web 服务
- 长时间运行的高并发程序
- 内存分配/释放频繁，负载不稳定
- 运行在限制内存的容器中（如 Kubernetes）

此时，切换到 mimalloc 或 jemalloc 是一个值得尝试的优化手段。Rust 本身提供了极高的控制力，一行代码就可以改变分配行为，让程序更快、更稳、内存更可控。

不要等到 OOM 才开始排查，“选对分配器”从一开始就能省去许多麻烦。