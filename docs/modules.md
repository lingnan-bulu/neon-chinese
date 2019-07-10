---
id: modules
title: Modules
sidebar_label: Modules
---

[Examples](https://github.com/neon-bindings/examples/tree/master/modules)

在Node中，通常导出值，函数和类。 这些数据结构也可以通过一些额外的步骤从Neon模块导出。
## Exporting Values, Functions, and Classes

Consider the following JS:

```js
// ./my-exports.js
export const foo = 'foo';

export function bar() {}

export class Baz {}
```

The Neon equivalent would be the following: neon 如下；

```rust
// --snip--
fn bar(mut cx: FunctionContext) -> JsResult<JsUndefined> {
    Ok(cx.undefined())
}

pub struct Baz;

declare_types! {
    pub class JsBaz for Baz {
        // --snip--
    }
}

register_module!(mut m, {
    let foo = m.string("foo");
    // Export `foo'
    m.export_value("foo", foo)?;
    // Export the `bar` function
    m.export_function("bar", bar)?;
    // Export the `Baz` class
    m.export_class::<JsBaz>("Baz")?;
    Ok(())
});
```

## Default Exports

Work in Progress
