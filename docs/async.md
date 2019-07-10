---
id: async
title: Async Tasks
sidebar_label: Async Tasks
---

[Examples](https://github.com/neon-bindings/examples/tree/master/async)

任务允许您执行在Node线程池中运行的异步后台任务。 在后台，Neon使用N-API的Microtasks API。 Microtasks是JS引擎中Promise和Callbacks的后来实现。 有关微任务的更多信息，请参阅 ["Tasks, microtasks, queues and schedules"](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

让我们看一下异步任务的最小实现：

```rust
use neon::prelude::*;

struct BackgroundTask;

impl Task for BackgroundTask {
    type Output = i32;
    type Error = String;
    type JsEvent = JsNumber;

    fn perform(&self) -> Result<Self::Output, Self::Error> {
        Ok(17)
    }

    fn complete(self, mut cx: TaskContext, result: Result<Self::Output, Self::Error>) -> JsResult<Self::JsEvent> {
        Ok(cx.number(result.unwrap()))
    }
}

pub fn perform_async_task(mut cx: FunctionContext) -> JsResult<JsUndefined> {
    let f = cx.argument::<JsFunction>(0)?;
    BackgroundTask.schedule(f);
    Ok(cx.undefined())
}

register_module!(mut m, {
    m.export_function("performAsyncTask", perform_async_task)
});
```

### Output

任务的结果类型，发送回主线程以将成功结果传回JavaScript

### Error

任务的错误类型，它被发送回主线程以将任务失败传回JavaScript。

### JsEvent

完成任务后，主线程上的异步回调生成的JavaScript值类型。

### `.perform()`

执行任务，产生成功的输出或不成功的错误。此方法在后台线程中作为libuv的内置线程池的一部分执行。

### `.complete()`

将任务结果转换为JavaScript值以传递给异步回调。在后台任务完成后的某个时刻，在主线程上执行此方法。

### `.schedule(f)`

安排要在后台线程上执行的任务。回调应具有以下签名：

```js
function callback(err, value) {}
```

## Calling Async Tasks

现在让我们看一下如何使用我们创建的BackgroundTask结构来调整异步任务：

```js
const { performAsyncTask } = require("../native");

// Iterate 10,0000 times in background thread
performAsyncTask((err, value) => {
  let count = 10;
  for (let i = 0; i < 100000; i++) {
    count++;
  }
  console.log(count, "first sum from background thread");
});

// Iterate 10 times
let count = 10;
for (let i = 0; i < 10; i++) {
  count++;
}
console.log(count, "second sum from main thread");
```

如果您运行此代码，您将获得以下结果

```
20 'second sum from main thread'
100010 'first sum from background thread'
```

如果performAsyncTask（）同步执行，那么后台线程将在主线程完成之前完成运行，结果将是：

```
100010 'first sum from background thread'
20 'second sum from main thread'
```

## 处理失败的任务

如果我们希望前面的示例抛出错误，我们可以使用以下内容简单地替换perform和complete方法：

```rust
// --snip--
fn perform(&self) -> Result<Self::Output, Self::Error> {
    Err(format!("I am a failing task"))
}

fn complete(self, mut cx: TaskContext, result: Result<Self::Output, Self::Error>) -> JsResult<Self::JsEvent> {
    cx.throw_error(&result.unwrap_err())
}
// --snip--
```

## Promises

如果我们想围绕一个Promises包装我们的任务，我们可以这样做：

```js
// ./lib/index.js
const { performAsyncTask } = require("../native");

// Iterate 10,0000 times in background thread
const promisePerformAsyncTask = () => {
  return new Promise((resolve, reject) => {
    performAsyncTask((err, res) => {
      let count = 10;
      for (let i = 0; i < 100000; i++) {
        count++;
      }
      console.log(count, "first sum from background thread");
    });
  });
};
```

## Runnable Example

For another example of tasks, you can clone and run [fibonacci-async-task](https://github.com/neon-bindings/examples/tree/master/fibonacci-async-task):

```bash
git clone https://github.com/neon-bindings/examples
cd fibonacci-async-task
npm install
```

This example computes the `100000`th fibonacci number on a background thread while keeping the main thread free
