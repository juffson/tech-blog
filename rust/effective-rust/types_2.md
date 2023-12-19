
## 使用 rust 基础类型表达通用的行为

上一小节推荐用 rust 的基础类型来表示数据结构，我们常说的一句话 程序 = 数据结构 + 算法，因此也推荐用基础的类型来表现通用的行为
rust 没有像 C++/java 有明确的 class 关键字和相关的官方设计，因此在封装通用行为上略有不同。
### 方法（methods）
如果我们对数据结构增加函数，类似面向对象的编程中，增加对象的行为，就是方法，rust 中虽然没有明确的 class 关键字，但是可以通过关键字`impl`来实现方法
   ```
         enum Shape {
         Rectangle { width: f64, height: f64 },
         Circle { radius: f64 },
      }

      impl Shape {
         pub fn area(&self) -> f64 {
            match self {
                  Shape::Rectangle { width, height } => width * height,
                  Shape::Circle { radius } => std::f64::consts::PI * radius * radius,
            }
         }
      }
   ```
   方法的名称为其编码的行为提供标签，方法签名为其输入和输出提供类型信息。方法的第一个输入应当是 self 变量，表示该方法通过数据对象来调用处理对象中的内容：

      - &self 参数表示数据结构的内容可以读取，但不会被修改。
      - &mut self 参数表示该方法可能会修改数据结构的内容。
      - self 参数表示该方法需要一个数据结构。

   如果没有传入 self，我们可以用类似静态方法的方式调用：Shape::func(arg1, arg2);
### 抽象
调用方法总是导致执行相同的代码；从调用到调用的所有变化都是该方法操作的数据。这涵盖了许多可能的场景，但如果代码需要在运行时发生变化怎么办？

Rust 在其类型系统中包括几个功能来适应这一点，本节将探讨这些功能。

#### 函数指针
最简单的行为抽象是函数指针：指向一些代码的指针，其类型反映了函数的签名。类型在编译时被检查，因此当程序运行时，该值只是指针的大小。
```
    fn sum(x: i32, y: i32) -> i32 {
        x + y
    }
    // Explicit coercion to `fn` type is required...
    let op: fn(i32, i32) -> i32 = sum;
```
函数指针没有与之相关的其他数据，因此它们以值的方式当作变量传递：

```
    // `fn` types implement `Copy`
    let op1 = op;
    let op2 = op;
    // `fn` types implement `Eq`
    assert!(op1 == op2);
    // `fn` implements `std::fmt::Pointer`, used by the {:p} format specifier.
    println!("op = {:p}", op);
    // Example output: "op = 0x101e9aeb0"
```

#### 闭包

裸函数指针是限制的，因为调用函数唯一可用的输入是那些作为参数值显式传递的输入。

例如，考虑一些使用函数指针修改切片每个元素的代码。

    // In real code, an `Iterator` method would be more appropriate.
    pub fn modify_all(data: &mut [u32], mutator: fn(u32) -> u32) {
        for value in data {
            *value = mutator(*value);
        }
    }
这适用于可变的切片：

        fn add2(v: u32) -> u32 {
            v + 2
        }
        let mut data = vec![1, 2, 3];
        modify_all(&mut data, add2);
        assert_eq!(data, vec![3, 4, 5,]);
然而，如果修改依赖于任何额外的状态，则不可能隐式地将其传递到函数指针中。

```
        let amount_to_add = 3;
        fn add_n(v: u32) -> u32 {
            v + amount_to_add
        }
        let mut data = vec![1, 2, 3];
        modify_all(&mut data, add_n);
        assert_eq!(data, vec![3, 4, 5,]);
```
```
error[E0434]: can't capture dynamic environment in a fn item
   --> use-types-behaviour/src/main.rs:142:17
    |
142 |             v + amount_to_add
    |                 ^^^^^^^^^^^^^
    |
    = help: use the `|| { ... }` closure form instead
```
错误消息指出了支持这个调用的正确方式：闭包。闭包是看起来像函数定义（lambda 表达式）的代码块，除了：

- 它可以作为表达式的一部分构建，因此不需要命名
- 输入参数以垂直条|param1, param2|给出（其相关类型通常可以由编译器自动推导出）
- 它可以捕获周围的部分环境。
```
    let amount_to_add = 3;
    let add_n = |y| {
        // a closure capturing `amount_to_add`
        y + amount_to_add
    };
    let z = add_n(5);
    assert_eq!(z, 8);
```
为了（大致）了解捕获的工作原理，想象一下编译器创建一个一次性的内部类型，其中包含 lambda 表达式中提及的所有环境部分。创建闭包时，将创建此临时类型的实例以保存相关值，当调用闭包时，该实例将用作附加上下文。
```
    let amount_to_add = 3;
    // *Rough* equivalent to a capturing closure.
    struct InternalContext<'a> {
        // references to captured variables
        amount_to_add: &'a u32,
    }
    impl<'a> InternalContext<'a> {
        fn internal_op(&self, y: u32) -> u32 {
            // body of the lambda expression
            y + *self.amount_to_add
        }
    }
    let add_n = InternalContext {
        amount_to_add: &amount_to_add,
    };
    let z = add_n.internal_op(5);
    assert_eq!(z, 8);
```
在这个名义上下文中保存的值通常是引用（项目 9），但它们也可以是对环境中变量的可变引用，或者完全移出环境的作用域（在输入参数之前使用 move 关键字）。

回到 modify_all 示例，不能在需要函数指针的地方使用闭包。
```
error[E0308]: mismatched types
   --> use-types-behaviour/src/main.rs:165:31
    |
165 |         modify_all(&mut data, |y| y + amount_to_add);
    |                               ^^^^^^^^^^^^^^^^^^^^^ expected fn pointer, found closure
    |
    = note: expected fn pointer `fn(u32) -> u32`
                  found closure `[closure@use-types-behaviour/src/main.rs:165:31: 165:52]`
note: closures can only be coerced to `fn` types if they do not capture any variables
   --> use-types-behaviour/src/main.rs:165:39
    |
165 |         modify_all(&mut data, |y| y + amount_to_add);
    |                                       ^^^^^^^^^^^^^ `amount_to_add` captured here
```
相反，接收闭包的代码必须接受 Fn*特征之一的实例。
```
    pub fn modify_all<F>(data: &mut [u32], mut mutator: F)
    where
        F: FnMut(u32) -> u32,
    {
        for value in data {
            *value = mutator(*value);
        }
    }
```
Rust 有三种不同的 Fn*特征，它们之间表达了围绕这种环境捕获行为的一些区别。

- FnOnce 描述一个只能调用一次的闭包。如果其环境的某些部分被 move 关闭状态，那么该 move 只能发生一次——没有其他源项目的副本可以 move——因此关闭只能调用一次。
- FnMut 描述了一个可以重复调用的闭包，它可以改变其环境，因为它可以相互借用环境。
- Fn 描述了一个可以重复调用的闭包，它只能不可改变地从环境中借用值。
编译器自动为代码中的任何 lambda 表达式实现这些 Fn*特征的适当子集；无法手动实现任何这些特征 1（与 C++ 的 operator() 重载不同）。

回到上述闭包的粗略模型，编译器自动实现的哪些特征大致对应于捕获的环境环境是否具有：

- FnOnce：任何移动值
- FnMut：对值的任何可变引用（&mut T）
- Fn：仅对值（&T）的正常引用。

上面列表中的后两个特征都具有前一个特征的特质约束，当你考虑使用闭包的东西时，这些都是很重要。

- 如果只希望调用一次闭包（通过接收 FnOnce 表示），则可以传递一个能够重复调用的闭包（FnMut）。
- 如果希望重复调用可能使其环境改变的闭包（通过接收 FnMut 表示），可以向其传递一个不可变其环境（Fn）的闭包。

裸函数指针类型 fn 也名义上属于上述特征的一部分；任何（unsafe）fn 类型都会自动实现所有 Fn*特征，因为它不从环境中借用任何东西。

因此，在编写接受闭包的代码时，**使用最通用的 Fn 特性**  ，为调用者提供最大的灵活性——例如，对于只使用一次的闭包，接受 FnOnce。

