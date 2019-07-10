---
id: json
title: JSON
sidebar_label: JSON
---

[Examples](https://github.com/neon-bindings/examples/tree/master/json)

有时你需要将rust的struct 转化成 jsObject 使用并返回 你可以借助[neon-serde](https://github.com/GabrielCastro/neon-serde)实现. 你需要安装 `neon-serde` 和 `serde_derive` 包.

`neon-serde` 更多的文档, 请查看 [repo](https://github.com/GabrielCastro/neon-serde) 和 [docs](https://docs.rs/crate/neon-serde/0.0.3)

首先我们先安装这些包:

```toml
# Cargo.toml
# --snip--
[dependencies]
neon = "0.2.0"
neon-serde = "0.1.1"
serde_derive = "1.0.80"
serde = "1.0.80"
```

然后导入必要的库并声明一个用serde标记为反序列化的User结构 [serde](https://github.com/serde-rs/serde):

```rust
#[macro_use]
extern crate neon;
#[macro_use]
extern crate neon_serde;
#[macro_use]
extern crate serde_derive;

#[derive(Serialize)]
struct User {
    name: String,
    age: u16,
}
```

## Serializing

我们可以序列化rust的struct为jsvalue 

```rust
// --snip--
fn serialize_something(mut cx: FunctionContext) -> JsResult<JsValue> {
    let value = AnObject {
        a: 1,
        b: vec![2f64, 3f64, 4f64],
        c: "a string".into()
    };

    let js_value = neon_serde::to_value(&mut cx, &value)?;
    Ok(js_value)
}

register_module!(mut m, {
    m.export_function("serialize_something", serialize_something)
});
```

In your `./lib/index.js` you can call your function like so:

```js
const addon = require('../native');
addon.serialize_something();
```

## Deserializing

将 `User` trait 改为可反序列化:

```rust
// --snip--
#[derive(Serialize, Deserialize)]
struct User {
    name: String,
    age: u16,
}
// --snip--
```

现在我们也可以反序列化一个JsObject结构并将其转换为JsValue，如下所示：
```rust
// --snip--
fn deserialize_something(mut cx: FunctionContext) -> JsResult<JsValue> {
    let arg0 = cx.argument::<JsValue>(0)?;

    let arg0_value: AnObject = neon_serde::from_value(&mut cx, arg0)?;
    println!("{:?}", arg0_value);

    Ok(cx.undefined().upcast())
}

register_module!(mut m, {
    m.export_function("serialize_something", serialize_something)?;
    m.export_function("deserialize_something", deserialize_something)?;
    Ok(())
});
```

你可以执行你的方法 :

```js
const addon = require('../native');
addon.deserialize_something();
```

## Macros

neon-serde提供了一些宏来简化函数的某些类型签名。它还处理导出我们的函数，因此我们不必使用register_module！手动宏

```rs
#[macro_use]
extern crate neon;
#[macro_use]
extern crate neon_serde;
#[macro_use]
extern crate serde_derive;

export! {
    fn say_hello(name: String) -> String {
        format!("Hello, {}!", name)
    }

    fn greet(user: User) -> String {
        format!("{} is {} years old", user.name, user.age)
    }

    fn fibonacci(n: i32) -> i32 {
        match n {
            1 | 2 => 1,
            n => fibonacci(n - 1) + fibonacci(n - 2)
        }
    }
}
```
在我们的JS中，我们只需导入方法并调用函数。请注意，宏为我们编写了类型检查：
```js
const addon = require('../native');

// Calling the function with incorrect arguments will fail
// console.log(addon.say_hello());
// fails: TypeError: not enough arguments

console.log(addon.say_hello('john'));
// Hello, john!

// Calling the function with incorrect arguments will fail
// console.log(addon.greet({ name: "afsd" }));
// Error(Msg("missing field `age`"), State { next_error: None, backtrace: None })

console.log(addon.fibonacci(32));
```
