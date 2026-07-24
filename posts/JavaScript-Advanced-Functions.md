---
title: JavaScript Advanced Functions
tags:
  - JavaScript
categories:
  - Frontend
date: 2026-06-26 00:00:00
katex: true
---

# JavaScript Advanced Functions

## 1. rest-parameters-spread

### 1.1. Rest

在函数定义中声明一个数组来收集参数。语法是这样的：`...变量名`，这将会声明一个数组并指定其名称，其中存有剩余的参数。这三个点的语义就是“收集剩余的参数并存进指定数组中”。

```js
function sumAll(...args) { // 数组名为 args  
let sum = 0;  
  
for (let arg of args) sum += arg;  
  
return sum;  
}
```

Rest 参数会收集剩余的所有参数，所以Rest 参数必须放到参数列表的末尾。

### 1.2. Spread

**Spread 语法** 看起来和 rest 参数很像，也使用 `...`，但是二者的用途完全相反。

当在函数调用中使用 `...arr` 时，它会把可迭代对象 `arr` “展开”到参数列表中。

```js
let arr = [3, 5, 1];  
  
alert( Math.max(...arr) ); // 5（spread 语法把数组转换为参数列表）
```

我们还可以使用 spread 语法来合并数组：

```js
let arr = [3, 5, 1];  
let arr2 = [8, 9, 15];  
  
let merged = [0, ...arr, 2, ...arr2];  
  
alert(merged); // 0,3,5,1,2,8,9,15（0，然后是 arr，然后是 2，然后是 arr2）
```

可以用 spread 语法这样操作任何**可迭代对象**。

```js
let str = "Hello";  
  
alert( [...str] ); // H,e,l,l,o
```

可以用来**浅拷贝**一个数组/对象

```js
let arr=[1,2,3,];
let arrCopy=[...arr];

let obj={a:1,b:2,c:3,};
let objCopy={...obj};
```


当我们在代码中看到 `"..."` 时，它要么是 rest 参数，要么是 spread 语法。

有一个简单的方法可以区分它们：

- 若 `...` 出现在函数参数列表的最后，那么它就是 rest 参数，它会把参数列表中剩余的参数收集到一个数组中。
- 若 `...` 出现在函数调用或类似的表达式中，那它就是 spread 语法，它会把一个数组展开为列表。

使用场景：

- Rest 参数用于创建可接受任意数量参数的函数。
- Spread 语法用于将数组传递给通常需要含有许多参数的函数。

我们可以使用这两种语法轻松地互相转换列表与参数数组。

## 2. settimeout-setinterval

- `setTimeout` 允许我们将函数推迟到一段时间间隔之后再执行。
- `setInterval` 允许我们重复运行一个函数，从一段时间间隔之后开始运行，之后以该时间间隔连续重复运行该函数

## 3. bind

在一个函数中将一个对象的方法作为参数传入会引发`this`指针丢失问题。

函数提供了一个内建方法 [bind](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)，它可以绑定 `this`。

```js
function count(obj){  
    return Object.entries(obj).length;  
}  
let user = {  
    firstName: "John",  
    sayHi() {  
        alert(`Hello, ${this.firstName}!`);  
    }  
};  
  
user.sayHi = user.sayHi.bind(user); // (*)  
  
// 可以在没有对象（译注：与对象分离）的情况下运行它  
user.sayHi(); // Hello, John!  
  
setTimeout(user.sayHi, 1000); // Hello, John!  
  
// 即使 user 的值在不到 1 秒内发生了改变  
// sayHi 还是会使用预先绑定（pre-bound）的值，该值是对旧的 user 对象的引用  
user = {  
    sayHi() { alert("Another user in setTimeout!"); }  
};
```

## 4. arrow-functions

箭头函数没有 `this`。如果访问 `this`，则会从外部获取

不具有 `this` 自然也就意味着另一个限制：箭头函数不能用作构造器（constructor）。不能用 `new` 调用它们。

## 1. Rest 与 Spread 的区别

二者都写作 `...`，但方向相反：

| 写法位置 | 名称 | 作用 |
|---|---|---|
| 函数定义的参数列表末尾 | Rest 参数 | 把剩余实参收集成数组 |
| 函数调用、数组字面量、对象字面量中 | Spread 语法 | 把数组/可迭代对象/对象展开 |

```js
function sum(...nums) {
  return nums.reduce((a, b) => a + b, 0);
}

const arr = [1, 2, 3];
sum(...arr); // 6
```

## 2. `setTimeout` 与 `setInterval`

```js
const id = setTimeout(() => {
  console.log("run once");
}, 1000);

clearTimeout(id);
```

```js
const timer = setInterval(() => {
  console.log("tick");
}, 1000);

clearInterval(timer);
```

实际使用时要记得保存返回值，以便后续取消定时任务。

## 3. `this` 丢失与 `bind`

对象方法被“拿出来”作为普通函数传递时，原来的调用者对象会丢失，所以 `this` 可能不再指向原对象。

```js
const user = {
  name: "John",
  sayHi() {
    console.log(this.name);
  }
};

setTimeout(user.sayHi, 1000); // this 丢失
setTimeout(user.sayHi.bind(user), 1000); // 绑定 this
```

`bind` 会返回一个新函数，这个新函数的 `this` 被固定为传入对象。

## 4. 箭头函数与 `this`

箭头函数没有自己的 `this`，它会从外层作用域捕获 `this`。因此：

- 适合短回调；
- 适合在对象方法内部保留外层 `this`；
- 不适合作为需要动态 `this` 的对象方法；
- 不能用作构造函数，不能 `new`。

对象方法中通常不要直接写箭头函数：

```js
const user = {
  name: "John",
  // 不推荐：这里 this 不指向 user
  sayHi: () => console.log(this.name)
};
```

更稳妥：

```js
const user = {
  name: "John",
  sayHi() {
    console.log(this.name);
  }
};
```
