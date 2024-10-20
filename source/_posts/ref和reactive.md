---
title: ref和reactive
date: 2024-10-18 10:19:33
categories: 前端
tags: Vue
---

# 区别

* ref可以定义基本数据类型和对象数据类型；reactive可以定义对象数据类型
* ref创建的对象必须使用`.value`
* reactive重新分配一个新对象，会失去响应式（可以使用`Object.assign`去整体替换）

使用原则：

* 若需要一个基本类型的响应式数据，必须使用`ref`
* 若需要一个响应式对象，层级不深，`ref`和`reactive`都可以
* 若需要一个响应式对象，层级较深，推荐使用`reactive`

<br />

# 原理

## reactive

Proxy对象，简单描述：

```js
function reactive(obj) {
    return new Proxy(obj, {
        get(target, key, receiver) {
            // 如果是普通对象，会对该对象再包装
            if (typeof target[key] === "object") {
                return reactive(target[key]);
            };
            // 如果是ref响应式对象，自动解包(.value)
            return Reflect.get(target, key, receiver);
        },
        set(target, key, value, receiver) {
            return Reflect.set(target, key, value, receiver);
        }
    });
};
```

解释问题：

* 为什么对reactive响应式对象重新赋值会丢失响应式？

  ```js
  let test = reactive({
    a: 1,
    b: 2
  })
  test = {x:1,y:2} // 会丢失响应式
  ```

  这是因为test实际上是个Proxy对象，重新赋值变成了一个普通对象。

* 为什么reactive解构赋值会丢失响应式？

  准确来说，只有普通类型的属性会丢失响应，如果是对象类型的属性，响应式仍有效。

  ```js
  let test = reactive({
    a: 1,
    b: 2,
    c: {
      x: 1,
      y: 2,
    },
  })
  const { a, b, c } = test
  console.log(a, b, c) // 1 2 Proxy(Object)
  ```

  这是因为解构赋值等价于：

  ```js
  const a = test.a
  const b = test.b
  const c = test.c
  ```

  由于c是对象类型，在get函数中被再次包装，所以不会丢失响应式。

* 为什么reactive响应式对象中的属性赋予一个新对象，该新对象具有响应式？

  ```js
  let test = reactive({
    a: 1,
    b: 2
  })
  test.c = { x: 1, y: 2 }
  console.log(test.a, test.b, test.c) // 1 2 Proxy(Object)
  ```

  原因和上一个问题其实是一样的，因为触发了get方法，在get方法中进行了再次包装

* reactive响应式对象中的属性使用ref包裹，为什么不需要再使用.value？

  ```js
  let test = reactive({
    a: 1,
    b: ref(2),
  })
  console.log(test.a, test.b) // 1 2，b可以直接访问到，而不需要test.b.value
  ```

  原因是get方法里对ref对象自动解包

<br/>

## ref

为什么会出现ref响应式对象？因为Proxy只能用来包装对象，无法包装基础类型数据，所以需要自己实现一个包装类：

```js
import { reactive } from "./reactive";
import { trackEffects, triggerEffects } from './effect'

// 判断是否是对象
export const isObject = (value) => {
    return typeof value === 'object' && value !== null
}

// 将对象转为响应式
function toReactive(value) {
    return isObject(value) ? reactive(value) : value
}

class RefImpl {
    public _value;
    public dep = new Set; // 依赖
    public __v_isRef = true; // 是ref的标识
    constructor(public rawValue, public _shallow) {
        // 判断是否是浅ref（shallowRef）：浅ref不需要再次代理
        this._value = _shallow ? rawValue : toReactive(rawValue);
    }
    get value() {
        // 收集依赖
        trackEffects(this.dep)
        return this._value;
    }
    set value(newVal) {
        // set的值不等于初始值
        if (newVal !== this.rawValue) {
            // 判断是否是浅ref（shallowRef）
            this._value = this._shallow ? newVal : toReactive(newVal);
            // 将初始值变为本次的值
            this.rawValue = newVal
            // 触发依赖更新
            triggerEffects(this.dep)
        }
    }
}
```

解释问题：

* 为什么ref需要用`.value`访问其中值？

  因为ref是自己包装的一个类，为了使基础数据类型也具有响应式，所以需要`.value`访问其中值。

* 为什么ref包裹对象，其value是个Proxy？

  ref的底层实现中，如果是对象类型，实际上仍然依靠reactive方法。

* 为什么对reactive响应式对象重新赋值会丢失响应式，而对ref响应式对象.value重新赋值不会？

  ```js
  let test = ref({
    a: 1,
    b: 2,
  })
  test.value = { a: 4, b: 5 }
  console.log(test) // RefImpl，其中的value仍然是Proxy
  ```

  对reactive响应式对象重新赋值会丢失响应式已经在上文中解释过了，

  而对ref响应式对象.value重新赋值，实际上触发了类中的set方法，如果是对象，再次经过reactive包装，所以依然是个响应式对象。

* ref响应式对象中的属性使用ref包裹，为什么不需要再使用.value？

  ```js
  let test = ref({
    a: 1,
    b: ref(2),
  })
  console.log(test.value.a, test.value.b) // 1 2，b可以直接访问到，而不需要test.value.b.value
  ```

  原因其实和上文中说到的一样，由于ref包裹的是对象，所以test.value实际是靠reactive包装的。

  reactive中的get方法会对ref对象进行自动解包。

