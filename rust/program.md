# Rust 程序设计
## 异常处理
### unwrap
unwrap 是 Rust 标准库中的一个方法，它用于从 Option 或 Result 类型中获取值。unwrap 方法会返回 Some 中的值或 Ok 中的值，如果是 None 或 Err 则会触发 panic。

通常情况下，我们可以在代码中使用 unwrap 方法来表示我们对 Option 或 Result 类型的值非常确信，我们希望在这些值不是 None 或 Err 的情况下获取真正的值。但是需要注意的是，如果 unwrap 方法在 None 或 Err 的情况下被调用，会导致程序崩溃。

因此，建议在使用 unwrap 方法时要确保对应的 Option 或 Result 值不会为 None 或 Err，否则应该使用更安全的方法来处理错误，例如 match 表达式或 unwrap_or 方法。
unwrap_or 是 Rust 标准库中的一个方法，它用于从 Option 或 Result 类型中获取值，但是当值为 None 或 Err 时，它会返回一个备选值作为默认值，而不是触发 panic。

unwrap_or 方法接受一个参数作为默认值，并在 Option 为 None 或 Result 为 Err 时返回该默认值。如果 Option 为 Some 或 Result 为 Ok，则会返回对应的值。

这个方法在处理可能出现错误的情况时非常有用，可以避免程序崩溃。可以根据实际需求选择合适的默认值，例如一个默认的空字符串、零值或者自定义的错误信息。

下面是 unwrap_or 方法的示例代码：
```
fn main() {
    let maybe_value: Option<i32> = None;
    let default_value = 42;

    let value = maybe_value.unwrap_or(default_value);
    println!("Value: {}", value); // Output: Value: 42
}
```
在上面的示例中，maybe_value 是一个 None 值的 Option，通过调用 unwrap_or 方法并传入 default_value 作为默认值，最终得到了 value 的值为 42。

## 包管理
### 使用 features 进行条件编译和版本特性控制
我们可以用 rust 的 features 特性，控制我们希望提供到某个版本时，有哪些特性；同时也可以在引入包时，通过指定 features 来控制这些特性的版本。

#### 定义包的 features

在使用 features 特性时，我们可以在 Cargo.toml 中指定这些特性的版本。这里的 feature（在现在的版本 rustc 1.75.0）中不支持和 package 同名，在 nightly 版本中可以同名
```toml
[features]
my_feature =  []  # 定义一个 feature
my_feature2 = ["my_feature"] # 定义一个 feature，依赖了其他 feature ,开启了该 feature，那么 my_feature 特性也会开启
```


#### 按 features 依赖
在依赖的地方，我们可以通过 Cargo.toml 的包依赖配置按特性开启。

```toml
[dependencies]
my_package = { version = "1.0.0", features = ["my_feature"] }
```

如果我们不想加入到构建版本中，可以通过 options 不进行编译
```toml
[dependencies]
my_package = { version = "1.0.0", features = ["my_feature"], optional = true }
```
