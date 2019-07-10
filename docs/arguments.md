---
id: arguments
title: Arguments
sidebar_label: Arguments
---

[Examples](https://github.com/neon-bindings/examples/tree/master/arguments)

Neon提供了用于访问arguments对象的内置机制。

参数可以从JS传递给Rust，可以是任何类型。判断某些值是某些类型是有用的

## 用索引执行function

我们首先定义一个函数并以sayHi的名称导出它：

```rust
fn say_hi(mut cx: FunctionContext) {}

register_module!(mut m, {
    m.export_function("sayHi", say_hi)
});
```

以下代码将第一个参数传递给sayHi函数，如果无法强制转换为函数则抛出该函数

```rust
fn say_hi(mut cx: FunctionContext) -> JsResult<JsFunction> {
    let arg0 = cx.argument::<JsFunction>(0)?.value();
    // --snip--
}
```

## 断言参数类型

```rust
pub fn foo(mut cx: FunctionContext) -> JsResult<JsUndefined> {
    cx.check_argument::<JsString>(0)?;
    cx.check_argument::<JsNumber>(1)?;
    Ok(cx.undefined())
}
```

Now in our `./lib/index.js`:

```js
const { foo } = require('../native');
foo(); // fails
foo(12); // fails
foo('foobar'); // fails
foo('foobar', 12); // passes!
```

## 获取参数的值

```rust
fn add1(mut cx: FunctionContext) -> JsResult<JsNumber> {
    // Attempt to cast the first argument to a JsNumber. Then
    // get the value if cast is successul
    let x = cx.argument::<JsNumber>(0)?.value();
    Ok(cx.number(x + 1.0))
}

register_module!(mut m, {
    m.export_function("add1", add1)
});
```

## Getting the Number of Arguments

This is a simple example of getting the length of `arguments`

```rust
pub fn get_args_len(mut cx: FunctionContext) -> JsResult<JsNumber> {
    let args_length = cx.len();
    println!("{}", args_length);
    Ok(cx.number(args_length))
}

register_module!(mut m, {
    m.export_function("getArgsLen", get_args_len)
});
```

Now in our `./lib/index.js` we add the following:

```js
// ./lib/index.js
const { getArgsLen } = require('../native');
getArgsLen(); // 0
getArgsLen(1); // 1
getArgsLen(1, 'foobar'); // 2
```

## Optional Arguments

Produces the `i`th argument, or `None` if `i` is greater than or equal to `self.len()`.

```rust
pub fn args_opt(mut cx: FunctionContext) -> JsResult<JsNumber> {
    match cx.argument_opt(0) {
        Some(arg) => {
            // Throw if the argument exist and it cannot be downcasted
            // to a number
            let num = arg.downcast::<JsNumber>().or_throw(&mut cx)?.value();
            println!"The 0th argument is {}", num);
        },
        None => panic!("0th argument does not exist, out of bounds!")
    }
    Ok(cx.undefined())
}
```

## Default Values

Handling default values is similar to handling **Optional Arguments**:

```rust
// --snip--

pub fn default_args(mut cx: FunctionContext) -> JsResult<JsUndefined> {
    let age = match cx.argument_opt(0) {
        Some(arg) => arg.downcast::<JsNumber>().or_throw(&mut cx)?.value(),
        // Default to 12 if no value is given
        None => 12 as f64
    };

    let name = match cx.argument_opt(1) {
        Some(arg) => arg.downcast::<JsString>().or_throw(&mut cx)?.value(),
        // Default to 12 if no value is given
        None => "John Doe".to_string()
    };

    println!("i am {} years old and my name is {}", age, name);

    Ok(cx.undefined())
}
```

Here's how we'd call those functions:

```js
// ./lib/index.js
const { defaultArgs } = require('../native');

defaultArgs(); // i am 12 years old and my name is John Doe
defaultArgs(22); // i am 22 years old and my name is John Doe
defaultArgs(22, 'Jane Doe'); // i am 22 years old and my name is Jane Doe
```
