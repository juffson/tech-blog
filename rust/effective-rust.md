# Effective Rust
## 介绍
> "The code is more what you'd call 'guidelines' than actual rules." – Hector Barbossa

构成这本书的项目分为六个部分：

- 类型：围绕 Rust 的核心类型系统的建议。
- 概念：构成 Rust 设计的核心理念。
- 依赖性：使用 Rust 软件包生态系统的建议。
- 工具：关于如何通过超越 Rust 编译器来改进代码库的建议。
- 异步 Rust：使用 Rust async 机制的建议。
- 超越标准 Rust：当您必须超越 Rust 的标准、安全环境时工作的建议。

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