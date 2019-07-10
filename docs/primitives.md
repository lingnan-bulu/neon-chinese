---
id: primitives
title: Primitives
sidebar_label: Primitives
---

[Examples](https://github.com/neon-bindings/examples/tree/master/primitives)

## Numbers

请注意，所有数字都必须转换为f64，因为这些是JS引擎支持的唯一数字类型

```rust
fn js_number(mut cx: FunctionContext) -> JsResult<JsNumber> {
    // Either use function context to create number
    let number = cx.number(23 as f64);
    // or use JsNumber struct
    Ok(number)
}
```

其他基本数据类型遵循类似的模式

## Strings

```rust
// --snip--
let string = cx.string("foobar");
// --snip--
```

## Booleans

```rust
// --snip--
let boolean = cx.boolean(true);
// --snip--
```

## Undefined

```rust
// --snip--
let undefined = cx.undefined();
// --snip--
```

## Null

```rust
// --snip--
let null = cx.null();
// --snip--
```
