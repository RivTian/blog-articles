在這篇文章中，我們將創建一個能執行的 Rust 腳本來測量 CPU 的性能。

爲了實現目標，我們將創建一些在循環中運行的虛擬計算，分佈在所有可用的 CPU 內核上。理想情況下，我們的計算需要 CPU 密集型任務，所以我們儘可能接近 100% 的 CPU 使用率。

創建一個 Rust 項目：

```sh
cargo new cpu-benchmark
```

在 Cargo.toml 文件中添加如下代碼：

```toml
[dependencies]
indicatif = "0.17.5"
sysinfo = "0.29.7"

[profile.release]
lto = "fat"
strip = "debuginfo"
```

因爲計算階乘是一項 cpu 密集型任務，所以我們首先在 src/main.rs 文件中寫入一個計算階乘的函數：

```rust
pub fn factorial(num: u128) -> u128 {
    (1..=num).product()
}
```

然後，我們在循環中調用這個階乘函數：

```rust
fn add_one_loop(&n_loops: &u64) {
    for _in in 0..n_loops {
        let _ = factorial(20);
    }
}
```

在 main 函數中，我們需要做一些處理：

- 找出有多少內核和線程可用
    
- 爲了方便打印一些系統信息
    
- 多線程處理
    
- 顯示進度條
    

對於系統信息，我們使用一個名爲 sysinfo 的 crate：

```rust
use sysinfo::{System, SystemExt};

fn main() {
    let mut sys = System::new_all();
    sys.refresh_all();
    // 顯示系統信息
    println!("System name:             {:?}", sys.name());
    println!("System kernel version:   {:?}", sys.kernel_version());
    println!("System OS version:       {:?}", sys.os_version());
    println!("System host name:        {:?}", sys.host_name());
    // 獲取CPU數
    println!("Number of available threads: {}", sys.cpus().len());
}
```

進度條我們使用另一個名爲 indicatif 的 crate，此外，我們還測量開始和結束之間的時間，並計算每秒的計算次數以獲得分數。

main.rs 的完整代碼如下：

```rust
use std::{thread::available_parallelism, time::Instant, env};

use indicatif::ProgressBar;
use sysinfo::{System, SystemExt};


pub fn factorial(num: u128) -> u128 {
    (1..=num).product()
}

fn add_one_loop(&n_loops: &u64) {
    for _in in 0..n_loops {
        let _ = factorial(20);
    }
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let num_calcs_arg: Option<&String> = args.get(1);
    let num_calcs: u64 = match num_calcs_arg {
        Some(num_calcs_arg) => num_calcs_arg.trim().parse::<u64>().unwrap(),
        None => 400000000, 
    };
    let num_iters: u64 = 20000;
    let total_calc: u64 = num_calcs * num_iters;
    println!(
        "Running {} calculations over {} iterations each with a total of {} calculations.",
        &num_calcs, &num_iters, &total_calc,
    );

    // 獲取系統信息
    let mut sys = System::new_all();
    sys.refresh_all();
    // 顯示系統信息
    println!("System name:             {:?}", sys.name());
    println!("System kernel version:   {:?}", sys.kernel_version());
    println!("System OS version:       {:?}", sys.os_version());
    println!("System host name:        {:?}", sys.host_name());
    // 獲取CPU數
    println!("Number of available threads: {}", sys.cpus().len());

    // 獲取我們可以使用多少線程的信息，並使用其中的一半
    let available_cores: u64 = available_parallelism().unwrap().get() as u64;
    let iter_per_core: u64 = num_calcs / available_cores;

    let now = Instant::now();

    let bar = ProgressBar::new(num_iters);
    for _i in 0..num_iters {
        let mut results = Vec::new();
        let mut threads = Vec::new();
        for _i in 0..available_cores {
            threads.push(std::thread::spawn(move || add_one_loop(&iter_per_core)));
        }
        for thread in threads {
            results.extend(thread.join());
        }
        bar.inc(1);
    }
    bar.finish();

    let elapsed = now.elapsed();
    let calc_per_sec: f64 = (total_calc as f64) / (elapsed.as_secs() as f64);
    println!("Total runtime: {:.2?}", elapsed);
    println!("Calculations per second: {:.2?} seconds.", calc_per_sec);
}
```

現在運行如下命令測試我們的簡單腳本：

```sh
cargo run --release
```

結果如下：

```sh
Running 400000000 calculations over 20000 iterations each with a total of 8000000000000 calculations.
System name:             Some("Darwin")
System kernel version:   Some("23.4.0")
System OS version:       Some("14.4.1")
System host name:        Some("Koshkaaas-MacBook-Pro.local")
Number of available threads: 10
████████████████████████████████████████████████████████████████████████████████████ 20000/20000Total runtime: 1.99s
Calculations per seconds: 4011255834575.31 seconds
```

我們獲得了系統信息、運行時間、進度條和可解釋的基準分數。