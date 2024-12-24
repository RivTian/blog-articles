1. 错误地使用可变和不可变借用

```rust
let mut data = vec![1, 2, 3];
let x = &data[0];
data.push(4);
println!("{}", x);
```

不能在有不可变引用时修改数据。

2. 忘记处理 Option

```rust
fn main() {
    let some_number = Some(5);
    let sum = some_number + 5; // 错误：Option 类型不能这样直接使用
}
```

使用 match 或 if let 来处理 Option。

```rust
let sum = if let Some(n) = some_number { n + 5 } else { 0 };
```

3. 期望的引用而不是实际值

```rust
fn foo(x: &i32) {
    // ...
}

fn main() {
    let x = 10;
    foo(x); // 错误：应传递引用而不是值
}
```

传递引用而不是值。

```rust
foo(&x);
```

4. 不匹配的类型

```rust
fn main() {
    let flag = true;
    let number = if flag { 5 } else { "six" }; // 错误：不匹配的类型
}
```

解决方案：确保所有的分支返回相同的类型。

```rust
let number = if flag { 5 } else { 6 };
```

5. 忘记导入trait

```rust
use std::io::Write; // 因为 Write trait 已导入

fn main() {
    let mut buffer = Vec::new();
    buffer.write_all(b"hello").unwrap();
    // 现在可以正常使用 write_all 方法，因为 Write trait 已导入
}
```

必须导入Write trait才能使用其方法。

6. 试图在遍历向量的同时修改它

```rust
fn main() {
    let mut vec = vec![1, 2, 3];
    for val in &vec {
        vec.push(*val + 10); // 错误！不能在遍历时修改vec
    }
}
```

修复: 使用迭代器的.for_each()方法或者先收集需要做的更改，然后再应用它们。

7. 不使用可变引用来修改向量中的值

```rust
fn main() {
    let mut vec = vec![1, 2, 3];
    for val in vec.iter() {
        *val *= 2; // 错误！因为`val`是不可变引用
    }
}
```

修复: 使用.iter_mut()来获取可变引用。

```rust
for val in vec.iter_mut() {
    *val *= 2;
}
```

8. 在向量迭代时错误地使用索引

```rust
fn main() {
    let vec = vec![10, 20, 30];
    for i in 0..vec.len() {
        println!("{}", vec[i]); // 索引方式正确
        // 如果修改为 vec[i+1] 就会出错
    }
}
```

修复: 确保在使用索引时不要越界。

9. 未处理get()返回Option的情况

```rust
fn main() {
    let vec = vec![1, 2, 3];
    let val = vec.get(5); // 返回None，因为索引超出范围
    println!("{}", val.unwrap()); // 错误！这会panic
}
```

修复: 使用.unwrap_or()或.unwrap_or_else()处理None情况。

```rust
let val = vec.get(5).unwrap_or(&0); // 使用默认值
println!("{}", val);
```

10. 使用不恰当的迭代器消费方法

```rust
fn main() {
    let vec = vec![1, 2, 3, 4, 5];
    let even_numbers: Vec<_> = vec.iter().map(|x| x * 2).collect(); // 正确
    let even_number = even_numbers.first().unwrap(); // 尝试获取第一个数
    println!("{}", even_number * 2); // 错误！even_number的类型是&&i32
}
```

修复: 在使用值之前进行适当的解引用。

```rust
println!("{}", (**even_number) * 2); // 解引用两次到i32
```

11. 在遍历过程中错误地使用for循环去显式创建迭代器

```rust
fn main() {
    let vec = vec![1, 2, 3];
    for i in 0..vec.iter().count() { // 错误！不必要的计数和索引
        println!("{}", vec[i]);
    }
}
```

修复: 直接遍历迭代器。

```rust
for &val in vec.iter() {
    println!("{}", val);
}
```

12. 忘记引用计数（Rc/Arc）中的借用规则

```rust
use std::rc::Rc;
let foo = Rc::new(5);
let a = *foo;
```

使用*foo应该先调用.clone()去获取Rc里面的值。

13. 错误地对字符串切片

```rust
let s = String::from("hello");
let part = &s[0..2];
```

字符串索引应该在字符边界处切片。使用.chars()和.bytes()。

14. 未处理泛型边界

```rust
fn print_it<T>(value: T) {
    println!("{}", value);
}
```

需要为T添加Display trait约束。





15. 尝试修改迭代器产生的不可变引用

```rust
let mut vector = vec![1, 2, 3];
for item in vector.iter_mut() {
    *item += 10;
}
```

使用iter_mut()来获取可变引用。

16. 没有正确使用借用和所有权

```rust
fn use_data(data: &Vec<i32>) {
    // do something
}
fn main() {
    let data = vec![1, 2, 3];
    use_data(&data);  // Correct
    use_data(data);   // Incorrect
}
```

保持对参数的引用和传递一致。

17. 在match表达式内使用=而不是if来匹配守卫

```rust
let num = 10;
match num {
    x if x = 5 => println!("Five"),
    _ => println!("Not five"),
}
```

守卫使用==来进行比较。

18. 枚举匹配不全

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
fn match_direction(dir: Direction) {
    match dir {
        Direction::Up => println!("Going up!"),
        // 缺少其他方向的匹配
    }
}
```

应该匹配枚举的所有变体。

19. 在非异步函数中使用await

```rust
fn main() {
    let future = async_operation();
    let result = future.await;  // Error: `await` is only valid in async function
}
```

必须在async函数中使用await。

20. 将&str推入String而不使用push_str或+操作符

```rust
let mut hello = String::from("Hello, ");
hello.push("world!");  // Error: `push` takes a single character
```

使用push_str或+来连接字符串。

21. rust语言的不变性

```rust
let x = 5;
let y = &x;
let z = &mut x;  // Error: Cannot borrow `x` as mutable because it is also borrowed as immutable
```

不允许在不可变引用存在时创建可变引用。

22. 没有标记可变的闭包变量

```rust
let mut count = 0;
let mut inc = || {
    count += 1;  // Error: Cannot borrow immutable local variable `count` as mutable
};
inc();
```

闭包外的变量需要被标记为mut。

23. 错误地使用结构体更新语法

```rust
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
let p2 = Point { y: 3, ..p1 };  // Error: `p1` used after move
```

p1在更新之后不能被再次使用，除非字段是类型实现了Copy trait。

24. 在包含引用的结构体中提供显式生命周期

```rust
struct Student<'a> {
    name: &'a str,
    age: u8,
}

let student_name = String::from("John");
let student = Student {
    name: &student_name,
    age: 20,
};
```

当结构体包含引用时，需要指定生命周期参数。

25. 没有意识到函数调用会获取所有权

```rust
let mut v = vec![1, 2, 3];
drop(v);
v.push(4); // Error: Use of moved value `v`
```

使用drop后，原值将不再有效。

26. 交替使用 `&` 和 `ref`

```rust
let mut x = 10;
let y = &x;
let ref z = x;
```

使用ref在模式匹配中创建引用，&用于创建引用。

27. 忘记在使用线程时处理潜在的失败情况

```rust
use std::thread;

let handle = thread::spawn(|| {
    panic!("oh no!");
});

handle.join(); // Error: Ignored `Result` which may indicate the thread has panicked
```

使用.join().unwrap()来处理线程中可能发生的错误。

28. 错误的转型

```rust
let x: i32 = 10;
let y: u64 = x as u64; // Correct
let z: f64 = x.into(); // Error: A type annotation is required
```

选择合适的类型转换方法，确保类型实现了相应的转换trait。

29. 使用 unwrap_or_else 时搭配非闭包参数

```rust
let result = Some(2);
let number = result.unwrap_or_else(0); // Error: Expected a closure that returns a value
```

使用unwrap_or_else(|| 0)确保提供的是一个闭包。

30. 没有初始化所有字段

```rust
struct Point {
    x: i32,
    y: i32,
}

let point = Point { x: 5 }; // Error: Missing field `y` in initializer of `Point`
```

确保初始化结构体的所有字段。



31. 未考虑运算符优先级

```rust
let x = 1 + 2 * 3; // x is 7, not 9
```

注意操作符的优先级顺序。

32. 忘记在闭包中考虑借用检查器

```rust
let x = 10;
let closure = || println!("{}", x);
drop(x); // Error: `x` moved due to use in closure
```

闭包引用的外部变量将受到借用检查器的约束。

33. 尝试使用引用来扩展一个 Vec

```rust
let mut vec = vec![1, 2, 3];
let items = &[4, 5, 6];
vec.extend(items); // This is correct
vec.extend(&items); // Error: Expected an iterator, found a reference
```

使用extend时确保提供的是一个迭代器。

34. 混淆数组长度和向量的大小

```rust
let array = [1; 10]; // Array of length 10
let vector = vec![1; 10]; // Vector of length 10
println!("{}", array.len()); // Outputs 10
println!("{}", vector.size()); // Error: No method named `size`, should be `len()`
```

向量(Vec)使用len方法来获取长度。

35. 试图返回对局部变量的引用

```rust
fn get_str() -> &str {
    let s = String::from("hello");
    &s // Error: `s` does not live long enough
}
```

返回的引用必须拥有有效的生命周期。

36. 将函数指针与闭包混淆

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
let closure = add_one; // Error: Expected closure, found function pointer
let real_closure = |x| add_one(x);
```

函数指针和闭包在Rust中是不同的类型。

37. 忽略了变量阴影（变量遮蔽）问题

```
let x = 5;
let x = x + 1; // Shadows the previous `x`
```

检查变量遮蔽（shadowing），确保没有无意的遮蔽。

38. 在可以使用迭代器的地方使用了循环

```
let numbers = vec![1, 2, 3];
let mut doubled = Vec::new();
for num in numbers {
    doubled.push(num * 2);
}
```

可以使用迭代器的map和collect的更为优雅的方式。

39. 忽略了函数和方法中生命周期的概念

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```

当返回引用时，需要确保所有引用都有一个探样的生命周期。

40. 在应该使用安全的引用之处使用了原始指针

```rust
let x = 5;
let y = &x as *const i32; // Raw pointer, not recommended for regular use
```

除非确实需要，否则尽量使用安全的借用来代替原始指针。

41. 错误地使用 ? 运算符进行错误处理

```rust
fn fails() -> Result<(), String> {
    Err("Failed".to_string())
}

fn test() -> Option<()> {
    fails()?; // Error: `?` can only be used in functions that return `Result`
    Some(())
}
```

?运算符只能用于返回Result或Option的函数中。

42. 在进行算术运算时未处理溢出情况

```rust
let x: u8 = 255;
let y: u8 = x + 1; // Error: Attempt to add with overflow
```

使用可检测溢出的方法，例如x.checked_add(1)。

43. 在使用实现了Copy特征的类型时，使用clone()代替&

```rust
let x = 5;
let y = x.clone(); // Unnecessary since `i32` implements the `Copy` trait
let z = x; // This is fine
```

如果类型实现了Copy trait，应直接赋值代替clone。

44. **尝试修改不可变字符串**

```rust
let s = "hello";
s.push_str(" world"); // error
```

解决方案：使用String类型而非&str类型来创建可变字符串。

```rust
let mut s = String::from("hello");
s.push_str(" world");
```

45. **错误地尝试使用索引访问字符串中的字符**

```rust
let s = "hello";
let c = s[0]; // error
```

解决方案：使用chars方法和迭代器或char_indices方法。

```rust
let s = "hello";
let c = s.chars().nth(0).unwrap();
```

46. **不合理地分割字符串**

```rust
let s = "hello world";
let first_word: &str = &s[..5]; // 可能截断了字符边界
```

解决方案：确保使用字符边界来分割。

```rust
let s = "hello world";
let space_index = s.find(' ').unwrap_or(s.len());
let first_word: &str = &s[..space_index];
```

47. **尝试直接拼接字符串 slice 和字符串引用**

```rust
let s1 = "hello";
let s2 = "world";
let s3 = s1 + &s2; // error，`+`运算符后面的字符串必须是`&str`类型
```

解决方案：确保第一个操作数使用String，后续的可以是&str。

```rust
let s1 = "hello";
let s2 = "world";
let s3 = s1.to_string() + &s2;
```

48. **使用了错误的字符拼接方法**

```rust
let mut s = String::from("hello");
s.push('w', 'o', 'r', 'l', 'd'); // error
```

解决方案：正确使用push方法来添加单个字符。

```rust
let mut s = String::from("hello");
for c in ['w', 'o', 'r', 'l', 'd'].iter() {
    s.push(*c);
}
```

49. **忽略了字符串的UTF-8编码特性而导致错误的截取**

```rust
let s = "你好世界";
let s1 = &s[0..3]; // error: 可能panic因为不一定是字符边界
```

解决方案：使用chars()方法和相关的迭代器来正确处理字符。

```rust
let s = "你好世界";
let s1: String = s.chars().take(2).collect();
```

50. **在字符串字面量中使用转义序列错误**

```rust
let s = "Hello, \nWorld"; // 不是错误，但可能是理解错误
```

解决方案：如果不想让\n被解析为换行符，可以使用原始字符串。

```rust
let s = r"Hello, \nWorld";
```

51. **将String和&str混淆**

```rust
fn takes_str(s: &str) {}

let s = String::from("hello");
takes_str(s); // error
```

解决方案：传递字符串引用。

```rust
fn takes_str(s: &str) {}

let s = String::from("hello");
takes_str(&s);
```