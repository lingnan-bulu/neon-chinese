---
id: objects
title: Objects
sidebar_label: Objects
---

[Examples](https://github.com/neon-bindings/examples/tree/master/objects)

JsObject用于在JS堆上创建对象。 JsObject结构有两种方法：get和set

## 获得属性

`.get()` 在运行时获得object的属性:

```rust
// --snip--
let js_object = JsObject::new(&mut cx);
js_object
    .get(&mut cx, "myProperty")?
    .downcast::<JsFunction>()
    .or_throw(&mut cx)?;
// --snip--
```

.downcast（）将尝试将属性转换为JsFunction。 .or_throw（）如果无法downcast，则会出错。

## 设置属性

set（）需要一个FunctionContext，要设置的属性的名称，以及要将属性设置为的值：

```rust
let js_object = JsObject::new(&mut cx);
let js_string = cx.string("foobar");

js_object.set(&mut cx, "myProperty", js_string)?;
```

.将尝试将该权利转移到JsFunction。 .or_throw（）如果无法downcast，则会出错

## 将struct映射到JsObjec

下面是使用JsObject将Rust struct 转换为JS object 的简单示例。我们首先定义struct：

```rust
struct Foo {
    pub bar: u64,
    pub baz: String
}
```
然后定义一个创建Foo结构实例的函数
```rust
fn convert_struct_to_js_object(mut cx: FunctionContext) -> JsResult<JsObject> {
    let foo = Foo {
        bar: 1234,
        baz: "baz".to_string()
    };
    let object = JsObject::new(&mut cx);
    let js_string = cx.string(&foo.baz);
    let js_number = cx.number(foo.bar as f64);
    object.set(&mut cx, "myStringProperty", js_string).unwrap();
    object.set(&mut cx, "myNumberProperty", js_number).unwrap();
    Ok(object)
}

register_module!(mut m, {
    m.export_function("convertStructToJsObject", convert_struct_to_js_object)
});
```
