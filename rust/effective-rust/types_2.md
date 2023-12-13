
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
