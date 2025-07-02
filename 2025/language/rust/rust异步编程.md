> 版本：1.41.0
> 前序文章: [rust 语言](./rust语言.md)

[流行异步库](https://lib.rs/asynchronous)

参考：

* [Rust 异步编程（中文版有点滞后）](https://learnku.com/docs/async-book/2018/streams/4797)
* [Asynchronous Programming in Rust](https://rust-lang.github.io/async-book/)
* [Rust异步入门](https://omarabid.com/async-rust)
* [Designing futures for Rust](https://aturon.github.io/blog/2016/09/07/futures-design/)
* [零成本异步I/O](https://zhuanlan.zhihu.com/p/97574385)

类似于ES6（JavaScript）中的 `async` 和 `await`

创建测试项目 `cargo new async_await`

## 一、入门

### 1、发展状态

里程碑: 1.39.0 async/.await 进入稳定版 （Future 版本 0.3）

目前（2020-02-01），大多数异步类库任然使用 Future 0.1 版本的API，此外存在如下问题

* `async fn` 目前仍不知在 trait 中使用
* 编译错误信息难以理解

### 2、async/.await示例

定义一个 异步函数 `async fn fn_name () {}`

```rust
// `block_on`会阻塞当前线程，直到提供的future完成为止。
// 其他执行器提供更复杂的行为，例如将多个期货调度到同一线程上。
use futures::executor::block_on;

async fn hello_world() {
    println!("hello, world!");
}

    let future = hello_world(); // Nothing is printed
    println!("调用了异步函数 hello_world，此句应该先打印");

    block_on(future);
```

需要添加如下依赖

```toml
futures = "0.3.1"
```

使用 `.await` 等待其他Future执行完成，以控制执行顺序

```rust
use futures::executor::block_on;
use async_std::task;

[derive(Debug)]
struct Song;

async fn learn_song() -> Song {
    println!("学习唱歌");
    // 模拟IO阻塞等
    task::sleep(time::Duration::from_secs(1)).await; // 不能使用 thread::sleep
    Song
}
async fn sing_song(song: Song) {
    println!("唱歌中... {:?}", song);
    task::sleep(time::Duration::from_secs(1)).await;
}
async fn dance() {
    println!("跳舞中...")
}

// 并行任务1：
async fn learn_and_sing() {
    // 在唱歌之前等待学歌完成
    // 这里我们使用 `.await` 而不是 `block_on` 来防止阻塞线程，这样就可以同时执行 `dance` 了。
    let song = learn_song().await;
    sing_song(song).await;
}

async fn async_main() {
    let f1 = learn_and_sing();
    let f2 = dance();
     // `join!` 类似于 `.await` ，但是可以等待多个 future 并发完成
     // 如果学歌的时候有了短暂的阻塞，跳舞将会接管当前的线程，如果跳舞变成了阻塞
     // 学歌将会返回来接管线程。如果两个futures都是阻塞的，
     // 这个‘async_main'函数就会变成阻塞状态，并生成一个执行器
    futures::join!(f1, f2); // f1, f2 并行完成
}

    println!("===顺序执行===");
    let song = block_on(learn_song());
    block_on(sing_song(song));
    block_on(dance());

    println!("===并行执行===");
    block_on(async_main());
```

需要添加如下依赖

```toml
futures = "0.3.1"
async-std = "1.4.0"
```

### 3、实例简单的异步HTTPServer

> 参考：https://hyper.rs/

添加依赖

```toml
hyper = "0.13"
tokio = { version = "0.2", features = ["full"] }
```

编写代码

```rust
use std::{convert::Infallible, net::SocketAddr};
use hyper::{Body, Request, Response, Server};
use hyper::service::{make_service_fn, service_fn};

async fn handle(_: Request<Body>) -> Result<Response<Body>, Infallible> {
    Ok(Response::new("Hello, World!".into()))
}

async fn run_server(addr: SocketAddr) {
    println!("Listening on http://{}", addr);

    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle))
    });

    // 创建绑定到提供的地址的服务器
    let serve_future = Server::bind(&addr)
        .serve(make_svc);
     // 等待服务完成服务或者因为某个错误而退出
    // 如果一个错误出现，输出它到stderr
    if let Err(e) = serve_future.await {
        eprintln!("server error: {}", e);
    }
}

#[tokio::main]
async fn main() {
    println!("===启动一个HttpServer===");
    run_server(SocketAddr::from(([127, 0, 0, 1], 3000))).await;
}
```

## 二、核心 trait 与 基本原理

### 1、Future

Rust 实现异步函数的核心特质为 `std::future::Future`，定义如下

```rust
use crate::marker::Unpin;
use crate::ops;
use crate::pin::Pin;
use crate::task::{Context, Poll};

pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

impl<F: ?Sized + Future + Unpin> Future for &mut F { // F 的可变引用实现 Future
    type Output = F::Output;

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        F::poll(Pin::new(&mut **self), cx)
    }
}

impl<P> Future for Pin<P> // 为 Pin<P=Unpin + ops::DerefMut<Target: Future>> 实现Future
where
    P: Unpin + ops::DerefMut<Target: Future>,
{
    type Output = <<P as ops::Deref>::Target as Future>::Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        Pin::get_mut(self).as_mut().poll(cx)
    }
}
```

观察 Future 特质，包含一个核心函数，poll。该函数传递一个 `&mut Context<'_>` 类型参数， 返回一个 `Poll` 类型参数

* Context 主要包含一个 `Waker` 对象，由执行器提供，用于告诉执行器，重新执行当前 `poll` 函数
* Poll 是一个枚举类型包含两个枚举
    * `Ready<Output>` 当任务已经就绪，返回该对象
    * `Pending` 任务没有就绪时返回该对象，此Future将让出CPU，直到在其他线程或者任务执行调用`Waker`为止
* 实现者需要保证 poll 是非阻塞，如果是阻塞的话会导致循环进行不下去

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

pub enum Poll<T> {
    Ready(#[stable(feature = "futures_api", since = "1.36.0")] T),
    Pending,
}

pub struct Context<'a> {
    waker: &'a Waker,
    _marker: PhantomData<fn(&'a ()) -> &'a ()>,
}
```

实现一个 Future 类型的方式

* 方式1：使用 `async fn`，编译器会自动生成实现 Future 特质的类型
* 方式2：自定义 结构体，并实现 Future 特质

可以看出 Rust 的异步实现并不是基于回调函数，而是基于 Future 的组合，从而简化设计提高并行度

#### Tip

tip1. 关于 `mut self`

```rust
trait Foo {
    fn m(self);
}

#[derive(Debug)]
struct Bar{
    pub val: i32,
}

impl Foo for Bar{
    fn m(mut self) {
        println!("{:x}", &self as *const Bar as usize);
        println!("Bar.val addr = {:x}", &self.val as *const i32 as usize);
        self.val = 1;
        println!("1");
    }
}

fn main() {
    let b = Bar {val: 2};
    println!("Bar addr = {:x}", &b as *const Bar as usize);
    println!("Bar.val addr = {:x}", &b.val as *const i32 as usize);
    b.m();
}
```

以上代码不报任何错误，说明没有任何问题，且地址都发生了变化，说明：

* `fn m(mut self) {` 等价于 `fn m(self) { let mut s = self;` 并不是说只能 `let mut` 的对象才能调用
    * 如果实现了Clone，则和C语言的结构体按值传递一致
    * 没有实现，也类似于C语言的按照值传递，但是传递后本身值不可用
* 说明 move 本质上是 先拷贝再将作用域内的变量设为不可用（编译器检查）（因为内存地址发生了变化）

### 2、手动实现Future类型

通过自定义类型的方式实现一个异步的sleep

思路

* 创建Future类型时，标记为未就绪，同时启动一个子线程，进行sleep
* 当子线程sleep结束后，将状态标记为就绪，并调用 `ctx.waker`
* `poll` 函数检测状态
    * 如果就绪，返回`Poll::Ready`
    * 如果未就绪，返回`Poll::Pending`

```rust
use std::future::Future;
use std::{time, thread};
use std::pin::Pin;
use std::task::{Waker, Poll, Context};
use std::sync::{Mutex, Arc};
use std::sync::mpsc::{sync_channel, SyncSender, Receiver};


// === 实现一个Future ===
// 实现一个异步的sleep类似于async_std::task:sleep
// 实现原理 创建一个新的线程，然后立即sleep，唤醒后，更改状态，唤醒轮训

pub struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

/// `future`与等待线程之间的共享状态
struct SharedState {
    /// 用于判断sleep的时间是不是已经过了
    completed: bool,

    /// 任务的唤醒者 `TimerFuture` 正在上面运行.
    /// 线程能够使用这个设置`completed = true`之后去调用
    /// `TimerFuture`的任务来唤醒, 观察 `completed = true`, 并前进
    waker: Option<Waker>,
}

impl Future for TimerFuture {
    type Output = ();
    // 轮询函数
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        // 观察睡眠时间是否完成
        println!("###sleep poll");
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            // 已经完成，就绪
            Poll::Ready(())
        } else {
            // 设置唤醒器，以便线程在计时器（timer）完成的时候可以唤醒当前任务
            // 确定 future已经再一次被轮询了，并且看`completed = true`.
            //
            // 这样做一次很诱人，而不是每次都重复克隆唤醒器。
            // 但是，`TimerFuture`可以在执行程序上的任务之间移动，（旧的地址失效了）
            // 这可能会导致过时的唤醒程序指向错误的任务，
            // 从而阻止`TimerFuture`正确唤醒。
            //
            // 注意：可以使用 `Waker::will_wake`这个函数来检查
            // 但是为了简单起见，我们忽略了这个。
            shared_state.waker = Some(cx.waker().clone()); // 相当于下方Task引用计数+1，导致Task不被销毁，从而导致SyncSender引用计数大于零，循环得以继续
            Poll::Pending
        }
    }
}

impl TimerFuture {
    /// 构造函数
    pub fn new(duration: time::Duration) -> Self {
        // 构造共享状态
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));

        // 创建新的线程
        let thread_shared_state = shared_state.clone();
        thread::spawn(move || {
            // 睡眠
            thread::sleep(duration);
            // 获得锁
            let mut shared_state = thread_shared_state.lock().unwrap();
            // 标记为已完成
            shared_state.completed = true;
            // 唤醒轮询
            if let Some(waker) = shared_state.waker.take() {
                waker.wake() // 将任务重新放到任务队列
            }
        });

        TimerFuture { shared_state }
    }
}

use futures::executor::block_on;

    {
        println!("===测试自定义Future===");
        block_on(async {
            println!("睡眠前!");
            // Wait for our timer future to complete after two seconds.
            TimerFuture::new(time::Duration::from_secs(2)).await;
            // for i in 0..2 {
            //     TimerFuture::new(time::Duration::from_secs(i)).await;
            // }
            println!("睡眠后!");
        });
    }
```

### 3、手动实现一个执行器

实现一个单线程的Future执行器

* 使用 `sync_channel` 作为 Future 唤醒 的通讯方式（不能使用同步通道，因为会阻塞）
* 实现一个Waker结构，本质上就是向 `sync_channel` 提交任务
* 实现一个提交 Future 的函数或者结构，该函数向 `sync_channel` 提交一个任务
* 实现一个开始运行 提交的顶级 Future 函数或者结构，该函数逻辑如下
    * 轮训 `sync_channel` 接收端，获取 Future，并调用其 `poll` 方法

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Waker, Poll, Context};
use futures::task::{ArcWake, waker_ref};
use std::sync::{Mutex, Arc};
use futures::future::FutureObj;
use std::sync::mpsc::{sync_channel, SyncSender, Receiver};


// === 实现一个Future执行器 ===

/// 执行器
struct Executor {
    ready_queue: Receiver<Arc<Task>>, // 任务接收端
}

/// 用于向执行器中推送任务
#[derive(Clone)]
struct Spawner {
    task_sender: SyncSender<Arc<Task>>, // 任务发送端
}

/// 任务
struct Task {
    // In-progress future that should be pushed to completion
    //
    // The `Mutex` is not necessary for correctness, since we only have
    // one thread executing tasks at once. However, `rustc` isn't smart
    // enough to know that `future` is only mutated from one thread,
    // so we use it in order to provide safety. A production executor would
    // not need this, and could use `UnsafeCell` instead.
    // FutureObj 对象
    future: Mutex<Option<FutureObj<'static, ()>>>,

    // Handle to spawn tasks onto the task queue
    // 任务发送者，用于将任务放到任务队列
    task_sender: SyncSender<Arc<Task>>,
}

impl Spawner {
    /// 将任务推送到队列
    fn spawn(&self, future: impl Future<Output = ()> + 'static + Send) {
        // 传递一个future，创建一个Task，并推送到任务队列
        let future_obj = FutureObj::new(Box::new(future));
        let task = Arc::new(Task {
            future: Mutex::new(Some(future_obj)),
            task_sender: self.task_sender.clone(),
        });
        self.task_sender.send(task).expect("too many tasks queued");
    }
}


impl ArcWake for Task {
    fn wake_by_ref(arc_self: &Arc<Self>) {
        // Implement `wake` by sending this task back onto the task channel
        // so that it will be polled again by the executor.
        let cloned = arc_self.clone();
        arc_self.task_sender.send(cloned).expect("too many tasks queued");
    }
}

impl Executor {
    fn run(&self) {
        while let Ok(task) = self.ready_queue.recv() {
            // Take the future, and if it has not yet completed (is still Some),
            // poll it in an attempt to complete it.
            // 从任务中拿出future，如果没有完成，将返回some
            let mut future_slot = task.future.lock().unwrap();
            println!("+++[exec] task.strong_count = {:?}", Arc::strong_count(&task));
            if let Some(mut future) = future_slot.take() { // 在此take出来了
                // Create a `LocalWaker` from the task itself
                // 创建一个本地的Waker，实际上调用的是Task的wake_by_ref
                let waker = waker_ref(&task); // 给Task
                let context = &mut Context::from_waker(&*waker);
                if let Poll::Pending = Pin::new(&mut future).poll(context) {
                    // 没有完成，放回到锁对象，等待wake后重新执行
                    println!("---[exec] task.strong_count = {:?}", Arc::strong_count(&task));
                    *future_slot = Some(future); // 需要放回到锁中
                }
            }
        }
    }
}

/// 创建执行器和任务推送者
fn new_executor_and_spawner() -> (Executor, Spawner) {
    // Maximum number of tasks to allow queueing in the channel at once.
    // This is just to make `sync_channel` happy, and wouldn't be present in
    // a real executor.
    // 消息队列最大尺寸
    const MAX_QUEUED_TASKS: usize = 10_000;
    let (task_sender, ready_queue) = sync_channel(MAX_QUEUED_TASKS);
    (Executor { ready_queue }, Spawner { task_sender})
}

    {
        println!("===测试自定义Executor===");
        let (executor, spawner) = new_executor_and_spawner();
        spawner.spawn(async {
            println!("howdy!");
            // Wait for our timer future to complete after two seconds.
            TimerFuture::new(time::Duration::from_secs(2)).await;
            // for i in 0..2 {
            //     TimerFuture::new(time::Duration::from_secs(i)).await;
            // }
            println!("done!");
        });
        // Drop the spawner so that our executor knows it is finished and won't
        // receive more incoming tasks to run.
        // 销毁发送端以可以结束
        drop(spawner);
        // 如何实现所有任务执行完毕，自动返回的呢？
        // 本质上是生命周期和Arc引用计数（Task持有一个SyncSender）
        // Task实现了ArcWake特质，std::sync::Arc<Task> 本质上 就是Future::poll函数Context中weak的实现
        // Future当处于等待的时候，weak.clone，让std::sync::Arc<Task>引用计数保持大于1，轮训继续进行
        // 当Future完成后，std::sync::Arc<Task>引用计数等于0，Task被销毁（在本例中就是睡眠线程执行完成，并销毁）
        // 从而SyncSender被销毁，
        // 当所有Task被销毁，SyncSender真正被销毁，导致Receiver返回Err
        // 从而run返回
        executor.run();
    }
```

### 4、async 代码块模拟

本小结探索如何通过 自定义 Future 实现类似 `async {}` 的效果，探索编译方式，模拟上一节的代码

```rust
async {
    println!("howdy!");
    TimerFuture::new(time::Duration::from_secs(2)).await;
    println!("done!");
}
```

模拟代码如下

```rust
    {
        // 以上代码段基本等价于如下内容
        // 不同在于子Future可能在poll过程中构造出来（因为需要上下文参数作为构造参数）
        struct AsyncFuture {
            // fut_one: FutureObj<'static, ()>,
            fut_one: TimerFuture,
            state: AsyncState,
        }
        enum AsyncState { // 记录状态信息，包含await使用到的参数状态信息
            Started,
            AwaitingFutOne,
            Done,
        }
        impl AsyncFuture {
            fn new(fut_one: TimerFuture) -> AsyncFuture {
                // futures_task::future_obj::FutureObj<'static, ()>
                AsyncFuture {
                    // fut_one: FutureObj::new(Box::new(fut_one)),
                    fut_one,
                    state: AsyncState::Started,
                }
            }
            // unsafe_pinned!(fut_one: TimerFuture); // 生成 fut_one 函数
        }
        impl Future for AsyncFuture {
            type Output = ();
            // 轮询函数
            fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
                loop {
                    match self.state {
                        AsyncState::Started => {
                            println!("howdy!");
                            self.state = AsyncState::AwaitingFutOne;
                        },
                        AsyncState::AwaitingFutOne => match Pin::new(&mut self.fut_one).poll(cx) {
                            Poll::Ready(()) => self.state = AsyncState::Done,
                            Poll::Pending => return Poll::Pending,
                        },
                        AsyncState::Done => {
                            println!("done!");
                            return Poll::Ready(())
                        },
                    }
                }
            }
        }
        println!("");
        let (executor, spawner) = new_executor_and_spawner();
        spawner.spawn(AsyncFuture::new(TimerFuture::new(time::Duration::from_secs(2))));
        drop(spawner);
        executor.run();
    }
```

### 5、执行器和系统IO

利用 操作系统系统提供的类似 epoll 的非阻塞IO，配合 Future 即可实现 非阻塞异步IO。合理的方式，最少需要两个线程

* IO线程，负责epollIO，当某个IO就绪后，唤醒IOFuture
* 工作线程，包含类Future
    * IOFuture 被唤醒后，遍历所有事件，调用事件中注册的workFuture的waker.work
    * workFuture，处理Event，需在调用执行器进行注册，注册时机有两种
        * 在执行器启动之前，手动提交
        * workFuture中调用执行器提供的异步方法

Rust 生态中有两个 异步执行器 分别为 [tokio](https://github.com/tokio-rs/tokio) 和 [async-std](https://async.rs/)

### 6、个人体会

* Future 设计的尽量精简，保证与Executor耦合在标准库层尽量小，保证足够高的性能
* Future 这种设计思路，是将异步操作转换为状态机（就是If-Else）（`pull`），而不是回调（`push`）
    * 因此新的异步任务的提交需要运行时环境的支持
    * 这种设计可以做到超级小的运行时，几乎是零成本的抽象
* 标准库只提供 Future 和相关抽象，不提供 Executor，Executor由第三方库提供（如 `futures`）
* Executor 执行的最小单位是 一个顶级Future（对应一个Task），一个顶级Future可以包含其他Future，非顶级Future不可以独立运行（比如A包含B，则 AB 在一个Task中，是Executor执行的最小单位（类似于一个线程））
* 如果想实现类似NodeJS这种类型的操作，需要Executor提供环境支持

和 JavaScript 不太一样，测试代码如下：

```rust
use futures::executor::block_on;
    block_on(async {
        dance(); // 和JavaScript不一样，这不会执行，如果想让他执行必须使用Executor上下文环境提供的提交Future功能
        learn_song().await;
    });
```

## 三、async/.await

`async` 块生命周期

```rust
// 这个函数
async fn foo2(x: &u8) -> u8 { *x }

// 等价于如下函数
fn foo2_expanded<'a>(x: &'a u8) -> impl Future<Output = u8> + 'a {
    async move { *x }
}
```

async 使用外部引用变量生命周期问题

```rust
async fn borrow_x(x: &u8) -> u8 { *x }

// fn bad() -> impl Future<Output = u8> {
//     let x = 5;
//     borrow_x(&x) // ERROR: `x` does not live long enough
// }

fn good() -> impl Future<Output = u8> {
    async {
        let x = 5;
        borrow_x(&x).await
    }
}

// 等价于
async fn good1() -> u8 {
    let x = 5;
    borrow_x(&x).await
}
```

async 块访问周围变量

* 多个不同的`async`块可以访问相同的局部变量
* 只要它们在变量的范围内执行，为什么可以这么做：
    * 因为不可能发送竞争，同一个Future内部的所有代码在同一时刻只可能在同一个线程中执行，不可能出现
    * 也就是说rust保证 future_one 和 future_two 不会再多个线程中并行执行

```rust
async fn a(s: &str) -> i32 {
    println!("{}", s);
    1
}

/// `async`块：
/// 多个不同的`async`块可以访问相同的局部变量
/// 只要它们在变量的范围内执行
/// 为什么可以这么做：因为不可能发送竞争，同一个Future内部的所有代码在同一时刻只可能在同一个线程中执行，不可能出现
/// 也就是说rust保证 future_one 和 future_two 不会再多个线程中并行执行
async fn blocks() {
    let my_string = "foo".to_string();

    let ar = a(&my_string).await;

    let future_one = async {
        // ...
        println!("{}", my_string);
    };

    let future_two = async {
        // ...
        println!("{}", my_string);
    };

    // Run both futures to completion, printing "foo" twice:
    // 等待两个future以完成操作，两次打印“foo”：
    let ((), ()) = futures::join!(future_one, future_two);
}
```

`async move` 块 所有权转移

```rust
/// `async move`块：
///
/// 捕获变量所有权（move）
fn move_block() -> impl Future<Output = ()> {
    let my_string = "foo".to_string();
    let f = async move {
        // ...
        println!("{}", my_string);
    };
    // println!("{}", my_string); // Error borrow of moved value: `my_string`
    f
}
```

## 四、Pin

> 参考： https://doc.rust-lang.org/beta/std/pin/index.html
> https://rust.cc/article?id=4479f801-d28d-40cb-906c-85d8a04e8679
> https://rustforce.net/article?id=82a7e562-8bf1-45a2-9d86-fa3e6977039f

```rust
use crate::marker::Unpin;
use crate::ops;
use crate::pin::Pin;
use crate::task::{Context, Poll};

pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

默认情况下，Rust中所有类型都是可以 move 的，Rust允许按值传递所有类型，并且像 `Box<T>` 、`&mut T` 之类的智能指针或者引用允许你通过 `mem::swap` 进行拷贝交换（移动），这样，如果存在结构体存在自引用，将导致引用失效。

而 async 编译后的结构可能就会出现一种自引用的结构（我们自己写是不行的，编译器生成不受限制），如下所示：

```rust
async {
    let mut x = [0; 128];
    let read_into_buf_fut = read_into_buf(&mut x);
    read_into_buf_fut.await;
    println!("{:?}", x);

}

// 编译后的伪代码如下
// 这是 最外层的 async {}
struct AsyncFuture {
    x: [u8; 128],
    read_into_buf_fut: ReadIntoBuf<'what_lifetime?>,
}

// 这是 read_into_buf_fut 的Future
struct ReadIntoBuf<'a> {
    buf: &'a mut [u8], // points to `x` below
}
```

这样 AsyncFuture 构造出来后，就存在自引用（`AsyncFuture.read_into_buf_fut.buf` 指向 `AsyncFuture.x`）

此时在调用，如果 Future::poll 声明为
* `fn poll(self, ` 发生 move 导致 ``AsyncFuture.read_into_buf_fut.buf` 引用失效
* `fn poll(&self,` 用户可能使用，安全代码 `mem:swap` 将会导致引用失效，这破坏了Rust 内存安全性
* 因此 Rust 定义了 `Pin<T>` 类型，从协议层保证了用户不适用 `unsafe` 代码情况下内存安全

### 1、Pin的作用

Pin 类型包着指针类型，保证指针背后的值将不被移动。例如 `Pin<&mut T>`，`Pin<&T>`， `Pin<Box<T>>` 都保证 `T` 不会移动（move）。

### 2、Pin的原理

* 首先 `Pin<T>` 和 `Box<T>` 类似都是一种智能指针
* 不同点在于 `Pin<&mut T>` 不能拿到 `&mut T`，因此保证 `mem::swap` 无法调用
    * 因为 `Pin::as_mut` 返回的仍是 `Pin<T>`
    * 只有 `Pin<DerefMut<T: Unpin>>` 或者 `Pin<Deref<T: Unpin>>` 或者 `Pin<T: Unpin>` 可以通过 `get_mut` 或者 `get_ref` 拿到 `T` 的引用
    * `Pin::new` 只能是针对实现了 `Unpin` 的类型（重要）
* 本质上实现不移动就是加了一层指针，并未违反任意值都是可以移动的
    * 比如 `Pin<T>` 发生移动时，仅仅是 `Pin` 这个结构发生了移动，但是 `T` 对象并没有移动

**Tips: Unpin标记**

Unpin是一个标记 trait。所有的类型都实现了 Unpin 标记（因为标准库所有类型和我们自定义的类型都没有自引用，可以安全的Unpin）。而我们自己实现的类型并不会自动标记。该标记主要用于Pin类型中是否可以通过 `get_mut` 或者 `get_ref` 拿到引用，以解除 Pin。目前只有 `async` 生成的匿名 Future 类型没有实现该标记

比较绕的一个地方是 `Pin` 也是标准库的内容，`Pin` 也实现了 `Unpin`

### 3、Pin 与 Future

Pin 与 Future 结合可以保证我们不使用 unsafe 的情况下，无法写出有内存问题的代码。场景如下：

```rust
async {
    let mut x = [0; 128];
    let read_into_buf_fut = ReadIntoBuf::new(&mut x);
    read_into_buf_fut.await;
    println!("{:?}", x);
}
```

* `async` 调用了 我们自己写的 Future 类型，在本例中就是 `ReadIntoBuf`
* 此时 `async` 块，编译后，显然会构造出包含自引用的类型，且必然不会实现 `Unpin`
* 当我们再实现我们自己的 Future 类型 需要包含一个 Future 时，我们一般会这样实现
    * `struct BugInfect { sub: Box<dyn Future<Output = ()> + 'static + Send>,}` 使用 dyn，
    * 因为 没有实现 `Unpin` 此时在实现 `poll` 的时候，我们就无法 `Pin::new(self.sub);` 以调用 sub 的 poll 方法，
    * 只能使用 unsafe 代码块强制构造，通过 `Pin::new_unchecked(self.sub.as_mut())` 强制构造
    * 这样就保证我们的 safe 代码不会造成引用失效

全部测试代码如下

```rust
use std::future::Future;
use std::task::{Waker, Poll, Context};
use futures::executor::block_on; // 引入 futures 依赖

    async fn read_into_buf(x: &mut[i32; 32]){
        // TimerFuture::new(time::Duration::from_secs(1));
        x[0] = 1;
    }

    async fn test() {
        let mut x = [0; 32];
        let read_into_buf_fut = read_into_buf(&mut x);
        read_into_buf_fut.await;
        println!("{:?}", x);
    }

    struct BugInfect {
        sub: Box<dyn Future<Output = ()> + 'static + Send>,
    }

    impl Future for BugInfect {
        type Output = ();
        fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
            unsafe {
                match Pin::new_unchecked(self.sub.as_mut()).as_mut().poll(cx) {
                    Poll::Ready(()) => Poll::Ready(()),
                    Poll::Pending => return Poll::Pending,
                }
            }
        }
    }
    block_on(BugInfect { sub: Box::new(test()) });
```

## 五、Stream

Stream 类似于 Future ，但是在完成之前可以产生多个值，类似于标准库中的Iterator特性：

* 阻塞时返回 `Poll::Pending`
* 有数据时返回 `Poll::Ready<Some<Item>>`
* 流结束时返回 `Poll::Ready<None>`

该特质目前定义在 `features-core` 库下 （`futures::stream::Stream`）

```rust
pub trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>,
    ) -> Poll<Option<Self::Item>>;

    fn size_hint(&self) -> (usize, Option<usize>) {
        (0, None)
    }
}
```

### 1、Stream 使用

该方法主要实现在 features 类库下，一个使用上的例子，异步版本的 `channel`：

* `Receiver` 实现了 Stream
* `StreamExt` 提供了将 `Stream` 的一个数据转换为 一个 `Future` 的能力（通过 `next`）

```rust
async fn send_recv() {
    use futures::channel::{mpsc};
    use futures::stream::{StreamExt};
    use futures::sink::{SinkExt};

    const BUFFER_SIZE: usize = 10;
    let (mut tx, mut rx) = mpsc::channel::<i32>(BUFFER_SIZE);

    tx.send(1).await.unwrap(); // Sender 本质上实现了 `futures::sink::Sink`
    tx.send(2).await.unwrap();
    drop(tx);

    // 本质上调用 `StreamExt::next` 类似于 `Iterator::next`, 但是实际上返回一个
    // 实现了 `Future<Output = Option<T>>` 的类型
    assert_eq!(Some(1), rx.next().await);
    assert_eq!(Some(2), rx.next().await);
    assert_eq!(None, rx.next().await);
}
```

### 2、Stream与并发迭代

```rust
/// 对流求和
async fn sum_with_next(mut stream: Pin<&mut dyn Stream<Item = i32>>) -> i32 {
    let mut sum = 0;
    while let Some(item) = stream.next().await {
        sum += item;
    }
    sum
}

/// 使用 try_next （当流为返回为Result类型时）
async fn sum_with_try_next(
    mut stream: Pin<&mut dyn Stream<Item = Result<i32, io::Error>>>,
) -> Result<i32, io::Error> {
    use futures::stream::TryStreamExt; // for `try_next`
    let mut sum = 0;
    while let Some(item) = stream.try_next().await? {
        sum += item;
    }
    Ok(sum)
}

/// 并发执行流
async fn jump_around(
    mut stream: Pin<&mut dyn Stream<Item = Result<u8, io::Error>>>,
) -> Result<(), io::Error> {
    use futures::stream::TryStreamExt; // for `try_for_each_concurrent`
    const MAX_CONCURRENT_JUMPERS: usize = 100;

    // 100 个线程执行遍历
    stream.try_for_each_concurrent(MAX_CONCURRENT_JUMPERS, |item| async move {
        println!("{}", item);
        Ok(())
    }).await?;

    Ok(())
}
```

## 六、使用技巧和注意事项

### 1、同时处理多个Future

核心思路就是构建一个顶级 Future 在这个 Future中自己调用 Poll 方法。

**并行执行多个Future，等待全部执行完成**

使用 `futures::join` 宏，语义是发执行join的 Future ，当 Future 执行完成后，返回

```rust
use futures::join;

async fn get_book_and_music() -> (Book, Music) {
    let book_fut = get_book();
    let music_fut = get_music();
    join!(book_fut, music_fut)
}
```

当 `Future<Output=Result>` 时，建议使用 `try_join` 宏（因为其可以快速中断，只要有一个返回Err，则立即返回）

```rust
use futures::try_join;

async fn get_book() -> Result<Book, String> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book();
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}
```

快速返回的例子

```rust
use futures::{
    future::TryFutureExt,
    try_join,
};

async fn get_book() -> Result<Book, ()> { /* ... */ Ok(Book) }
async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }

async fn get_book_and_music() -> Result<(Book, Music), String> {
    let book_fut = get_book().map_err(|()| "Unable to get book".to_string());
    let music_fut = get_music();
    try_join!(book_fut, music_fut)
}
```

**并行执行多个Future，只要有一个Future完成，则立即做出响应，并返回**

> 参考： https://docs.rs/futures/0.3.1/futures/macro.select.html

`futures::select` 宏可以实现此功能（类似 Linux 的Select 或者 epoll）

`select` 宏 参数的语法为 `<pattern> = <expression> => <code>`

* `code` 返回值类型必须一致
* `expression` 如果是 `Future` 变量，则必须固定
    * 通过 `futures::pin_mut` 宏
    * 或者 `Box::pin()` 函数
* `expression` 是函数调用，则不需要固定，`select` 帮忙固定

```rust
        use futures::{
            future::FutureExt, // for `.fuse()`
            pin_mut,
            select,
        };
        use async_std::task;
        async fn task_one() { task::sleep(time::Duration::from_millis(1)).await }
        async fn task_two() { task::sleep(time::Duration::from_millis(2)).await }

        async fn race_tasks() {
            // fuse 将返回 FuseFuture，该 Future 包装一个子Future
            // 当子 Future 返回 Pending 时，返回Pending
            // 当子 Future 第一次返回 Ready 时，返回Ready
            // 此后 不会调用再调用 子Future，且永远返回Pending
            let t1 = task_one().fuse();
            let t2 = task_two().fuse();
            pin_mut!(t1, t2);

            select! {
                () = t1 => println!("task one completed first"),
                () = t2 => println!("task two completed first"),
                () = task_two().fuse() => println!("task third completed second")
            }
        }
        block_on(race_tasks());
```

select 还支持 `default` 和 `complete`

* `default` 当存在 `default` 时，select 就是非阻塞的，不会阻塞等待有一个就绪
* `complete`
    * 当所有的 future 都 Ready 后，再会执行 complete 语句
    * 在 loop 中 `<expression>` 不能使用 函数调用，使用的话会造成死循环（需要实现准备好）

```rust
        use futures::{future};

        async fn count() {
            async fn b () -> i32 {
                task::sleep(time::Duration::from_millis(2)).await;
                6
            }
            let mut a_fut = future::ready(4);
            // let mut b_fut = future::ready(6);
            let mut b_fut = Box::pin(b().fuse());
            let mut total = 0;

            loop {
                println!("---");
                select! {
                    a = a_fut => total += a,
                    b = b_fut => total += b,
                    // c = b().fuse() => { // 死循环
                    //     println!("c");
                    //     total += c
                    // },
                    complete => break,
                    // default => println!("default"),
                };
            }
            assert_eq!(total, 10);
        }
        block_on(count())
```

### 2、async 块中的`?`

```rust
    async fn foo() -> Result<(), String> { Ok(())}
    async fn bar() -> Result<(), String> { Ok(())}

    // let fut = async { // 报错，需要类型声明
    //     foo().await?;
    //     bar().await?;
    //     Ok(())
    // };

    let fut2 = async {
        foo().await?;
        bar().await?;
        Ok::<(), String>(()) // <- note the explicit type annotation here
    };
```

### 3、 async 块 的 Send 推断

```rust
    {
        use std::rc::Rc;

        #[derive(Default)]
        struct NotSend(Rc<()>);

        async fn bar() {}
        async fn foo() {
            // NotSend::default(); // 不报错，实现了Send
            // let x = NotSend::default(); // 报错，未实现Send
            // 解决方案 创建一个作用域
            {
                let x = NotSend::default();
            }
            bar().await;
        }

        fn require_send(_: impl Send) {}

        require_send(foo());
    }
```

### 4、async 递归

```rust
use futures::future::{BoxFuture, FutureExt};

fn recursive() -> BoxFuture<'static, ()> {
    async move {
        recursive().await;
        recursive().await;
    }.boxed()
}
```

可以考虑使用 [async-recursion](https://github.com/dcchut/async-recursion) 库

### 5、异步方法和异步特质

rust 支持异步方法

```rust
    {
        struct A {}
        impl A {
            async fn test(&self) -> i32 {
                1
            }
        }
    }
```

rust 原生不支持异步 trait，可以使用第三方库实现 https://github.com/dtolnay/async-trait
