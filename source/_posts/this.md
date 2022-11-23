---
title: this
date: 2022-03-11 11:35:57
tags: JavaScript
categories: JavaScript
---



# 1.this指向

- 全局环境中的this -> 指向`window`对象
- 普通函数中的this -> 严格模式下指向`undefined`，非严格模式下指向`window`对象
- 构造函数中的this -> 指向`new`出来的对象
- `call、apply、bind`调用 -> 指向这三个方法的第一个参数，如果参数为`null`或者`undefined`，在非严格模式下指向`window`对象
- 箭头函数中的this -> 取决于箭头函数外的this指向

> 在**非严格模式**下，this指向不能是`undefined`或`null`，如果得出this将指向`undefined`或者`null`，那么this会指向`window`对象。
>
> 在浏览器环境下是`window`对象，在Node环境下是`global`。

简单一句话概括就是：谁调用，this就指向谁。

<br />

# 2.优先级

当有多种情况决定this指向时，优先级依次为：

1. 箭头函数
2. new
3. bind
4. apply和call
5. 对象调用方法
6. 直接调用函数
7. 全局环境

例如：

```javascript
let func = () => {
  console.log(this);
}
func.bind({})(); // 输出window，而不是空对象
```

<br />

# 3.apply、call和bind

这三个方法都会改变函数中的this指向，那他们有什么区别呢？以下方代码为例：

```javascript
function test(x, y){
    console.log(this, x, y);
}
test();    // window undefined undefined
```

<br />

## (1) call

`call`方法接收多个参数，第一个参数为要改变的this指向，后边参数为函数自身的参数。

```javascript
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
test.call(temp,1,2);    // temp对象 1 2
```

接下来我们尝试自己去实现一下`call`方法。

我们思考一下，怎么做才能让函数中的this指向一个对象？如果这个函数是对象体内的方法，那通过对象调用这个方法，函数中的this是不是就指向调用的对象。然后在调用完函数后，将这个方法删除掉，对象内容不变：

```javascript
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
temp.fn = test;
temp.fn(1, 2);    // temp对象 1 2
delete temp.fn;
```

按照这个思路，我们试着自己去写一个`call`方法：

```javascript
Function.prototype._call = function (context) {
    // 如果没有传入指定对象，默认为window
    context = context || window;
    // 将函数作为对象的方法，为了保证方法名唯一，使用Symbol
    // this指向函数
    const fn = Symbol();
    context[fn] = this;
    // 执行方法
    const res = context[fn](...[...arguments].slice(1));
    // 删除方法
    delete context[fn];
    // 返回结果
    return res;
};
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
test._call(temp, 1, 2);    // temp对象 1 2
```

<br />

## (2) apply

`apply`方法接收两个参数，第一个参数为要改变的this指向，第二个参数为函数自身的参数，以数组的形式传入。`apply`与`call`方法不同的地方就是传参的形式。

```javascript
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
test.apply(temp,[1,2]);    // temp对象 1 2
```

接下来我们尝试自己去实现一下`apply`方法。

`apply`方法和`call`方法一样，只不过是传入参数的方式不同而已：

```javascript
Function.prototype._apply = function (context) {
    context = context || window;
    const fn = Symbol();
    context[fn] = this;
    let res = null;
    // 如果存在第二个参数
    if (arguments[1]) {
        res = context[fn](...arguments[1]);
    } else {
        res = context[fn]();
    }
    delete context[fn];
    return res;
};
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
test._apply(temp, [1, 2]);    // temp对象 1 2
```

<br />

## (3) bind

`bind`方法接收多个参数，第一个参数是要改变的this指向，后边参数为函数自身的参数。它与上边两个方法不同的是，它不会立刻执行函数，而是返回一个新函数。

```javascript
let temp = { name: "小明" };
function test(x, y){
    console.log(this, x, y);
}
// 下面这样是不会执行的
// test.bind(temp, 1, 2);
let fn = test.bind(temp, 1, 2);
fn();  // temp对象 1 2
// 或者
test.bind(temp, 1, 2)();    // temp对象 1 2
// 另外可以将函数的参数进行拆分
test.bind(temp)(1, 2);    // temp对象 1 2
test.bind(temp, 1)(2);    // temp对象 1 2
```

接下来我们尝试自己去实现一下`bind`方法。

`bind`返回的是一个新函数，所以实现起来和上面两个有所不同：

```javascript
Function.prototype._bind = function (context) {
    const args = [...arguments].slice(1);
    const fn = this;
    return function() {
        return fn.apply(
            context,
            // 这个arguments是指返回的函数的参数
            // 这部分涉及函数柯里化
            args.concat(...arguments)
        );
    };
};
let temp = { name: "小明" };
function test(x, y) {
    console.log(this, x, y);
}
test._bind(temp, 1, 2)(); // temp对象 1 2
test._bind(temp, 1)(2); // temp对象 1 2
```

但是这样还有个瑕疵，在MDN中有这么一句话：

> bind()中的第一个参数：调用绑定函数时作为this参数传递给目标函数的值。 如果使用new运算符构造绑定函数，则忽略该值。

这句话中的绑定函数指`bind`返回的函数，那这句话是什么意思呢？由于`bind`方法返回的是一个函数，那么这个函数可以作为构造函数使用，当这个函数作为构造函数使用时，原来函数中的this指向的应该是这个构造函数创建的实例对象，而不是`bind`绑定的对象。

我们还是看回这个例子：

```javascript
let temp = { name: "小明" };
function test(x, y) {
    console.log(this, x, y);
}
// 这里输出的不再是temp对象，而是新创建的obj对象
let obj = new (test.bind(temp, 1, 2))();
```

如果用我们上面自己的写的`bind`方法，当返回函数作为构造函数时，this指向还是temp对象，所以要进一步修改：

```javascript
Function.prototype._bind = function (context) {
    const args = [...arguments].slice(1);
    const fn = this;
    return function Fn() {
        return fn.apply(
            this instanceof Fn ? this : context,
            args.concat(...arguments)
        );
    };
};
```

------

<br />

> 参考：
>
> https://juejin.cn/post/6844903746984476686
>
> https://juejin.cn/post/6946021671656488991