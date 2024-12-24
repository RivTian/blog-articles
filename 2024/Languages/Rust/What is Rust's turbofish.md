#Rust 

Have you ever heard about the “turbofish”? It is that piece of Rust syntax that looks like `::<SomeType>`. In this post I will describe what it does and how to use it.

First of, if you were to write something like this:

```rust
fn main() {
    let numbers: Vec<i32> = vec![
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
    ];

    let even_numbers = numbers
        .into_iter()
        .filter(|n| n % 2 == 0)
        .collect();

    println!("{:?}", even_numbers);
}
```

The compiler would yell at you with the following message:

```
$ cargo check
    Checking blog-post v0.1.0 (/Users/davidpdrsn/Desktop/blog-post)
error[E0282]: type annotations needed
 --> src/main.rs:6:9
  |
6 |     let even_numbers = numbers
  |         ^^^^^^^^^^^^
  |         |
  |         cannot infer type
  |         consider giving `even_numbers` a type

error: aborting due to previous error

For more information about this error, try `rustc --explain E0282`.
error: Could not compile `blog-post`.

To learn more, run the command again with --verbose.
```

What this message says is that it doesn’t know what type you’re trying to “collect” your iterator into. Is it a `Vec`, `HashMap`, `HashSet`, or something else that implements [`FromIterator`](https://doc.rust-lang.org/std/iter/trait.FromIterator.html)?

This can be fixed in two different ways. Either by declaring the type of `even_numbers` when you declare the variable:

```rust
let even_numbers: Vec<i32> = ...
```

Or by using a turbofish:

```rust
let even_numbers = numbers
    .into_iter()
    .filter(|n| n % 2 == 0)
    .collect::<Vec<i32>>();
```

The `::<Vec<i32>>` part is the turbofish and means “collect this iterator into a `Vec<i32>`”.

You can actually replace `i32` with `_` and let the compiler infer it.

```rust
let even_numbers = numbers
    .into_iter()
    .filter(|n| n % 2 == 0)
    .collect::<Vec<_>>();
```

The compiler is able to do that because it knows the iterator yields `i32`s.

With this change our final program looks like this:

```rust
fn main() {
    let numbers: Vec<i32> = vec![
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
    ];

    let even_numbers = numbers
        .into_iter()
        .filter(|n| n % 2 == 0)
        .collect::<Vec<_>>();

    println!("{:?}", even_numbers);
}
```

Now the compiler is happy:

```
$ cargo check
    Checking blog-post v0.1.0 (/Users/davidpdrsn/Desktop/blog-post)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
```

## When the turbofish can be used

Lets look at another, similar example:

```rust
fn main() {
    let s = "Hello, World!";
    let string = s.into();
}
```

Compiling this gives us an error similar to what we had before:

```
$ cargo check
    Checking blog-post v0.1.0 (/Users/davidpdrsn/Desktop/blog-post)
error[E0282]: type annotations needed
 --> src/main.rs:3:9
  |
3 |     let string = s.into();
  |         ^^^^^^
  |         |
  |         cannot infer type
  |         consider giving `string` a type

error: aborting due to previous error

For more information about this error, try `rustc --explain E0282`.
error: Could not compile `blog-post`.

To learn more, run the command again with --verbose.
```

We might assume that we fix this by using the turbofish again like so:

```rust
fn main() {
    let s = "Hello, World!";
    let string = s.into::<String>();
}
```

However that gives us a new error:

```
$ cargo check
    Checking blog-post v0.1.0 (/Users/davidpdrsn/Desktop/blog-post)
error[E0107]: wrong number of type arguments: expected 0, found 1
 --> src/main.rs:3:27
  |
3 |     let string = s.into::<String>();
  |                           ^^^^^^ unexpected type argument

error: aborting due to previous error

For more information about this error, try `rustc --explain E0107`.
error: Could not compile `blog-post`.

To learn more, run the command again with --verbose.
```

When I was still new to Rust this error confused me a lot. Why does `::<>` work on `collect` but not on `into`? The answer is in the type signatures of those two functions.

`collect` looks like this:

```rust
fn collect<B>(self) -> B 
```

And `into` looks like this:

```rust
fn into(self) -> T
```

Notice that `collect` is written as `fn collect<B>` and `into` is written as just `fn into`.

The fact that `collect` has a generic type `<B>` is what allows you to use `::<>`. Had `into` somehow been written as `fn into<B>` then you would have been able to write `.into::<String>()`, but since it isn’t, you can’t.

If you encountered a function like `fn foo<A, B, C>()` then you would be able to call it like `foo::<String, i32, f32>()`.

The turbofish can also be applied to other things such as structs with `SomeStruct::<String>::some_method()`. This will work if the struct is defined as `struct SomeStruct<T> { ... }`.

You can almost think of the things inside `(` and `)` to be the “value arguments” to a function and the things inside `::<` and `>` to be the “type arguments”.

So to fix our code from above we would have to declare the type when declaring the variable:

```rust
fn main() {
    let s = "Hello, World!";
    let string: String = s.into();
}
```

And again the compiler is now happy:

```
$ cargo check
    Finished dev [unoptimized + debuginfo] target(s) in 0.04s
```

We can technically also use [the fully qualified trait syntax](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name):

```rust
fn main() {
    let s = "Hello, World!";
    let string = <&str as Into<String>>::into(&s);
}
```

Or optionally

```rust
fn main() {
    let s = "Hello, World!";
    let string = Into::<String>::into(s);
}
```

I would personally just use give `s` a type check declaring the variable in this case.

I recommend you check out the [the book](https://doc.rust-lang.org/book/ch10-00-generics.html) if you want to learn more about how generics work in Rust.

By the way after some digging I found [this reddit comment](https://www.reddit.com/r/rust/comments/3fimgp/why_double_colon_rather_that_dot/ctozkd0/) to be the origin on the word “turbofish”.