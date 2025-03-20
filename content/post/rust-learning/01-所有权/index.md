---
title: "Rust学习笔记01 - 所有权"
description:
date: "2024-12-09T12:46:41+08:00"
slug: "ownership"
image: ""
license: false
hidden: false
comments: false
draft: false
tags: ["Rust", "所有权"]
categories: ["Rust"]
# weight: 1 # You can add weight to some posts to override the default sorting (date descending)
---

## 所有权，ownership

[What is Ownership? - The Rust Programming Language (rust-lang.org)](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html)

### 所有权规则

- Rust 中的每一个值都有一个 **所有者**（_owner_）
- 值在任一时刻有且只有一个所有者
- 当所有者（变量）离开作用域，这个值将被丢弃，（例如，函数执行完）

### 变量作用域

变量的作用域从声明开始，到最后一次使用的地方结束

```rust
{
    // s 在这里无效，它尚未声明
    let s = "hello"; // 从此处起，s 是有效的
    println!("{}", s); // 使用 s
} // 此作用域已结束，s 不再有效
```

> [!tip]
>
> - 当 `s` **进入作用域** 时，它就是有效的
> - 这一直持续到它 **离开作用域** 为止

### 内存与分配

在有 **垃圾回收**（_garbage collector_，_GC_）的语言中，GC 记录并清除不再使用的内存，而我们并不需要关心它。
在大部分没有 GC 的语言中，识别出不再使用的内存并调用代码显式释放就是我们的责任了，跟请求内存的时候一样。
从历史的角度上说正确处理内存回收曾经是一个困难的编程问题。

- 如果忘记回收了会浪费内存
- 如果过早回收了，将会出现无效变量
- 如果重复回收，这也是个 bug、
- 我们需要精确的为一个 `allocate` 配对一个 `free`。

**Rust 采取了一个不同的策略：内存在拥有它的变量离开作用域后就被自动释放。**

当变量离开作用域，Rust 为我们调用一个特殊的函数。这个函数叫做 [`drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop)。
在这里 `String` 的作者可以放置释放内存的代码，Rust 在结尾的 `}` 处自动调用 `drop`。

```rust
{
 let s = String::from("hello"); // 从此处起，s 是有效的

 // 使用 s
}                                  // 此作用域已结束，
           // s 不再有效
```

> [!warning]
> 在 C++ 中，这种 item 在生命周期结束时释放资源的模式有时被称作 **资源获取即初始化**（_Resource Acquisition Is Initialization (RAII)_）

#### 移动: 变量与数据交互的方式

在 Rust 中，多个变量可以采取不同的方式与同一数据进行交互。

将变量 `x` 的整数值赋给 `y`：

- 将 `5` 绑定到 `x`
- 将 `x` 的拷贝并绑定到 `y`
- 整数是有已知固定大小的简单值，所以这两个 `5` 被放入了栈中

```rust
let x = 5;
let y = x;
```

现在看看这个 `String` 版本：
第二行可能会生成一个 `s1` 的拷贝并绑定到 `s2` 上。不过，事实上并不完全是这样

```rust
let s1 = String::from("hello");
let s2 = s1;
```

`String` 由三部分组成

- 左侧所示：
  - `ptr`，指向存放字符串内容内存的指针
  - `len`，长度，表示当前 `String` 内容使用了多少字符数
  - `capacity`，容量，表示当前 `String` 内容从分配器总共获取了多少字节的内存
  - `capacity >= len`，`capacity` 包括了为字符串内容预留的内存量，即使在字符串为空时也是如此
- 这一组数据存储在栈上，右侧则是堆上存放内容的内存部分
![[1.学习/1.开发语言/Rust/assets/trpl04-01.svg|300]]

当我们将 `s1` 赋值给 `s2`，`String` 的数据被**复制**了，这意味着我们从栈上拷贝了它的指针、长度和容量。
我们并没有复制指针指向的堆上数据。
![[1.学习/1.开发语言/Rust/assets/trpl04-02.svg|300]]

如果 Rust 也**拷贝**了堆上的数据，那么内存看起来就是这样的。
如果 Rust 这么做了，那么操作 `s2 = s1` 在堆上数据比较大的时候会对运行时性能造成非常大的影响。
![[1.学习/1.开发语言/Rust/assets/trpl04-03.svg|300]]

> [!WANING]
> 当变量离开作用域后，Rust 自动调用 `drop` 函数并清理变量的堆内存。
> 复制，两个数据指针指向了同一位置。
> 这就有了一个问题：当 `s2` 和 `s1` 离开作用域，它们都会尝试释放相同的内存。
> 这是一个叫做 **二次释放**（_double free_）的错误，也是之前提到过的内存安全性 bug 之一。
> 两次释放（相同）内存会导致内存污染，它可能会导致潜在的安全漏洞。

为了确保内存安全，在 `let s2 = s1;` 之后，Rust 认为 `s1` 不再有效，因此 Rust 不需要在 `s1` 离开作用域后清理任何东西。
这段代码不能运行：

```rust
let s1 = String::from("hello");
let s2 = s1;
println!("{s1}, world!");
```

你会得到一个类似如下的错误，因为 Rust 禁止你使用无效的引用。

```bash nums
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
  --> kafka-producer/src/main.rs:16:15
   |
14 |     let s1 = String::from("hello");
   |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
15 |     let s2 = s1;
   |              -- value moved here
16 |     println!("{s1}, world!");
   |               ^^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
   |
15 |     let s2 = s1.clone();
   |                ++++++++

For more information about this error, try `rustc --explain E0382`.
warning: `kafka-producer` (bin "kafka-producer") generated 1 warning
error: could not compile `kafka-producer` (bin "kafka-producer") due to 1 previous error
```

> [!TIP]
> 如果你在其他语言中听说过术语 **浅拷贝**（_shallow copy_）和 **深拷贝**（_deep copy_），那么拷贝指针、长度和容量而不拷贝数据可能听起来像浅拷贝。
> 不过因为 Rust 同时使第一个变量无效了，这个操作被称为 **移动**（_move_），而不是叫做浅拷贝。
> 上面的例子可以解读为 `s1` 被 **移动** 到了 `s2` 中。
> ![[1.学习/1.开发语言/Rust/assets/trpl04-04.svg|300]]

这样就解决了我们的问题！因为只有 `s2` 是有效的，当其离开作用域，它就释放自己的内存，完毕。

另外，这里还隐含了一个设计选择：Rust 永远也不会自动创建数据的 “深拷贝”。
因此，任何 **自动** 的复制都可以被认为是对运行时性能影响较小的。

#### 克隆: 变量与数据交互的方式

如果 **确实** 需要深度复制 `String` 中堆上的数据，而不仅仅是栈上的数据，可以使用一个叫做 `clone` 的通用函数。

这是一个实际使用 `clone` 方法的例子：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {s1}, s2 = {s2}");
```

这段代码能正常运行，这里堆上的数据 **确实** 被复制了。
![[1.学习/1.开发语言/Rust/assets/trpl04-03.svg|300]]

#### 拷贝: 只在栈上的数据

```rust
let x = 5;
let y = x;
println!("x = {x}, y = {y}");
```

但这段代码似乎似乎和上面的内容相矛盾：没有调用 `clone`，不过 `x` 依然有效且没有被移动到 `y` 中。

原因是像整型这样的在编译时已知大小的类型被整个存储在栈上，所以拷贝其实际的值是快速的。**这意味着没有理由在创建变量 `y` 后使 `x` 无效**。
换句话说，这里没有深浅拷贝的区别，所以这里调用 `clone` 并不会与通常的浅拷贝有什么不同，可以不用管它。

Rust 有一个叫做 `Copy` trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上。
如果一个类型实现了 `Copy` trait，那么一个旧的变量在将其赋值给其他变量后仍然可用。

Rust 不允许自身或其任何部分实现了 `Drop` trait 的类型使用 `Copy` trait。
如果我们对其值离开作用域时需要特殊处理的类型使用 `Copy` 注解，将会出现一个编译时错误。

那么哪些类型实现了 `Copy` trait 呢？你可以查看给定类型的文档来确认，不过作为一个通用的规则，任何一组简单标量值的组合都可以实现 `Copy`，任何不需要分配内存或某种形式资源的类型都可以实现 `Copy` 。如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也都实现 `Copy` 的时候。比如，`(i32, i32)` 实现了 `Copy`，但 `(i32, String)` 就没有。

### 所有权与函数

将值传递给函数与给变量赋值的原理相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，
                                    // 所以在后面可继续使用 x

} // 这里，x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 没有特殊之处

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。
  // 占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。没有特殊之处
```

当尝试在调用 `takes_ownership` 后使用 `s` 时，Rust 会抛出一个编译时错误。这些静态检查使我们免于犯错。

### 返回值与作用域

返回值也可以转移所有权。

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 转移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中，
                                        // 它也将返回值移给 s3
} // 这里，s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 离开作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 会将
                                             // 返回值移动给
                                             // 调用它的函数

    let some_string = String::from("yours"); // some_string 进入作用域。

    some_string                              // 返回 some_string
                                             // 并移出给调用的函数
                                             //
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域
                                                      //

    a_string  // 返回 a_string 并移出给调用的函数
}
```

变量的所有权总是遵循相同的模式：将值赋给另一个变量时移动它。
当持有堆中数据值的变量离开作用域时，其值将通过 `drop` 被清理掉，除非数据被移动为另一个变量所有。

虽然这样是可以的，但是在每一个函数中都获取所有权并接着返回所有权有些啰嗦。如果我们想要函数使用一个值但不获取所有权该怎么办呢？如果我们还要接着使用它的话，每次都传进去再返回来就有点烦人了，除此之外，我们也可能想返回函数体中产生的一些数据。

我们可以使用元组来返回多个值。

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
```

但是这未免有些形式主义，而且这种场景应该很常见。幸运的是，Rust 对此提供了一个不用获取所有权就可以使用值的功能，叫做 **引用**（_references_）。
