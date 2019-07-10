---
id: errors
title: Errors
sidebar_label: Errors
---

[Examples](https://github.com/neon-bindings/examples/tree/master/errors)

Neon支持在JS中创建和抛出所有Error对象。 这些对象包括Error，TypeError和RangeError。 在Neon中调用panic！（）将在节点中抛出错误。 所以panic！（“program errored!”）相当于throw new Error（''program errored!''）。

## Creating Errors

 `FunctionContext` trait 用于创建和抛出错误.

## Catching Errors

工作正在进行中
