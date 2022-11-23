---
title: ES6模块与CommonJS模块的区别
date: 2022-01-29 11:35:57
tags: JavaScript
categories: JavaScript
---



# 1.写法不同

最直观的，两者对于模块的导入导出写法不同

- ES6

```javascript
/* 导出 */
export let a = 10
export let b = 20
export default {...}

/* 导入 */
import {a, b} from './test.js'
import c from './test.js'
```

- CommonJS

```javascript
/* 导出 */
module.exports = { a: 10, b: 20 };

/* 导入 */
let test = require("./test.js");
console.log(test.a, test.b); // 输出 10 20
```

> 注意：上述 CommonJS 导出写法会改变原本指向的内存，使其与 exports 指向不同内存空间，语法非本篇重点，可看这篇博客
>
> https://www.cnblogs.com/fightjianxian/p/12151010.html

<br />

# 2.加载阶段不同

- CommonJS 模块加载的是对象（即`module.exports`属性），它只有在脚本运行完才会生成，所以是`运行时加载`
- 而 ES6 模块不是对象，它的对外接口是一种静态定义，在代码静态解析阶段就会生成，所以是`编译时输出接口`

<br />

# 3.输出内容不同

- 刚刚说了 CommonJS 模块输出的是一个对象，所以是一个值的拷贝（浅拷贝）

请看下面这个例子，模块内有一个变量 a 和一个自增函数

```javascript
/* 模块test.js */
let a = 1;
function add() {
  a++;
}
module.exports = {
  a: a,
  add: add,
};
/* 需要引入模块的文件main.js */
let mod = require("./test.js");
console.log(mod.a); // 1
mod.add();
console.log(mod.a); // 1
```

上述代码说明，`test.js`模块加载以后，它内部的变化影响不到已经输出的`mod.a`了，因为 a 会被缓存（a 是一个简单数据类型，导出时直接拷贝值）。除非写成一个函数，才能获得内部变动后的值

```javascript
/* 模块test.js */
let a = 1;
function add() {
  a++;
}
module.exports = {
  get a() {
    return a;
  },
  add: add,
};
/* 需要引入模块的文件main.js */
let mod = require("./test.js");
console.log(mod.a); // 1
mod.add();
console.log(mod.a); // 2
```

上面代码中，输出的`a`实际上是一个取值器函数。如果想输出后的`a`发生改变，可以这样修改

```javascript
/* 模块test.js */
let a = 1;
function add() {
  this.a++;
}
module.exports = {
  a: a,
  add: add,
};
/* 需要引入模块的文件main.js */
let mod = require("./test.js");
console.log(mod.a); // 1
mod.add();
console.log(mod.a); // 2
```

这样改变的是输出的对象的`a`属性，模块内部并未受影响

```javascript
/* 模块test.js */
let a = 1;
function add() {
  this.a++;
  console.log(a); // 1
}
module.exports = {
  a: a,
  add: add,
};
/* 需要引入模块的文件main.js */
let mod = require("./test.js");
mod.add();
console.log(mod.a); // 2
```

这或许对你理解`CommonJS模块输出的是一个对象`有所帮助

- 而 ES6 模块输出的是值的引用

ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令`import`，就会生成一个`只读引用`。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。同样的代码

```javascript
/* 模块test.js */
export let a = 1;
export function add() {
  a++;
}
/* 需要引入模块的文件main.js */
import { a, add } from "./test.js";
console.log(a); // 3
add();
console.log(a); // 4
```

上述代码说明 ES6 模块输出的变量`a`完全反应其在所在模块内部的变化

<br />

# 4.加载方式不同

- CommonJS 模块的`require()`是同步加载模块
- ES6 模块的`import`命令是异步加载，有一个独立的模块依赖的解析阶段

<br />

> 参考自阮一峰老师的博客
>
> [https://es6.ruanyifeng.com/#docs/module-loader#ES6-%E6%A8%A1%E5%9D%97%E4%B8%8E-CommonJS-%E6%A8%A1%E5%9D%97%E7%9A%84%E5%B7%AE%E5%BC%82](https://es6.ruanyifeng.com/#docs/module-loader#ES6-模块与-CommonJS-模块的差异)