---
id: type-checking
title: Type Checking
sidebar_label: Type Checking
---

如果我们可以使用可以从JS调用的Rust声明函数，那么我们需要知道传递给参数的参数的类型，以便在Rust中使用。 这就是casting发挥作用的地方。  **Upcasting**使类型不那么具体，而**Downcasting**使类型更具体。 一个JsValue，它代表一个我们不知道其类型的任意JS值。 我们可以将此值转换为更具体的内容，如JsNumber，以便我们可以在Rust中使用它，就像它是一个数字一样。 当我们想要将值传递回JS引擎时，向下转换使用很有用。 有关详细信息，请参阅类部分。


## Upcasting

JS类隐含的每个方法都返回一个JsValue。不能返回比JsValue更具特异性的类型。
例如，以下类方法将无法编译
```rust
declare_types! {
    /// JS class 包装记录
    pub class JsEmployee for Employee {
        method talk(mut cx) {
            Ok(cx.string("Hello").upcast())
        }
    }
}
```
安全地向上转换超类型的句柄。此方法不需要执行上下文，因为它只复制句柄。

## Downcasting

尝试将句柄向下转换为另一种类型，这可能会失败。向下转换失败不会抛出JavaScript异常，因此如果此方法产生错误结果，则继续与JS引擎交互是可以的。

```rust
// --snip
cx.number(17).downcast();
cx.number(17).downcast_or_throw();
// --snip--
```

## 检测类型
测试此值是否是给定类型的实例。

```rust
// --snip--
let v: Handle<JsValue> = cx.number(17).upcast();
v.is_a::<JsString>(); // false
v.is_a::<JsNumber>(); // true
v.is_a::<JsValue>();  // true
// --snip--
```
