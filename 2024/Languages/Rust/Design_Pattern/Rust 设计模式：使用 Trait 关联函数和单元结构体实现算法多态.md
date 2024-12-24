#Rust 

最近在用 Rust 编写在一个 NLP 的模块，需要表达一个算法的多种不同实现，只用单元结构体和 Trait 的关联函数的抽象方式，可以非常优雅的组织代码，现在记录如下。

###  单元结构体 

是一个没有字段的结构体，这种结构体被称作 "单元结构体"（Unit Struct）。单元结构体类似于没有任何成员的元组（即 ()，或说 "unit" 类型），通常用于在类型层面提供一个独特的标识。

```rust
struct MyUnitStruct;

fn main() {
    let _instance = MyUnitStruct;
}
```

Rust 编译器会自动为单元结构体定义一个同名常量，所以你可以无需构造即可使用。

单元结构体可以用在需要类型标记但无需实际数据的场合，例如在 impl 块中为其实现特定的方法，或者作为标识类型使用在泛型编程中。对于组成更复杂类型系统的 "零值"（zero-sized types）而言，它们是很有用的。零大小类型（ZST）在内存中不占用空间，但在编译时期它们为类型系统提供有用的信息。

###  关联函数 

在 Trait 中定义的函数，如果函数签名并不包含 self，那么称之为关联函数，反之成为关联方法。如果 Trait 只是包含关联函数，那么它不会要求实现该特质的类型具备某种内部状态，因为它们不会操作类型的实例。这种特质主要用于定义一组相关的函数，而不是对象的方法。

### 算法

举个例子，假设我们有一个名为 StringDistance 的特质，它只包含关联函数：

```rust
trait StringDistance {
    // 關聯函數來計算兩個字符串之間的距離
    fn distance(s1: &str, s2: &str) -> usize;

    // 默認函數是否有空串，是則返回長度差作爲距離，否則使用指定的距離算法
    fn is_empty_distance(s1: &str, s2: &str) -> usize {
        match (s1.is_empty(), s2.is_empty()) {
            (true, true) => 0,
            (true, false) => s2.len(),
            (false, true) => s1.len(),
            (false, false) => Self::distance(s1, s2),
        }
    }
}
```

这个特质可以在不同的上下文中实现，每次实现都会根据特定算法来实现 compute 函数。例如

```rust
// 海明距離的實現
struct Hamming;

impl StringDistance for Hamming {
    fn distance(s1: &str, s2: &str) -> usize {
        s1.chars()
            .zip(s2.chars())
            .filter(|(a, b)| a != b)
            .count()
    }
}

struct Levenshtein;

impl StringDistance for Levenshtein {
    fn distance(s1: &str, s2: &str) -> usize {
        // 初始化一個二維數組，用於存儲距離
        let mut matrix = vec![vec![0; s2.len() + 1]; s1.len() + 1];

        // 初始化矩陣的第一行和第一列
        for i in 0..=s1.len() {
            matrix[i][0] = i;
        }

        for j in 0..=s2.len() {
            matrix[0][j] = j;
        }

        // 填充矩陣
        for (i, c1) in s1.chars().enumerate().map(|(i, c)| (i + 1, c)) {
            for (j, c2) in s2.chars().enumerate().map(|(i, c)| (i + 1, c)) {
                let cost = if c1 == c2 { 0 } else { 1 };
                matrix[i][j] = *[
                    matrix[i - 1][j] + 1,     // 刪除
                    matrix[i][j - 1] + 1,     // 插入
                    matrix[i - 1][j - 1] + cost, // 替換
                ]
                .iter()
                .min()
                .unwrap();
            }
        }

        matrix[s1.len()][s2.len()]
    }
}
```

此外，这种特质可以用作泛型约束，允许编写可对多种算法实现进行操作的代码：

```rust
// 定義一個泛型函數，它使用了實現了 StringDistance 特質的類型
fn calculate_string_distance<T: StringDistance>(s1: &str, s2: &str) -> usize {
    T::distance(s1, s2)
}

fn main() {
    let s1 = "kitten";
    let s2 = "sitting";

    // 使用泛型函數計算海明距離
    let hamming_dist = calculate_string_distance::<Hamming>(s1, s2);
    println!("Hamming distance between '{}' and '{}' is {}", s1, s2, hamming_dist);

    // 使用泛型函數計算萊文斯坦距離
    let levenshtein_dist = calculate_string_distance::<Levenshtein>(s1, s2);
    println!("Levenshtein distance between '{}' and '{}' is {}", s1, s2, levenshtein_dist);
}
```

在这个例子中，calculate_string_distance 是一个泛型函数，它接受任意实现了 StringDistance 特质的类型。我们指定了泛型类型参数 T，并通过约束 T: StringDistance 来确保传递给该函数的类型必须实现了 StringDistance 特质。

这种方式允许我们编写高度通用的代码，因为函数 calculate_string_distance 不依赖于特定的字符串距离算法实现。我们可以轻松地添加更多实现 StringDistance 的结构体，并将它们用于同一个泛型函数中，而无需修改现有代码。