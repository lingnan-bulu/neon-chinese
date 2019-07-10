---
id: arrays
title: Arrays
sidebar_label: Arrays
---

[Examples](https://github.com/neon-bindings/examples/tree/master/arrays)

## 把 `Vec` 转换为 `JsArray`

这里有一个使用`JsArray`把 `rust` 的 `Vec` 转换为 `JS` 的 `Array` 的简单例子 :

```rust
fn convert_vec_to_array(mut cx: FunctionContext) -> JsResult<JsArray> {
    let vec: Vec<String> = Vec::with_capacity(100);

    // Create the JS array
    let js_array = JsArray::new(&mut cx, vec.len() as u32);

    // Iterate over the rust Vec and map each value in the Vec to the JS array
    for (i, obj) in vec.iter().enumerate() {
        let js_string = cx.string(obj);
        js_array.set(&mut cx, i as u32, js_string).unwrap();
    }

    Ok(js_array)
}
```

## 把 `JsArray` 转换为 `Vec`

把 `JsArray` 转换为 `Vec`很简单:

```rust
fn convert_js_array_to_vec(mut cx: FunctionContext) -> JsResult<JsNumber> {
    // 取得第一个参数 他必须是数组
    let js_arr_handle: Handle<JsArray> = cx.argument(0)?;
    // 转换js的array 为rust vec
    let vec: Vec<Handle<JsValue>> = js_arr_handle.to_vec(&mut cx)?;
    // 给js返回这个vec的长度
    Ok(cx.number(vec.len() as f64))
}
```

## 返回一个空元素

```rust
pub fn return_js_array(mut cx: FunctionContext) -> JsResult<JsArray> {
    Ok(cx.empty_array())
}
```

## 给数组添加一个元素

这是一个给数组添加number的例子

```rust
pub fn return_js_array_with_number(mut cx: FunctionContext) -> JsResult<JsArray> {
    let array: Handle<JsArray> = JsArray::new(&mut cx, 1);
    let n = cx.number(9000.0);
    array.set(&mut cx, 0, n)?;
    Ok(array)
}
```

这是一个给数组添加string的例子

```rust
pub fn return_js_array_with_string(mut cx: FunctionContext) -> JsResult<JsArray> {
    let array: Handle<JsArray> = JsArray::new(&mut cx, 1);
    let s = cx.string("hello node");
    array.set(&mut cx, 0, s)?;
    Ok(array)
}
```

## `ArrayBuffer`
Neon还通过 [`JsArrayBuffer`](https://neon-bindings.com/api/neon/prelude/struct.jsarraybuffer) 结构提供对ES6 [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 的支持。它与JsArray具有相同的构造函数和方法


## Node `Buffer`
Neon还通过[`JsBuffer`](https://neon-bindings.com/api/neon/prelude/struct.jsbuffer)结构支持node 的 Buffer 类型。它与JsArray的构造方法和方法相同

#### 可运行的例子

有关将Node的Buffer类与Neon一起使用的工作示例，请参阅sharing-binary-data示例。您可以通过运行以下命令开始使用它 
```bash
git clone https://github.com/neon-bindings/examples
cd sharing-binary-data
npm install
```
