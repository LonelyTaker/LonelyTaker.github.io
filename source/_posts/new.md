---
title: new
date: 2022-03-09 11:35:57
categories: 前端
tags: JavaScript
---



以下方代码为例

```javascript
function Person(name){
    this.name = name;
}
let xiaoming = new Person('xiaoming');
```

我们知道创建一个对象是通过`new`操作符，那么`new`操作符到底做了哪些事情呢？

实际上它帮我们做了四件事情：

1. 创建一个空的对象

2. 将空对象原型`__proto__`指向函数的原型对象`prototype`

3. 函数的this指向这个空对象，并执行代码

4. 将这个对象返回

   构造函数返回值取决于函数的`return`：

   - 不写return -> 返回默认创建的对象
   - return this -> 返回默认创建的对象
   - return 基本数据类型 -> 返回默认创建的对象
   - return 对象 -> 返回该对象而非创建的对象

那么上面的代码，我们可以这样理解：

```javascript
let xiaoming = new Person('xiaoming');
// 上面这句代码可以看成这样
let temp = {};
temp.__proto__ = Person.prototype;
let res = Person.apply(temp, ["xiaoming"]);
let xiaoming = typeof res === 'object' ? res:temp;
```

再扩展一下，我们可以自己封装一个`new`函数：

```javascript
function _new(context) {
    let temp = {};
    temp.__proto__ = context.prototype;
    let res = context.apply(temp, [...arguments].slice(1));
    return typeof res === "object" ? res : temp;
}
function Person(name) {
    this.name = name;
}
let xiaoming = _new(Person, "xiaoming");    // 效果和 new Person("xiaoming") 一样
```

> 在讲原型和原型链的时候我们说`__proto__`属性不能直接使用，这是由于在ES6之前没有标准的方法能够直接操作隐式原型，所以才有了`__proto__`属性，通过它我们可以访问到对象的原型，所以`__proto__`属性其实是可以使用的，但是并不建议，因为不是所有的浏览器都支持通过`__proto__`来访问。我们这里只是用来模拟`new`操作符的实现，必须要用到`__proto__`而已。