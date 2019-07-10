---
id: classes
title: Classes
sidebar_label: Classes
---

[Examples](https://github.com/neon-bindings/examples/tree/master/classes)

现在，请参考以下测试中的 <a href="https://github.com/neon-bindings/neon/blob/master/test/dynamic/native/src/js/classes.rs" target="_blank">代码片段:</a>

## Basics

让我们创建一个我们的类将使用的简单结构：

```rust
pub struct Employee {
    // Rust struct properties map to JS class properties
    id: i32,
    name: String
}
```

现在让我们定义一个自定义JS类，其实例包含一个Employee记录。 init是JsEmployee对象的构造函数。 我们定义的方法以方法为前缀。 因此，如果我们希望我们的JS对象拥有一个名为alertUser的方法，那么我们的方法签名就是方法alertUser（mut cx）{。 所有方法都需要返回JsValue的类型，因此我们需要对它们进行upcast。 由于此要求，以下方法将失败：

❌ This will not work

```rust
method talk(mut cx) {
    Ok(cx.string("How are you doing?"))
}
```

✅ But this will work:

```rust
method talk(mut cx) {
    Ok(cx.string("How are you doing?").upcast())
}
```

### Defining the `constructor`

Now let's define our class. `init` will construct the class:

```rust
// --snip--
declare_types! {
    /// JS class wrapping Employee records.
    pub class JsEmployee for Employee {
        init(mut cx) {
            let id = cx.argument::<JsNumber>(0)?.value();
            let name: String = cx.argument::<JsString>(1)?.value();

            Ok(Employee {
                id: id as i32,
                name: name,
            })
        }
    }
}

// Export the class
register_module!(mut m, {
    // <JsEmployee> tells neon what class we are exporting
    // "Employee" is the name of the export that the class is exported as
    m.export_class::<JsEmployee>("Employee")?;
    Ok(())
});
```

### Adding Methods

Now let's add some methods to our class:

```rust
// --snip--
init(mut cx) {
    let id = cx.argument::<JsNumber>(0)?.value();
    let name: String = cx.argument::<JsString>(1)?.value();

    Ok(Employee {
        id: id as i32,
        name: name,
    })
}

method name(mut cx) {
    let this = cx.this();
    let name = {
        let guard = cx.lock();
        this.borrow(&guard).name
    };
    println!("{}", &name);
    Ok(cx.undefined().upcast())
}

method greet(mut cx) {
    let this = cx.this();
    let msg = {
        let guard = cx.lock();
        let greeter = this.borrow(&guard);
        format!("Hi {}!", greeter.name)
    };
    println!("{}", &msg);
    Ok(cx.string(&msg).upcast())
}

method askQuestion(mut cx) {
    println!("{}", "How are you?");
    Ok(cx.undefined().upcast())
}
// --snip--
```

Then you can use instances of this type in JS just like any other object:

```js
const { Employee } = require('../native');

console.log(new addon.Employee()); // fails: TypeError: not enough arguments

const john = new addon.Employee('John');
john.name(); // John
john.greet(); // Hi John!
john.askQuestion(); // How are you?
```

由于Employee上的方法希望它具有正确的二进制布局，因此它们会检查以确保不会在不适当的对象类型上调用它们。这意味着您不能通过执行类似的操作来拦截Node

```js
Employee.prototype.name.call({});
```

This safely throws a `TypeError` exception just like methods from other native classes like `Date` or `Buffer` do.

### Getting and Setting Class Properties

```rust
// --snip--
let this = cx.this();
// Downcast the object so we can call .get and .set
let this = this.downcast::<JsObject>().or_throw(&mut cx)?;
let is_raining = this
  .get(&mut cx, "raining")?
  .downcast::<JsBoolean>().or_throw(&mut cx)?
  .value();
if is_raining {
  let t = cx.boolean(false);
  this.set(&mut cx, "shouldGoOutside", t)?;
}
// --snip--
```

### Handling Methods That Take Multiple Types

Sometimes you may want your function to handle arguments that can be of multiple types. Here's an example showing just that:

```rust
// --snip--
method introduce(mut cx) {
    let name_or_age = cx.argument::<JsValue>(0)?;

    if name_or_age.is_a::<JsString>() {
        let name = name_or_age
            .downcast::<JsString>()
            .or_throw(&mut cx)?
            .value();
        println!("Hi, this is {}", name);
    } else if name_or_age.is_a::<JsNumber>() {
        let age = name_or_age
            .downcast::<JsNumber>()
            .or_throw(&mut cx)?
            .value();
        println!("Her birthday is on the {}th", age);
    } else {
        panic!("Name is not a string and age is not a number");
    }

    Ok(cx.undefined().upcast())
}
// --snip--
```

```js
const addon = require('../native');

const john = new addon.Employee(0, 'Lisa');
john.introduce('Mary'); // Hi, this is Mary
john.introduce(12); // Her birthday is on the 12th
```

## Advanced Example

```rust
#[macro_use]
extern crate neon;

use neon::prelude::*;

pub struct User {
    id: i32,
    first_name: String,
    last_name: String,
    email: String,
}

type Unit = ();

declare_types! {
  pub class JsUser for User {
    init(mut cx) {
      let id = cx.argument::<JsNumber>(0)?;
      let first_name: Handle<JsString> = cx.argument::<JsString>(1)?;
      let last_name: Handle<JsString> = cx.argument::<JsString>(2)?;
      let email: Handle<JsString> = cx.argument::<JsString>(3)?;

      Ok(User {
        id: id.value() as i32,
        first_name: first_name.value(),
        last_name: last_name.value(),
        email: email.value(),
      })
    }

    method get(mut cx) {
      let attr: String = cx.argument::<JsString>(0)?.value();

      let this = cx.this();

      match &attr[..] {
        "id" => {
          let id = {
            let guard = cx.lock();
            let user = this.borrow(&guard);
            user.id
          };
          Ok(cx.number(id).upcast())
        },
        "first_name" => {
          let first_name = {
            let guard = cx.lock();
            let user = this.borrow(&guard);
            user.first_name.clone()
          };
          Ok(cx.string(&first_name).upcast())
        },
        "last_name" => {
          let last_name = {
            let guard = cx.lock();
            let user = this.borrow(&guard);
            user.last_name.clone()
          };
          Ok(cx.string(&last_name).upcast())
        },
        "email" => {
          let email = {
            let guard = cx.lock();
            let user = this.borrow(&guard);
            user.email.clone()
          };
          Ok(cx.string(&email).upcast())
        },
        _ => cx.throw_type_error("property does not exist")
      }
    }

    method panic(_) {
      panic!("User.prototype.panic")
    }
  }
}
register_module!(mut m, {
    m.export_class::<JsUser>("User")
});
```
