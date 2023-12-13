# 高效 Rust (Effective Rust)
## 类型（Types）
相比于其他语言的类型，这里有两个核心的类型：


- [**枚举**](https://doc.rust-lang.org/book/ch06-00-enums.html) 相比其他语言，更具有表现力
- [**trait**](https://doc.rust-lang.org/book/ch10-02-traits.html)

### 使用 rust 基础类型表达数据结构
> "who called them programers and not type writers" – thingskatedid

rust 的基础类型对使用过静态类型语言的人来说都非常熟悉（像 C++ 、golang、Java、等）。相比于 C++，rust 的数字类型更加严格。例如尝试把 `i32` 类型赋值给 `i16` 类型会报错：
```
    let x: i32 = 42;
    let y: i16 = x;
```
```
error[E0308]: mismatched types
  --> use-types/src/main.rs:14:22
   |
14 |         let y: i16 = x;
   |                ---   ^ expected `i16`, found `i32`
   |                |
   |                expected due to this
   |
help: you can convert an `i32` to an `i16` and panic if the converted value doesn't fit
   |
14 |         let y: i16 = x.try_into().unwrap();
   |                       ++++++++++++++++++++
```
这是因为 rust 是强规则约束，针对类型的转化，rust 会在编译时检查，相比 C++，在针对安全的转换上，rust 编译时依然会报错，推荐在类型转换时，明确转换的过程。
```
   let x = 42i32; // Integer literal with type suffix
   let y: i64 = x;
```
```
   error[E0308]: mismatched types
   --> use-types/src/main.rs:23:22
      |
   23 |         let y: i64 = x;
      |                ---   ^ expected `i64`, found `i32`
      |                |
      |                expected due to this
      |
   help: you can convert an `i32` to an `i64`
      |
   23 |         let y: i64 = x.into();
      |                       +++++++
```
后文会对类型转换详细说明
#### 聚合类型（Aggregate Types）
rust 有以下聚合的类型
- 数组 (Arrarys), 存储同一类型的多个值，数组的长度是固定的（即在编译时已知），例如[u32; 4] 是 4 个 4 字节整数
- 元组 (Tuples), 存储多个异质（heterogeneous）类型，类型和长度都是在编译时已知。
- 结构体 (Structs),保存编译时已知的异构类型的实例，但允许整体类型和单个字段都用名称引用。

"Tuple struct" 是结构体（struct）和元组（tuple）的结合体：它既有整体类型的名称，但没有为单个字段指定名称 - 它们通过数字来引用，例如 s.0、s.1 等。

在编程中，"tuple struct" 是一种特殊类型的结构体，它的字段没有具体的名称，而是通过索引号来引用。这种设计使得 tuple struct 的字段可以按照顺序进行访问和操作，而不需要使用具体的字段名。这在某些情况下可以提供更灵活和简洁的代码实现方式。
```
   struct TextMatch(usize, String);
   let m = TextMatch(12, "needle".to_owned());
   assert_eq!(m.0, 12);
```
这就引出了 Rust 类型系统中的一颗明珠，即 "enum"（枚举）。

从基本形式来看，可能很难看出它有什么令人兴奋的地方。与其他编程语言类似，"enum" 允许您定义一组互斥的值，可能附带数值或字符串值。

在 Rust 中，"enum" 是一种特殊的数据类型，它允许您定义一组具有不同取值的变体。每个变体可以附带额外的数据，也可以是单独的单元类型。这使得您可以更清晰地表示某个值的可能状态，并且可以根据不同的状态采取不同的逻辑处理。

"enum" 在 Rust 中被广泛用于处理模式匹配、状态转换、错误处理等场景，它是 Rust 强大而灵活的类型系统中的一个重要组成部分。
```
    enum HttpResultCode {
        Ok = 200,
        NotFound = 404,
        Teapot = 418,
    }
    let code = HttpResultCode::NotFound;
    assert_eq!(code as i32, 404);
```
由于每个 enum 定义都创建了一个不同的类型，因此可用于提高 bool 入参的函数的可读性和可维护性。而不是：

```
   print_page(/* both_sides= */ true, /* colour= */ false);
```
比如使用枚举对
```
  pub enum Sides {
        Both,
        Single,
    }

    pub enum Output {
        BlackAndWhite,
        Colour,
    }

    pub fn print_page(sides: Sides, colour: Output) {
        // ...
    }
```
在调用时更类型安全，更容易阅读：
```
        print_page(Sides::Both, Output::BlackAndWhite);
```
对比上面 bool 参数的版本，当用户传错参数顺序时，编译阶段便会发现告警
```
error[E0308]: mismatched types
  --> use-types/src/main.rs:89:20
   |
89 |         print_page(Output::BlackAndWhite, Sides::Single);
   |                    ^^^^^^^^^^^^^^^^^^^^^ expected enum `enums::Sides`, found enum `enums::Output`
error[E0308]: mismatched types
  --> use-types/src/main.rs:89:43
   |
89 |         print_page(Output::BlackAndWhite, Sides::Single);
   |                                           ^^^^^^^^^^^^^ expected enum `enums::Output`, found enum `enums::Sides`
```

#### 带字段的 enums
相比于传统的 enum 类型，rust 的 enum 类型真正强大的地方在于，可以当作 ADT（algebraic data type）使用，对比 C/C++ 中，就像结合了 union 和 enum，并且类型安全。
基于这种模型，数据结构的不变量可以编码到 Rust 的类型系统中；不符合这些不变量的状态甚至不会编译。设计良好的 enum 在可读性和编译性上都有非常大的优势。
```
pub enum SchedulerState {
    Inert,
    Pending(HashSet<Job>),
    Running(HashMap<CpuId, Vec<Job>>),
}
```
仅从类型定义中，可以合理地猜测 Jobs 在 Pending 状态下排队，直到调度器完全处于活动状态，此时它们被分配到一些每个 CPU 池。

这就是该章节的中心主题，即使用 Rust 的类型系统来表达与软件设计相关的概念。

使用 enum 封装后，下文例子中所述的注释其实是不需要的
```
struct DisplayProps {
    x: u32,
    y: u32,
    monochrome: bool,
    // `fg_colour` must be (0, 0, 0) if `monochrome` is true.
    fg_colour: RgbColour,
}
```
使用 enum 封装替换后：
```
#[derive(Debug)]
enum Colour {
    Monochrome,
    Foreground(RgbColour),
}

struct DisplayProperties {
    x: u32,
    y: u32,
    colour: Colour,
}
```
这个小例子说明了一条非常有用的编写技巧：不要在代码中表达无意义的状态。使用有效的组合，通过编译器检查错误，能让代码更精简、更安全。

Options 和 Errors

回到 enum 的作用，有两个概念非常常见，Rust 通过内置 enum 类型来表达。

第一个是 Option 的概念：要么有特定类型的值（Some(T)），要么没有（None）。对于可能不存在的值，始终使用 Option；永远不要退回到使用特殊值（-1，nullptr，...）来尝试表达相同的概念。

不过，有一个特殊点需要考虑。如果你正在处理一个集合，你需要确认集合为空和没有该集合是不是相同。对于大多数情况，没有区别，直接使用 Vec<Thing>, count 为 0 的时候就是没有值。

然而，也有其他罕见的情况，需要用 Option<Vec<Thing>>区分这两种情况——例如，加密系统可能需要区分数据分离 和数据为空。（和 SQL 中的 NULL 类似）

还有一种讨论比较多的情况，空字符串或者是没有值，但在 rust 中 Option<String>能很明确的区分，没有字符串和字符串为空的情况

第二个常见概念源于错误处理：如果函数失败，应该返回错误？历史上，使用了（例如 Linux 系统调用使用 errno 返回值）或全局变量（POSIX 系统的 errno）。或者是像 go 语言一样，支持函数多返回值或元组返回值 (result, error) ，当 error 非“空”时，result 使用一些合适的“空”值。

在 Rust 中，始终将可能失败的操作的结果编码为 Result<T, E>。T 类型持有成功结果（在 Ok 变量中），E 类型在失败时持有错误详细信息（在 Err 变量中）。使用标准类型可以让函数设计意图清晰，并允许使用标准转换（项目 3）和错误处理（项目 4）；还可以通过？操作符号简化错误的处理过程。

