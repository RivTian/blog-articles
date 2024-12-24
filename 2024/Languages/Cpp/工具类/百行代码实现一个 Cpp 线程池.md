
#Cpp  #ThreadBool 

发现了一个轻量级的线程池，源代码不足百行，用于理解线程池是个不错的选择！下面进行代码分析。

原地址：[https://github.com/mtrebi/thread-pool](https://github.com/mtrebi/thread-pool)（叠甲：用于生产要充分测试）

## 源码

```cpp
#pragma once

#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <utility>
#include <vector>

#include "SafeQueue.h"

class ThreadPool {
private:
    class ThreadWorker {
    private:
        int m_id;
        ThreadPool * m_pool;
    public:
        ThreadWorker(ThreadPool * pool, const int id)
                : m_pool(pool), m_id(id) {
        }

        void operator()() {
            std::function<void()> func;
            bool dequeued;
            while ( !m_pool->m_shutdown )
            {
                {
                    std::unique_lock<std::mutex> lock(m_pool->m_conditional_mutex);
                    if (m_pool->m_queue.empty())
                    {
                        m_pool->m_conditional_lock.wait(lock);
                    }
                    dequeued = m_pool->m_queue.dequeue(func);
                }
                if (dequeued)
                {
                    func();
                }
            }
        }
    };

    bool m_shutdown;
    SafeQueue <std::function<void()>> m_queue;
    std::vector<std::thread> m_threads;
    std::mutex m_conditional_mutex;
    std::condition_variable m_conditional_lock;
public:
    ThreadPool(const int n_threads)
            : m_threads(std::vector<std::thread>(n_threads)), m_shutdown(false) {
    }

    ThreadPool(const ThreadPool &) = delete;

    ThreadPool(ThreadPool &&) = delete;

    ThreadPool & operator=(const ThreadPool &) = delete;

    ThreadPool & operator=(ThreadPool &&) = delete;

    // Inits thread pool
    void init() {
        for ( int i = 0; i < m_threads.size(); ++i )
        {
            m_threads[i] = std::thread(ThreadWorker(this, i));
        }
    }

    // Waits until threads finish their current task and shutdowns the pool
    void shutdown() {
        m_shutdown = true;
        m_conditional_lock.notify_all();

        for ( int i = 0; i < m_threads.size(); ++i )
        {
            if (m_threads[i].joinable())
            {
                m_threads[i].join();
            }
        }
    }

    // Submit a function to be executed asynchronously by the pool
    template<typename F, typename...Args>
    auto submit(F && f, Args && ... args) -> std::future<decltype(f(args...))> {
        // Create a function with bounded parameters ready to execute
        std::function<decltype(f(args...))()> func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
        // Encapsulate it into a shared ptr in order to be able to copy construct / assign 
        auto task_ptr = std::make_shared<std::packaged_task<decltype(f(args...))()>>(func);

        // Wrap packaged task into void function
        std::function<void()> wrapper_func = [task_ptr]() {
            (*task_ptr)();
        };

        // Enqueue generic wrapper function
        m_queue.enqueue(wrapper_func);

        // Wake up one thread if its waiting
        m_conditional_lock.notify_one();

        // Return future from promise
        return task_ptr->get_future();
    }
};
```

这段C++代码实现了一个简单的线程池，允许异步提交任务，并在需要时关闭线程池。下面逐行解释这段代码：

## ThreadWorker

```cpp
class ThreadPool {
private:
  class ThreadWorker {
  private:
    int m_id;
    ThreadPool * m_pool;
  public:
    ThreadWorker(ThreadPool * pool, const int id)
      : m_pool(pool), m_id(id) {
    }
  // ...
  }
}
```

`ThreadPool` 类的声明，包含一个私有嵌套类 `ThreadWorker`。每个工作线程将执行一个 `ThreadWorker` 实例。`ThreadWorker` 类有一个ID和指向拥有它的 `ThreadPool` 的指针。

### void operator()()

```cpp
void operator()() {
      std::function<void()> func;
      bool dequeued;
      while (!m_pool->m_shutdown) {
        {
          std::unique_lock<std::mutex> lock(m_pool->m_conditional_mutex);
          if (m_pool->m_queue.empty()) {
            m_pool->m_conditional_lock.wait(lock);
          }
          dequeued = m_pool->m_queue.dequeue(func);
        }
        if (dequeued) {
          func();
        }
      }
    }
  };
```

这是线程池中每个工作线程执行的运算符重载函数。这个函数 (`operator()`) 定义了工作线程的主体，即线程在执行期间将执行的操作。让我们逐步解释它：

1. `void operator()()`: 这是一个函数调用运算符的重载，允许对象像函数一样被调用。在这里，它使得每个线程对象可以像函数一样运行。
    
2. `std::function<void()> func;`: 定义了一个可调用对象的实例，它没有参数并且没有返回值。这个对象将用于存储从任务队列中取出的待执行函数。
    
3. `bool dequeued;`: 一个布尔变量，用于跟踪是否成功地从任务队列中取出了一个任务。
    
4. `while (!m_pool->m_shutdown) {`: 在线程池没有被关闭的情况下，持续执行以下操作。
    
5. `{`: 这是一个块，创建一个新的作用域，以便在离开这个作用域时释放 `std::unique_lock<std::mutex>` 对象。
    
6. `std::unique_lock<std::mutex> lock(m_pool->m_conditional_mutex);`: 创建一个独占锁，使用 `std::mutex` 对象 `m_conditional_mutex` 进行加锁。这是为了保护对条件变量的访问。
    
7. `if (m_pool->m_queue.empty()) {`: 如果任务队列为空。
    
8. `m_pool->m_conditional_lock.wait(lock);`: 等待条件变量，即等待有任务可用时被唤醒。在等待期间，锁会被释放，以允许其他线程访问共享资源。一旦被唤醒，锁会再次被获取。
    
9. `}`: 结束条件变量等待的块。
    
10. `dequeued = m_pool->m_queue.dequeue(func);`: 从任务队列中取出一个任务（通过 `dequeue` 函数），并将它存储在 `func` 中。`dequeue` 返回一个布尔值，表示是否成功取出任务。
    
11. `}`: 结束块，将 `std::unique_lock<std::mutex>` 的作用域结束，锁会被释放。
    
12. `if (dequeued) { func(); }`: 如果成功取出了任务，则执行该任务。这是通过调用 `func()` 来实现的，`func` 存储了待执行的函数。
    

整个 `while` 循环会不断尝试从任务队列中取出任务，并在任务队列为空时等待条件变量被唤醒。当线程池被关闭 (`m_pool->m_shutdown` 为 `true`) 时，循环结束，线程将退出。这确保了工作线程在关闭线程池前完成其任务。

## ThreadPool

### 成员变量

```cpp
bool m_shutdown;
  SafeQueue<std::function<void()>> m_queue;
  std::vector<std::thread> m_threads;
  std::mutex m_conditional_mutex;
  std::condition_variable m_conditional_lock;
public:
  ThreadPool(const int n_threads)
    : m_threads(std::vector<std::thread>(n_threads)), m_shutdown(false) {
  }
```

1. `ThreadPool` 类的公共部分包含各种成员：
    
2. `m_shutdown`：表示线程池是否应该关闭的标志。
    
3. `m_queue`：`SafeQueue` 类的一个实例，这是一个线程安全的队列，用于存储任务。
    
4. `m_threads`：代表工作线程的 `std::thread` 对象的向量。
    
5. `m_conditional_mutex`：用于同步访问条件变量的互斥锁。
    
6. `m_conditional_lock`：条件变量，用于在队列中有任务时通知线程。
    

构造函数初始化工作线程的数量，并将关闭标志设置为 `false`。

```cpp
ThreadPool(const ThreadPool &) = delete;
ThreadPool(ThreadPool &&) = delete;

ThreadPool & operator=(const ThreadPool &) = delete;
ThreadPool & operator=(ThreadPool &&) = delete;
```

删除了复制和移动构造函数，以及复制和移动赋值运算符。这确保了 `ThreadPool` 对象不能被复制或移动。

### init()

```cpp
void init() {
    for (int i = 0; i < m_threads.size(); ++i) {
      m_threads[i] = std::thread(ThreadWorker(this, i));
    }
  }
```

`init()` 方法通过创建 `ThreadWorker` 实例并将其存储在 `m_threads` 向量中初始化工作线程。

### shutdown()

```cpp
void shutdown() {
    m_shutdown = true;
    m_conditional_lock.notify_all();

    for (int i = 0; i < m_threads.size(); ++i) {
      if(m_threads[i].joinable()) {
        m_threads[i].join();
      }
    }
  }
```

`shutdown()` 方法将关闭标志设置为 `true`，通过条件变量通知所有线程，然后加入所有线程以等待它们完成。

### submit函数

```cpp
template<typename F, typename...Args>
  auto submit(F&& f, Args&&... args) -> std::future<decltype(f(args...))> {
    // 创建一个带有绑定参数的可执行函数
    std::function<decltype(f(args...))()> func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
    // 将其封装到 shared ptr 中以便能够进行复制构造和赋值
    auto task_ptr = std::make_shared<std::packaged_task<decltype(f(args...))()>>(func);

    // 将封装的任务包装成 void 函数
    std::function<void()> wrapper_func = [task_ptr]() {
      (*task_ptr)(); 
    };

    // 将通用的封装函数加入队列
    m_queue.enqueue(wrapper_func);

    // 唤醒一个等待的线程
    m_conditional_lock.notify_one();

    // 从 promise 返回 future
    return task_ptr->get_future();
  }
};
```

这是线程池中的 `submit` 方法，用于提交任务给线程池进行异步执行。下面是对该方法的详细解释： 1. `template<typename F, typename...Args>`: 这是一个函数模板声明，允许 `submit` 函数接受可变数量的模板参数。这使得 `submit` 能够接受任意类型和数量的参数。

1. `auto submit(F&& f, Args&&... args) -> std::future<decltype(f(args...))>`: 这是函数的声明，返回一个 `std::future` 对象，表示将来会有一个返回值。返回值的类型由 `decltype(f(args...))` 推导得出，即根据传递给 `submit` 函数的参数类型和返回值类型来确定。
    
2. `std::function<decltype(f(args...))()> func = std::bind(std::forward<F>(f), std::forward<Args>(args)...);`: 创建了一个 `std::function` 对象 `func`，用于存储一个可调用对象，这个对象是通过使用 `std::bind` 绑定传递给 `submit` 函数的函数 `f` 和参数 `args` 而得到的。
    
3. `auto task_ptr = std::make_shared<std::packaged_task<decltype(f(args...))()>>(func);`: 使用 `std::packaged_task` 将 `func` 封装起来，返回一个 `std::shared_ptr`，使得可以在不同的线程之间共享该任务。
    
4. `std::function<void()> wrapper_func = [task_ptr]() { (*task_ptr)(); };`: 创建一个无参数、无返回值的 `std::function` 对象 `wrapper_func`，其内容是一个 lambda 表达式，该 lambda 表达式会调用之前封装的任务 (`*task_ptr`)。
    
5. `m_queue.enqueue(wrapper_func);`: 将封装的任务加入线程池的任务队列。
    
6. `m_conditional_lock.notify_one();`: 唤醒一个等待的线程，通知它有任务可执行。
    
7. `return task_ptr->get_future();`: 返回一个 `std::future` 对象，表示将来会有一个返回值。该 `future` 对象通过 `task_ptr` 的 `get_future` 方法获得，可以用于获取任务执行的结果。
    

这样，通过调用 `submit` 函数，用户可以将一个函数及其参数提交给线程池异步执行，并得到一个 `std::future` 对象，以便将来获取任务执行的结果。

## 细节说明

### std::function<void()> func

`std::function` 是 C++ 中的一个模板类，用于表示可调用对象（函数、函数指针、函数对象等）的包装器。它允许你以一种类型安全的方式存储、复制和调用可调用对象。在你提到的例子中，`std::function<void()> func;` 表示一个可以保存返回类型为 `void`，没有参数的可调用对象的 `std::function` 对象。

下面是一些关键概念的详细说明：

1. **std::function**: 这是 C++ 标准库中的一个模板类，定义在头文件 `<functional>` 中。它是一个通用的函数封装器，可以用来存储、复制和调用可调用对象，如函数、函数指针、成员函数指针、Lambda 表达式等。
    
2. **void()**: 这是函数的签名，表示一个不带参数并且返回类型为 `void` 的函数。在 `std::function` 中，你可以指定被包装的可调用对象的签名，以确保类型安全。
    
3. **func**: 这是一个名为 `func` 的对象，类型为 `std::function<void()>`。它可以存储任何满足指定签名的可调用对象。在这个例子中，`func` 只能包装返回类型为 `void`，没有参数的可调用对象。
    

下面是一个简单的例子，演示如何使用 `std::function<void()>`：

```cpp
#include <iostream>
#include <functional>

// 一个简单的可调用对象
void simpleFunction() {
    std::cout << "Hello, I'm a simple function!" << std::endl;
}

int main() {
    // 定义一个 std::function 对象，可以包装返回类型为 void，没有参数的可调用对象
    std::function<void()> func;

    // 将 simpleFunction 包装到 func 中
    func = simpleFunction;

    // 调用 func，实际上调用了 simpleFunction
    func();

    return 0;
}
```

在这个例子中，`std::function<void()>` 被用于包装一个简单的函数 `simpleFunction`，然后通过 `func()` 调用它。这使得代码更加灵活，因为你可以在运行时切换不同的可调用对象给 `func`。

### std::unique_lock[std::mutex](std::mutex) lock

`std::unique_lock` 是 C++ 中与互斥量一起使用的锁管理类，用于提供对互斥量的灵活性和安全性。在你提到的例子中，`std::unique_lock<std::mutex> lock(m_pool->m_conditional_mutex);` 表示创建了一个 `std::unique_lock` 对象，并使用了 `m_pool->m_conditional_mutex` 作为关联的互斥量。

下面是对这个语句的详细解释：

1. **`std::unique_lock`**: 是一个模板类，用于提供锁的管理。它位于头文件 `<mutex>` 中。`std::unique_lock` 提供了比 `std::lock_guard` 更灵活的锁定机制，允许你在锁定期间释放和重新获取锁。
    
2. **`<std::mutex>`**: 表示使用的互斥量的类型。在这个例子中，使用了 `std::mutex` 类型的互斥量。
    
3. **`lock(m_pool->m_conditional_mutex)`**: 创建了一个 `std::unique_lock` 对象 `lock`，并通过构造函数传入 `m_pool->m_conditional_mutex` 作为关联的互斥量。这就意味着 `lock` 对象将管理这个互斥量的锁定和解锁。
    

在多线程编程中，使用 `std::unique_lock` 的一个主要优势是它提供了更多的控制，例如：

- **手动锁定和解锁**: 与 `std::lock_guard` 不同，`std::unique_lock` 允许你手动调用 `lock()` 和 `unlock()` 方法来手动控制锁的状态。
    
- **灵活性**: `std::unique_lock` 的构造函数允许你指定锁的延迟时间、锁定策略等。
    
- **条件变量支持**: `std::unique_lock` 与条件变量结合使用时，可以通过在构造函数中传递条件变量来支持在等待条件时释放锁，这在实现复杂的同步逻辑时非常有用
    

总的来说，`std::unique_lock<std::mutex> lock(m_pool->m_conditional_mutex);` 表示创建了一个 `std::unique_lock` 对象，该对象管理 `m_pool->m_conditional_mutex` 的锁定和解锁，提供了更灵活的锁定机制。

### std::future<decltype(f(args...))>

`std::future<decltype(f(args...))>` 是一个使用在 C++ 中的模板类型，用于声明一个 `std::future` 对象，其中包含了一个异步操作的结果，而这个结果的类型是通过对 `f(args...)` 表达式进行求值所得到的。

让我们逐步解释这个类型：

1. **`decltype(f(args...))`**: 这是一种使用在 C++ 中的关键字，用于获取某个表达式的类型。在这个例子中，`f(args...)` 表达式是可调用对象 `f` 被传递参数 `args...` 调用后的结果。`decltype` 用于获取这个表达式的类型。
    
2. **`std::future<...>`**: 这部分是一个模板，用于声明一个 `std::future` 对象，它包含了异步操作的结果。在尖括号中填入了上一步中获取到的类型，即 `decltype(f(args...))`。
    

因此，`std::future<decltype(f(args...))>` 表示一个 `std::future` 对象，该对象中包含了异步操作的结果，而这个结果的类型是通过对 `f(args...)` 表达式进行求值所得到的。在实际的使用中，你可以通过这个 `std::future` 对象来获取异步操作的结果。