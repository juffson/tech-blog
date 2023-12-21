
## 不要滥用 match 处理 Option 和 Result
###
在第一节提到了 两个普遍使用的 enum 类型 Option 类型和 Result 类型。
第一种没必要使用 match 的情况是，没有值的时候可以忽略
```
 struct S {
        field: Option<i32>,
    }

    let s = S { field: Some(42) };
    match &s.field {
        Some(i) => println!("field is {}", i),
        None => {}
    }
```
这种情况下，直接通过 if let 表达式，代码更加短小、清晰、
```
  if let Some(i) = &s.field {
        println!("field is {}", i);
    }
```

不过大部分的情况下，错误或者空值都是需要处理的，原则是处理时尽量简短，可以参考“鸵鸟原则”：
```
  let result = std::fs::File::open("/etc/passwd");
    let f = match result {
        Ok(f) => f,
        Err(_e) => panic!("Failed to open /etc/passwd!"),
    };
```
上述代码中 match 也是没必要的，针对 panic 的场景，option 和 result 都可以通过 unwrap 和 expect 处理：
```
   let f = std::fs::File::open("/etc/passwd").unwrap();
```
当我们希望 panic 的时候，直接 unwrap 就可以了。多数情况下，我们都不希望我的程序直接 panic，函数设计也是期望上层调用可以处理我们的错误。这种情况，更推荐 Result；即使需要转换错误类型，Result 的方式返回错误也会比 Option 更容易处理。