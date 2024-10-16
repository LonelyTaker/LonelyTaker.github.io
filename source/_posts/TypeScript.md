---
title: TypeScript
date: 2024-10-16 15:36:40
categories: 前端
tags: TypeScript
---

# 快速入门

* 手动编译

  ```shell
  npm i typescript -g # 全局安装ts
  tsc index.ts # 编译ts文件
  ```

* 自动化编译

  ```shell
  tsc --init # 初始化配置
  tsc --watch # 监控目录下ts文件变更
  ```

<br/>

# 变量声明/推断

```ts
let a: string = '123'
let b: number = 9
let c: boolean = true

function count(x:number, y:number):number{
    return x + y;
}

// 字面量类型
let d: 'hello'
d = 'hello' // d只能赋值hello

// TS会根据我们的代码，进行类型推断（但类型推断不是万能的，尽量还是写明类型）
let e = -99 // TS会推断类型为数字
```

<br/>

# 类型总览

## js中的数据类型

简单数据类型

* number
* string
* boolean
* null
* undefined

复杂数据类型

* Object（包括Array、Function、Date等）

<br/>

## ts中的数据类型

ts的数据类型包括上面js的类型，同时新增了几个类型：

* any
* unknown
* never
* void
* tuple
* enum

另外还有两个用于自定义类型的方式：

* type
* interface

<br/>

## 类型声明中的大小写问题

```ts
let str1: string // Ts官方推荐的写法
str1 = 'hello'
str1 = new String('hello') // 会报错，不能将String分配给string
// string是基元，String是包装器对象，声明string类型的变量不能赋值一个包装器对象，推荐使用string

let str2: String // 这种写法表示这个变量既支持基本类型，也支持包装器对象
str2 = 'hello'
str2 = new String('hello')
```

> 在JS中的这些内置构造函数：Number、String、Boolean，它们用于创建对应的包装对象，在日常开发时很少使用，在TS中也是同理，所以在TS中进行类型声明时，通常都是用小写的number、string、boolean

>额外补充：包装对象、自动装箱

<br/>

# 常用类型与语法

## any

`any`表任意类型，一旦将变量类型设置为`any`，那就意味着放弃了对该变量的类型检查。

```ts
let a: any
a = 100
a = '你好'
a = false
```

注意：`any`类型的变量，可以赋值给任意类型的变量

```js
let c: any
c = 9

let x: string
x = c // 无警告
```

<br/>

## unknown

`unknown`的含义是个未知类型

* 它可以理解为一个类型安全的`any`，适用于：不确定数据的具体类型。

  ```js
  let a: unknown
  // 以下均正常
  a = 100
  a = false
  a = '你好'
  
  let x: string
  x = a // 警告：不能将类型“unknown”分配给类型“string”
  ```

* 它会强制开发者在使用之前进行类型检查，从而提供更强的类型安全性。

  ```ts
  // 方式一
  if(typeof a === 'string'){
      x = a
  }
  // 方式二（断言）
  x = a as string
  x = <string>a
  ```

* 读取`any`类型数据的任何属性都不会报错，而`unknown`正好与之相反。

  ```ts
  let str1: string
  str1 = 'hello'
  str1.toUpperCase() // 无警告
  
  let str2: any
  str2 = 'hello'
  str2.toUpperCase() // 无警告
  
  let str3: unknown
  str3 = 'hello'
  str3.toUpperCase() // 警告：”str3“类型为”unknown“
  ```

<br/>

## never

`never`的含义是任何值都不是，简言之就是不能有值，`undefined`、`null`、`''`、`0`都不行。

* 几乎不用`never`去直接限制变量，因为没有意义

  ```ts
  let a: never
  
  // 下方都会有警告
  a = 1
  a = true
  a = undefined
  a = null
  ```

* `never`一般是TS主动推断出来的

  ```ts
  let a: string
  a = 'hello'
  
  if(typeof a === 'string'){
      console.log(a.toUpperCase())
  }else{
      console.log(a) // TS会推断此处的a是never，因为没有任何一个值符合此处的逻辑
  }
  ```

* `never`也可用于限制函数的返回值

  ```ts
  // 限制throwError函数不需要有任何返回值，任何值都不行，包括undefined、null
  function throwError(str: string): never {
      throw new Error('异常退出：' + str)
  }
  ```

<br/>

## void

* `void`通常用于函数返回值声明，含义：函数不返回任何值，或者说函数返回空，调用者也不应该依赖其返回值进行任何操作。

  ```ts
  function logMessage(msg: string):void{
      console.log(msg)
  }
  logMessage("你好")
  ```

  >注意：虽然没有return显示指定返回值，但会有一个隐式返回值，就是`undefined`。所以，虽然函数返回类型为`void`，但也是可以接受`undefined`的。
  >
  >总结：`undefined`是`void`可以接受的一种“空”。

* 以下写法均符合规范

  ```ts
  function logMessage(msg:string):void{
      console.log(msg)
  }
  
  function logMessage(msg:string):void{
      console.log(msg)
      return;
  }
  
  function logMessage(msg:string):void{
      console.log(msg)
      return undefined;
  }
  ```

* 理解`void`和`undefined`

  ```ts
  function logMessage(msg:string):void{
      console.log(msg)
  }
  let result = logMessage("你好")
  
  if(result){} // 警告
  
  function logMessage(msg:string):undefined{
      console.log(msg)
  }
  let result = logMessage("你好")
  
  if(result){} // 无警告
  ```

  * `void`是一个广泛的概念，用来表示“空”，而`undefined`是这种“空”的具体实现之一
  * 因此可以说`undefined`是`void`可以接受“空”状态的一种具体形式。
  * 话句话说：`void`包含`undefined`，但`void`表达的语义超越了单纯的`undefined`，它是一种意图上的约定，而不仅仅是特定值的限制。

  总结：若函数返回类型为`void`，那么：

  * 从语法上讲，函数是可以返回`undefined`的，无论是显示返回还是隐式返回。
  * 从语义上讲，函数调用者不应该关心函数返回的值，也不应依赖返回值进行任何操作，即使返回了`undefined`值。

<br/>

## object

>关于object和Object，实际开发中用的相对较少，因为范围太大了。

* object的含义是：所有非原始对象，可存储：对象、函数、数组等，由于限制的范围比较宽泛，在实际开发中使用的相对较少。

  ```ts
  let a:object
  
  // 以下无警告
  a={}
  a={name:"张三"}
  a=[1,3,5,7,9]
  a=function(){}
  a=new String("123")
  class Person {}
  a = new Person()
  
  // 以下有警告
  a = 1
  a = true
  a = '你好'
  a = null
  a = undefined
  ```

* Object的含义是：所有可以调用Object方法的类型。

  简单记忆：除了`undefined`和`null`的任何值。

  由于限制范围太大，所以实际开发中使用频率极低。

  ```ts
  let a:Object
  
  // 以下无警告
  a={}
  a={name:"张三"}
  a=[1,3,5,7,9]
  a=function(){}
  a=new String("123")
  class Person {}
  a = new Person()
  a = 1
  a = true
  a = '你好'
  
  // 以下有警告
  a = null
  a = undefined
  ```

既然object和Object限制都太宽泛，那么该如何声明一个对象呢？

### 声明对象类型

* 实际开发中，限制一般对象，通常使用以下形式

  ```ts
  // 限制person对象必须有name属性，age为可选属性
  let person1: {name: string, age?: number}
  // 含义同上，也能用分号做分隔
  let person2: {name: string; age?: number}
  // 含义同上，也能用换行做分隔
  let person3: {
      name: string
      age?: number
  }
  
  person1 = {name:'Tom',age:18}
  person2 = {name:'Tom'}
  ```

* 索引签名

  允许定义对象可以具有任意数量的属性，这些属性的键和类型是可变的，常用于：描述类型不确定的属性（具有动态属性的对象）

  ```ts
  let person: {
      name: string
      age?: number
      [key:string]:any // 索引签名，完全可以不用key这个单词，换成其他的也可以
  }
  
  person = {
      name:'张三',
      age:18,
      gender:"男"
  }
  ```

### 声明函数类型

```ts
let count: (a: number, b: number) => number

count = function (x, y) {
 return x + y 
}
```

* TypeScript 中的 `=>` 在函数类型声明时表示函数类型，描述其参数类型和返回类型。
* JavaScript 中的 `=>` 是⼀种定义函数的语法，是具体的函数实现。
* 函数类型声明还可以使⽤：接⼝、⾃定义类型等⽅式，下⽂中会详细讲解。

### 声明数组类型

```ts
let arr1: string[] 
let arr2: Array<string>

arr1 = ['a','b','c']
arr2 = ['hello','world']
```

上述代码中的 Array 属于泛型，下⽂会详细讲解。

<br/>

## tuple

元组 (Tuple) 是⼀种特殊的**数组类型**，可以存储固定数量的元素，并且每个元素的类型是已知的且可以不同。元组⽤于精确描述⼀组值的类型， `?`表示可选元素。

```ts
// 第⼀个元素必须是 string 类型，第⼆个元素必须是 number 类型。
let arr1: [string,number]
// 第⼀个元素必须是 number 类型，第⼆个元素是可选的，如果存在，必须是 boolean 类型。
let arr2: [number,boolean?]
// 第⼀个元素必须是 number 类型，后⾯的元素可以是任意数量的 string 类型
let arr3: [number,...string[]]

// 可以赋值
arr1 = ['hello',123]
arr2 = [100,false]
arr2 = [200]
arr3 = [100,'hello','world']
arr3 = [100]

// 不可以赋值，arr1声明时是两个元素，赋值的是三个
arr1 = ['hello',123,false]
```

<br/>

## enum

枚举（enum）可以定义⼀组命名常量，它能增强代码的可读性，也让代码更好维护。

如下代码的功能是：根据调⽤ walk 时传⼊的不同参数，执⾏不同的逻辑，存在的问题是调⽤ walk 时传参时没有任何提示，编码者很容易写错字符串内容；并且⽤于判断逻辑的up、down、left、right是连续且相关的⼀组值，那此时就特别适合使⽤ 枚举（enum）。

```ts
function walk(str:string) {
 if (str === 'up') {
 console.log("向【上】⾛");
 } else if (str === 'down') {
 console.log("向【下】⾛");
 } else if (str === 'left') {
 console.log("向【左】⾛");
 } else if (str === 'right') {
 console.log("向【右】⾛");
 } else {
 console.log("未知⽅向");
 }
}

walk('up')
walk('down')
walk('left')
walk('right')
```

* 数组枚举

  数字枚举⼀种最常⻅的枚举类型，其成员的值会**⾃动递增**，且数字枚举还具备反向映射的特点，在下⾯代码的打印中，不难发现：可以通过值来获取对应的枚举成员名称 。

  ```ts
  // 定义⼀个描述【上下左右】⽅向的枚举Direction
  enum Direction {
      Up,
      Down,
      Left,
      Right
  }
  
  console.log(Direction) // 打印Direction会看到如下内容
  /* 
      {
          0:'Up', 
          1:'Down', 
          2:'Left', 
          3:'Right', 
          Up:0, 
          Down:1, 
          Left:2,
          Right:3
      } 
  */
  
  // 反向映射
  console.log(Direction.Up)
  console.log(Direction[0])
  
  // 此⾏代码报错，枚举中的属性是只读的
  Direction.Up = 'shang'
  ```

  也可以指定枚举成员的初始值，其后的成员值会⾃动递增。

  ```ts
  enum Direction {
      Up = 6,
      Down,
      Left,
      Right
  }
  
  console.log(Direction.Up); // 输出: 6
  console.log(Direction.Down); // 输出: 7
  ```

  使⽤数字枚举完成刚才 walk 函数中的逻辑，此时我们发现： 代码更加直观易读，⽽且类 型安全，同时也更易于维护。

  ```ts
  enum Direction {
      Up,
      Down,
      Left,
      Right,
  }
  
  function walk(n: Direction) {
      if (n === Direction.Up) {
          console.log("向【上】⾛");
      } else if (n === Direction.Down) {
          console.log("向【下】⾛");
      } else if (n === Direction.Left) {
          console.log("向【左】⾛");
      } else if (n === Direction.Right) {
          console.log("向【右】⾛");
      } else {
          console.log("未知⽅向");
      }
  }
  
  walk(Direction.Up)
  walk(Direction.Down)
  ```

* 字符串枚举

  枚举成员的值是字符串

  ```ts
  enum Direction {
      Up = "up",
      Down = "down",
      Left = "left",
      Right = "right"
  }
  
  let dir: Direction = Direction.Up;
  console.log(dir); // 输出: "up"
  ```

  注意：字符串枚举会丢失反向映射

* 常量枚举

  >官⽅描述：常量枚举是⼀种特殊枚举类型，它使⽤ const 关键字定义，在编译时会被**内联，避免⽣成⼀些额外的代码**。
  >
  >何为编译时内联？
  >
  >所谓“内联”其实就是 TypeScript 在编译时，会将枚举成员引⽤替换为它们的实际值，⽽不是⽣成额外的枚举对象。这可以减少⽣成的 JavaScript 代码量，并提⾼运⾏时性能。 

  使⽤普通枚举的 TypeScript 代码如下：

  ```ts
  enum Directions {
   Up,
   Down,
   Left,
   Right
  }
  
  let x = Directions.Up;
  ```

  编译后⽣成的 JavaScript 代码量较⼤ ：

  ```js
  "use strict";
  var Directions;
  (function (Directions) {
   Directions[Directions["Up"] = 0] = "Up";
   Directions[Directions["Down"] = 1] = "Down";
   Directions[Directions["Left"] = 2] = "Left";
   Directions[Directions["Right"] = 3] = "Right";
  })(Directions || (Directions = {}));
  
  let x = Directions.Up;
  ```

  使⽤常量枚举的 TypeScript 代码如下：

  ```ts
  const enum Directions {
   Up,
   Down,
   Left,
   Right
  }
  
  let x = Directions.Up;
  ```

  编译后⽣成的 JavaScript 代码量较⼩：

  ```js
  "use strict";
  let x = 0 /* Directions.Up */;
  ```

<br/>

## type

`type`可以为任意类型创建别名，让代码更简洁、可读性更强，同时能更⽅便地进⾏类型复⽤和扩展。

### 基本用法

类型别名使⽤ type 关键字定义， type 后跟类型名称，例如下⾯代码中 num 是类型别名。

```ts
type num = number

let price: num
price = 100
```

### 联合类型

联合类型是⼀种⾼级类型，它表示⼀个值可以是⼏种不同类型之⼀。

```ts
type Status = number | string
type Gender = '男' | '⼥'

function printStatus(status: Status) {
 console.log(status);
}

function logGender(str:Gender){
 console.log(str)
}

printStatus(404);
printStatus('200');
printStatus('501');

logGender('男')
logGender('⼥')
```

### 交叉类型

交叉类型（Intersection Types）允许将多个类型合并为⼀个类型。合并后的类型将拥有所有被合并类型的成员。交叉类型通常⽤于对象类型。

```ts
// ⾯积
type Area = {
 height: number; // ⾼
 width: number; // 宽
};

// 地址
type Address = {
 num: number; // 楼号
 cell: number; // 单元号
 room: string; // 房间号
};

// 定义类型House，且House是Area和Address组成的交叉类型
type House = Area & Address;
const house: House = {
 height: 180,
 width: 75,
 num: 6,
 cell: 3,
 room: '702'
};
```

<br/>

### 特殊情况

先观察下面两段代码：

* 代码1（正常）

  在函数定义时，限制函数返回值为 void ，那么函数的返回值就必须是空。

  ```ts
  function demo():void{
   // 返回undefined合法
   return undefined
  
   // 以下返回均不合法
   return 100
   return false
   return null
   return []
  }
  
  demo()
  ```

* 代码2（特殊）

  使⽤**类型声明**限制函数返回值为 void 时， TypeScript 并**不会严格要求**函数返回空。

  ```ts
  type LogFunc = () => void
  
  const f1: LogFunc = () => {
   return 100; // 允许返回⾮空值
  };
  const f2: LogFunc = () => 200; // 允许返回⾮空值
  const f3: LogFunc = function () {
   return 300; // 允许返回⾮空值
  };
  ```

为什么会这样？

是为了确保如下代码成⽴，我们知道`Array.prototype.push`的返回值是⼀个数字，⽽`Array.prototype.forEach`⽅法期望其回调的返回类型是 void 。

```ts
const src = [1, 2, 3];
const dst = [0];

src.forEach((el) => dst.push(el));
```

> 官方说明：[Assignability of Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html#assignability-of-functions)

<br/>

## 类

```ts
class Person {
    // 属性声明
    name: string
    age: number
    // 构造器
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
    // ⽅法
    speak() {
        console.log(`我叫：${this.name}，今年${this.age}岁`)
    }
}

// Person实例
const p1 = new Person('周杰伦', 38)
```

继承

```ts
class Student extends Person {
    grade: string
    // 构造器
    constructor(name: string, age: number, grade: string) {
        super(name, age)
        this.grade = grade
    }
    // 备注：本例中若Student类不需要额外的属性，Student的构造器可以省略
    // 重写从⽗类继承的⽅法
    override speak() {
        console.log(`我是学⽣，我叫：${this.name}，今年${this.age}岁，在读${this.grade}年级`)
    }
    // ⼦类⾃⼰的⽅法
    study() {
        console.log(`${this.name}正在努⼒学习中......`)
    }
}
```

<br/>

## 属性修饰符

| 修饰符      | 含义     | 具体规则                         |
| ----------- | -------- | -------------------------------- |
| `public`    | 公开的   | 可以被：类内部、子类、类外部访问 |
| `protected` | 受保护的 | 可以被：类内部、子类访问         |
| `private`   | 私有的   | 可以被：类内部访问               |
| `readonly`  | 只读属性 | 属性无法修改                     |

### public修饰符

```ts
class Person {
    // name写了public修饰符，age没写修饰符，最终都是public修饰符
    public name: string
    age: number
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
    // 方法没写修饰符，也是public
    speak() {
        // 类的【内部】可以访问public修饰的name和age
        console.log(`我叫：${this.name}，今年${this.age}岁`)
    }
}

const p1 = new Person('张三', 18)
// 类的【外部】可以访问public修饰的属性
console.log(p1.name)
```

```ts
class Student extends Person {
    constructor(name: string, age: number) {
        super(name, age)
    }
    study() {
        // 【⼦类中】可以访问⽗类中public修饰的：name属性、age属性
        console.log(`${this.age}岁的${this.name}正在努⼒学习`)
    }
}
```

属性的简写形式：

```ts
// 完整写法
class Person {
    public name: string;
    public age: number;
    constructor(name: string, age: number) {
        this.name = name;
        this.age = age;
    }
}
// 简写
class Person {
    constructor(
        public name: string,
        public age: number
    ){}
}
```

### protected修饰符

```ts
class Person {
    // name和age是受保护属性，不能在类外部访问，但可以在【类】与【⼦类】中访问
    constructor(
        protected name: string,
        protected age: number
    ) {}
    // getDetails是受保护⽅法，不能在类外部访问，但可以在【类】与【⼦类】中访问
    protected getDetails(): string {
        // 类中能访问受保护的name和age属性
        return `我叫：${this.name}，年龄是：${this.age}`
    }
    // introduce是公开⽅法，类、⼦类、类外部都能使⽤
    introduce() {
        // 类中能访问受保护的getDetails⽅法
        console.log(this.getDetails());
    }
}

const p1 = new Person('杨超越',18)
// 可以在类外部访问introduce
p1.introduce()
// 以下代码均报错
p1.getDetails()
p1.name
p1.age
```

```ts
class Student extends Person {
    constructor(name:string,age:number){
        super(name,age)
    }
    study(){
        // ⼦类中可以访问introduce
        this.introduce()
        // ⼦类中可以访问name
        console.log(`${this.name}正在努⼒学习`)
    }
}

const s1 = new Student('tom',17)
s1.introduce()
```

### private修饰符

```ts
class Person {
    constructor(
        public name: string,
        public age: number,
        // IDCard属性为私有的(private)属性，只能在【类内部】使⽤
        private IDCard: string
    ) { }
    private getPrivateInfo(){
        // 类内部可以访问私有的(private)属性 —— IDCard
        return `身份证号码为：${this.IDCard}`
    }
    getInfo() {
        // 类内部可以访问受保护的(protected)属性 —— name和age
        return `我叫: ${this.name}, 今年刚满${this.age}岁`;
    }
    getFullInfo(){
        // 类内部可以访问公开的getInfo⽅法，也可以访问私有的getPrivateInfo⽅法
        return this.getInfo() + '，' + this.getPrivateInfo()
    }
}

const p1 = new Person('张三',18,'110114198702034432')
console.log(p1.getFullInfo())
console.log(p1.getInfo())

// 以下代码均报错
// p1.name
// p1.age
// p1.IDCard
// p1.getPrivateInfo()
```

### readonly修饰符

```ts
class Car {
    constructor(
        public readonly vin: string, //⻋辆识别码，为只读属性
        public readonly year: number,//出⼚年份，为只读属性
        public color: string,
        public sound: string
    ) { }
    // 打印⻋辆信息
    displayInfo() {
        console.log(`
            识别码：${this.vin},
            出⼚年份：${this.year},
            颜⾊：${this.color},
            ⾳响：${this.sound}
        `);
    }
}

const car = new Car('1HGCM82633A123456', 2018, '⿊⾊', 'Bose⾳响');

car.displayInfo()

// 以下代码均错误：不能修改 readonly 属性
// car.vin = '897WYE87HA8SGDD8SDGHF';
// car.year = 2020;
```

<br/>

## 抽象类

概述：抽象类是⼀种⽆法被实例化的类，专⻔⽤来定义类的结构和⾏为，类中可以写抽象⽅法，也可以写具体实现。抽象类主要⽤来为其派⽣类提供⼀个基础结构，要求其派⽣类必须实现其中的抽象⽅法。

简记：抽象类不能实例化，其意义是可以被继承，抽象类⾥可以有普通⽅法、也可以有抽象⽅法。

```ts
abstract class Package {
    constructor(public weight: number) { }
    // 抽象⽅法：⽤来计算运费，不同类型包裹有不同的计算⽅式
    abstract calculate(): number
    // 通⽤⽅法：打印包裹详情
    printPackage() {
        console.log(`包裹重量为: ${this.weight}kg，运费为:${this.calculate()}元`);
    }
}
```

```ts
// 标准包裹
class StandardPackage extends Package {
    constructor(
        weight: number,
        public unitPrice: number // 每公⽄的固定费率
    ) { super(weight) }
    // 实现抽象⽅法：计算运费
    calculate(): number {
        return this.weight * this.unitPrice;
    }
}

// 创建标准包裹实例
const s1 = new StandardPackage(10,5)
s1.printPackage()

// 特快包裹
class ExpressPackage extends Package {
    constructor(
        weight: number,
        private unitPrice: number, // 每公⽄的固定费率（快速包裹更⾼）
        private additional: number // 超出10kg以后的附加费
    ) { super(weight) }
    // 实现抽象⽅法：计算运费
    calculate(): number {
        if(this.weight > 10){
            // 超出10kg的部分，每公⽄多收additional对应的价格
            return 10 * this.unitPrice + (this.weight - 10) * this.additional
        } else {
            return this.weight * this.unitPrice;
        }
    }
}

// 创建特快包裹实例
const e1 = new ExpressPackage(13,8,2)
e1.printPackage()
```

总结：何时使⽤抽象类？

* 定义通用接口：为⼀组相关的类定义通⽤的⾏为（⽅法或属性）时。 

* 提供基础实现：在抽象类中提供某些⽅法或为其提供基础实现，这样派⽣类就可以继承这 些实现。

* 确保关键实现：强制派⽣类实现⼀些关键⾏为。

* 共享代码和逻辑：当多个类需要共享部分代码时，抽象类可以避免代码重复。

<br/>

## interface

interface 是⼀种定义结构的⽅式，主要作⽤是为：类、对象、函数等规定⼀种契约，这样可以确保代码的⼀致性和类型安全，但要注意 interface 只能定义格式，不能包含任何实现 ！

### 定义类结构

```ts
// PersonInterface接⼝，⽤与限制Person类的格式
interface PersonInterface {
 name: string
 age: number
 speak(n: number): void
}

// 定义⼀个类 Person，实现 PersonInterface 接⼝
class Person implements PersonInterface {
    constructor(
        public name: string,
        public age: number
    ) { }
    // 实现接⼝中的 speak ⽅法
    speak(n: number): void {
        for (let i = 0; i < n; i++) {
            // 打印出包含名字和年龄的问候语句
            console.log(`你好，我叫${this.name}，我的年龄是${this.age}`);
        }
    }
}

// 创建⼀个 Person 类的实例 p1，传⼊名字 'tom' 和年龄 18
const p1 = new Person('tom', 18);
p1.speak(3)
```

### 定义对象结构

```ts
interface UserInterface {
    name: string
    readonly gender: string // 只读属性
    age?: number // 可选属性
    run: (n: number) => void
}

const user: UserInterface = {
    name: "张三",
    gender: '男',
    age: 18,
    run(n) {
        console.log(`奔跑了${n}⽶`)
    }
};
```

### 定义函数结构

```ts
interface CountInterface {
    (a: number, b: number): number;
}

const count: CountInterface = (x, y) => {
    return x + y
}
```

### 接口继承

⼀个 interface 继承另⼀个 interface ，从⽽实现代码的复⽤

```ts
interface PersonInterface {
    name: string // 姓名
    age: number // 年龄
}

interface StudentInterface extends PersonInterface {
    grade: string // 年级
}

const stu: StudentInterface = {
    name: "张三",
    age: 25,
    grade: '⾼三',
}
```

### 接口合并

接口可重复定义，相同接口名会自动合并。

```ts
// PersonInterface接⼝
interface PersonInterface {
    // 属性声明
    name: string
    age: number
}

// 给PersonInterface接⼝添加新属性
interface PersonInterface {
    // ⽅法声明
    speak(): void
}

// Person类实现PersonInterface
class Person implements PersonInterface {
    name: string
    age: number
    // 构造器
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
    }
    // ⽅法
    speak() {
        console.log('你好！我是⽼师:', this.name)
    }
}
```

总结：何时使⽤接⼝？

* 定义对象的格式：描述数据模型、API响应格式、配置对象......等等，是开发中⽤的最多的场景。

* 类的契约：规定⼀个类需要实现哪些属性和⽅法。

* 自动合并：⼀般⽤于扩展第三⽅库的类型， 这种特性在⼤型项⽬中可能会⽤到。

<br/>

## 一些相似概念的区别

### interface和type的区别

相同点：

* `interface`和`type`都可以⽤于定义对象（函数）结构，在定义对象结构时两者可以互换。

不同点：

* `interface`更关注定义对象和类的结构，支持继承、合并。
* `type`可以定义类型别名、联合类型、交叉类型，但不⽀持继承和⾃动合并。

```ts
// 使⽤ interface 定义 Person 对象
interface PersonInterface {
    name: string;
    age: number;
    speak(): void;
}

// 使⽤ type 定义 Person 对象
type PersonType = {
    name: string;
    age: number;
    speak(): void;
};

// 使⽤PersonInterface
let person: PersonInterface = {
    name:'张三',
    age:18,
    speak(){
        console.log(`我叫：${this.name}，年龄：${this.age}`)
    }
}
// 使⽤PersonType
let person: PersonType = {
    name:'张三',
    age:18,
    speak(){
        console.log(`我叫：${this.name}，年龄：${this.age}`)
    }
}
```

```ts
// 接口可以继承、合并
interface PersonInterface {
    name: string // 姓名
    age: number // 年龄
}

interface PersonInterface {
    speak: () => void
}

interface StudentInterface extends PersonInterface {
    grade: string // 年级
}

const student: StudentInterface = {
    name: '李四',
    age: 18,
    grade: '⾼⼆',
    speak() {
        console.log(this.name,this.age,this.grade)
    }
}
```

```ts
// 使⽤ type 定义 Person 类型，并通过交叉类型实现属性的合并
type PersonType = {
    name: string; // 姓名
    age: number; // 年龄
} & {
    speak: () => void;
};

// 使⽤ type 定义 Student 类型，并通过交叉类型继承 PersonType
type StudentType = PersonType & {
    grade: string; // 年级
};

const student: StudentType = {
    name: '李四',
    age: 18,
    grade: '⾼⼆',
    speak() {
        console.log(this.name, this.age, this.grade);
    }
};
```

### interface和抽象类的区别

相同点：

* 都可以定义一个类的格式（定义类应遵循的契约）

不同点：

* 接口只能描述结构，不能有任何实现代码，⼀个类可以实现**多个**接⼝。
* 抽象类既可以包含抽象⽅法，也可以包含具体⽅法，⼀个类只能继承⼀个抽象类。

```ts
// FlyInterface 接⼝
interface FlyInterface {
    fly(): void;
}

// 定义 SwimInterface 接⼝
interface SwimInterface {
    swim(): void;
}

// Duck 类实现了 FlyInterface 和 SwimInterface 两个接⼝
class Duck implements FlyInterface, SwimInterface {
    fly(): void {
        console.log('鸭⼦可以⻜');
    }
    swim(): void {
        console.log('鸭⼦可以游泳');
    }
}

// 创建⼀个 Duck 实例
const duck = new Duck();
duck.fly(); // 输出: 鸭⼦可以⻜
duck.swim(); // 输出: 鸭⼦可以游泳
```

<br/>

# 泛型

泛型允许我们在定义函数、类或接⼝时，使⽤类型参数来表示未指定的类型，这些参数在具体使⽤时，才被指定具体的类型，泛型能让同⼀段代码适⽤于多种类型，同时仍然保持类型的安全性。

```ts
// 泛型函数
function logData<T>(data: T): T {
    console.log(data)
    return data
}

logData<number>(100)
logData<string>('hello')
```

```ts
// 泛型可以有多个
function logData<T, U>(data1: T, data2: U): T | U {
 console.log(data1,data2)
 return Date.now() % 2 ? data1 : data2
}

logData<number, string>(100, 'hello')
logData<string, boolean>('ok', false)
```

```ts
// 泛型接口
interface PersonInterface<T> {
    name: string,
    age: number,
    extraInfo: T
}

let p1: PersonInterface<string>
let p2: PersonInterface<number>

p1 = { name: '张三', age: 18, extraInfo: '⼀个好⼈' }
p2 = { name: '李四', age: 18, extraInfo: 250 }

```

```ts
// 泛型类
class Person<T> {
    constructor(
        public name: string,
        public age: number,
        public extraInfo: T
    ) { }
    speak() {
        console.log(`我叫${this.name}今年${this.age}岁了`)
        console.log(this.extraInfo)
    }
}

// 测试代码1
const p1 = new Person<number>("tom", 30, 250);

// 测试代码2
type JobInfo = {
    title: string;
    company: string;
}
const p2 = new Person<JobInfo>("tom", 30,
    {
        title: '研发总监', 
        company: '发发发科技公司' 
    }
);
```

```ts
// 泛型约束
interface LengthInterface {
    length: number
}

// 约束规则是：传⼊的类型T必须具有 length 属性
function logPerson<T extends LengthInterface>(data: T): void {
    console.log(data.length)
}

logPerson<string>('hello')
// 报错：因为number不具备length属性
// logPerson<number>(100)
```

<br/>

# 类型声明文件

类型声明⽂件是 TypeScript 中的⼀种特殊⽂件，通常以 `.d.ts` 作为扩展名。它的主要作⽤是为现有的 **JavaScript** 代码提供**类型信息**，使得 TypeScript 能够在使⽤这些 JavaScript 库或模块时进⾏**类型检查和提示**。

```js
// demo.js
export function add(a, b) {
 return a + b;
}

export function mul(a, b) {
 return a * b;
}
```

```ts
// demo.d.ts
declare function add(a: number, b: number): number;
declare function mul(a: number, b: number): number;

export { add, mul };
```

```ts
// example.ts
import { add, mul } from "./demo.js";

const x = add(2, 3); // x 类型为 number
const y = mul(4, 5); // y 类型为 number
console.log(x,y)
```

<br/>

# 装饰器

## 简介

* 装饰器本质是一种特殊的**函数**，它可以对：类、属性、方法、参数进行扩展，同时能让代码更简洁。
* 装饰器自`2015`年在`ECMAScript-6`中被提出到现在，已将近10年。
* 截止目前，装饰器依然是实验性特性 ，需要开发者手动调整配置，来开启装饰器支持。
* 装饰器有 5 种：
  * 类装饰器
  * 属性装饰器
  * 方式装饰器
  * 访问器装饰器
  * 参数装饰器

>备注：虽然`TypeScript5.0`中可以直接使用`类装饰器`，但为了确保其他装饰器可用，现阶段使用时，仍建议使用`experimentalDecorators`配置来开启装饰器支持，而且不排除在来的版本中，官方会进一步调整装饰器的相关语法！
>参考：[《TypeScript 5.0发版公告》](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0-rc/)

<br/>

## 类装饰器

类装饰器是一个应用在**类声明**上的**函数**，可以为类添加额外的功能，或添加额外的逻辑。

```ts
/* 
  Demo函数会在Person类定义时执行
  参数说明：
    ○ target参数是被装饰的类，即：Person
*/
function Demo(target: Function) {
  console.log(target)
}

// 使用装饰器
@Demo
class Person {}
```

