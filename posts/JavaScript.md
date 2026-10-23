---
title: JavaScript
tags:
  - JavaScript
categories:
  - Frontend
date: 2026-06-26 00:00:00
katex: true
---

# 1. 语言介绍

## 1.1. 语言核心与宿主环境

JavaScript 的语言核心（类型、运算符、控制流、函数）在浏览器和 Node 里是同一套，差别在宿主额外提供的能力：浏览器补上 `window`、`document`，Node 补上 `process`、`fs`。浏览器里用 `<script type="module" src="app.js"></script>` 引入脚本，放在文档头部会阻塞后续解析，除非加 `defer` 或 `type="module"`；Node 在终端用 `node app.js` 执行。`console.log` 两边都能输出，浏览器在 Console 面板，Node 在终端，所以调试语句不必分环境改写。

```js
// 同一份文件在浏览器与 Node 下都能跑
const runtime = typeof document === "undefined" ? "node" : "browser";
console.log(`running in ${runtime}`);
```

## 1.2. 动态类型与弱类型

变量本身没有类型，类型属于它当前持有的值，所以同一个变量可以先存数字再存字符串。函数参数与返回值也不做类型检查，把字符串当数字用不会在赋值处报错，而是拖到参与运算或调用方法时才抛 `TypeError`。类型判断只能在运行时自己做：`typeof` 分原始类型，`Array.isArray` 分数组，`value === null` 判空。

```js
let total = 10;
total = total + " items";   // 隐式转换，变量改存 string
console.log(total);         // "10 items"
// (5).toUpperCase();       // TypeError：运行时才暴露类型错误
```

类型不匹配不会报错，只会由运算符按固定规则转换：`+` 有字符串操作数就拼接，其余算术运算符把操作数转数字；比较运算符（严格相等除外）在两边类型不同时转数字，两边都是字符串时按字典序比较。判等因此必须写 `===`，写 `==` 要把两边的转换路径一起推演，两条规则的细节都在第 3 节。

```js
console.log("1" + 2, "3" * "2");     // "12" 6
console.log("10" < 9, "10" < "9");   // false true：混合类型转数字，同为字符串按字典序
```

## 1.3. 原始类型与对象

原始类型赋值、传参、返回都复制整个值，对象复制的是指向同一块数据的引用。`const` 只锁绑定不锁内容，`const user = { name: "a" }` 之后 `user.name = "b"` 合法。改任意一个变量里的对象，另一个都能看到，判断内容相同要逐字段比较。浅拷贝与 `this` 的指向问题都由这条规则产生。

```js
const original = { count: 0 };
const alias = original;
alias.count = 1;
console.log(original.count);      // 1：同一块数据
console.log({ count: 1 } === { count: 1 }); // false：不同引用
```

## 1.4. 单线程与事件循环

调用栈上的代码依次执行，定时器回调、事件回调、网络回调进任务队列，必须等当前同步代码全部结束才轮到。长同步计算会推迟所有定时器，所以定时器不保证准时；定时器也不会打断正在执行的代码，所以回调不会插在同步代码中间。`setTimeout(fn, 0)` 的准确含义是把 `fn` 排到当前同步代码之后。

```js
console.log("1");
setTimeout(() => console.log("3"), 0); // 0 毫秒只是尽快排队
console.log("2");                      // 输出顺序：1 2 3
```

定时器本身用 `setTimeout(fn, delay)` 与 `setInterval(fn, delay)` 创建，返回值是标识，保存下来才能用 `clearTimeout(id)` 与 `clearInterval(id)` 取消。`delay` 只是下限：回调要等当前代码跑完、主线程空闲时才执行，回调跑得比间隔还慢时下一次也不会与它重叠。

```js
const once = setTimeout(() => console.log("run once"), 1000);
clearTimeout(once); // 在触发前取消

const timer = setInterval(() => console.log("tick"), 1000);
clearInterval(timer); // 不再需要时取消
```

## 1.5. 变量声明与作用域

`let` 是块级作用域，可以重新赋值；`const` 也是块级作用域，绑定不能重新赋值而对象内容仍可改；`var` 是函数作用域，允许重复声明并且声明提升，容易从循环和条件块里泄漏，现代代码不用。默认写 `const`，确实要重新赋值时才换 `let`。

```js
const config = { retries: 3 };
config.retries = 5;  // 合法：改的是对象内容
// config = {};      // TypeError: Assignment to constant variable.
```

## 1.6. 命名与语句风格

语句以分号结尾，字符串用双引号，缩进两格，文件名小写加连字符，一个文件只导出同一组功能；标识符按种类换写法，读代码时不必先找定义：

| 种类 | 写法 | 例子 |
| --- | --- | --- |
| 变量、函数 | camelCase | `userName`、`fetchData` |
| 类、构造函数 | PascalCase | `DateRange` |
| 模块级常量 | UPPER_SNAKE | `MAX_RETRY` |

```js
const MAX_RETRY = 3, userName = "Ada";  // 常量全大写，变量小驼峰
```

# 2. 值与类型

## 2.1. typeof

`typeof value` 返回小写字符串，对未声明的变量也安全（返回 `"undefined"` 而不抛错），但分不清数组与普通对象，也分不清 `null` 与对象，各值的返回结果见下表。

| 值 | `typeof` 结果 |
| --- | --- |
| `1`、`NaN` | `"number"` |
| `"a"`、`true` | `"string"`、`"boolean"` |
| `undefined` | `"undefined"` |
| `null` | `"object"`，历史遗留，`null` 不是对象 |
| `[]`、`{}`、`new Date()` | `"object"` |
| `function () {}` | `"function"` |

```js
function describe(value) {  // 先排除 null，再用 Array.isArray 排除数组
  if (value === null) return "null";         // typeof null 是 "object"
  if (Array.isArray(value)) return "array";  // 数组与对象都返回 "object"
  return typeof value;
}
console.log(describe(null), describe([1]), describe(() => {})); // null array function
```

## 2.2. 值的比较

原始类型按值比较、对象按引用比较：`5 === 5` 为 `true`，`[] === []` 为 `false`。`number`、`string`、`boolean`、`null`、`undefined`、`bigint` 按值比较，赋值与传参都是复制；`{}`、`[]`、函数、`Date`、`Map`、`Set` 是对象，按引用比较，字面量每次求值都创建新对象。整数超过 `2 ** 53 - 1` 时 `number` 不再精确，超出安全范围要用 `bigint`，它不能与 `number` 混合运算，也不能用 `Math` 方法。

```js
const a = { x: 1 };
const b = a;
console.log(a === b);   // true：同一个引用
console.log(2 ** 53 === 2 ** 53 + 1, 9007199254740993n); // true 9007199254740993n：超出安全整数范围就丢精度，bigint 才精确
```

## 2.3. 模板字符串

把变量嵌进字符串写成 `` `hello ${expression}` ``：`${}` 里的表达式先求值，结果按字符串规则转换后插入，换行也能直接写在字符串里。普通引号不做插值，写 `${name}` 会原样输出这几个字符。

```js
const name = "Ada";
console.log(`cart of ${name}: ${12 * 2} total`); // "cart of Ada: 24 total"
console.log('no interpolation ${name}');         // 单引号只输出字面量
```

# 3. 类型转换

## 3.1. 显式转换与隐式转换

显式转换用 `Number(value)`、`String(value)`、`Boolean(value)`；`parseInt(s, 10)` 从左往右读到不能读为止，适合 `"12px"` 这类输入，`Number("12px")` 则因为整串校验不通过得 `NaN`。隐式转换的触发点是算术运算符、`+` 的拼接分支、非严格比较运算符和条件位置。转换规则固定，下面两张表分别是数字与布尔的对应关系：

| 值 | 转成 number |
| --- | --- |
| `undefined` | `NaN` |
| `null` | `0` |
| `true` / `false` | `1` / `0` |
| 字符串 | 去掉两端空白后按数字字面量解析，空串为 `0`，有非法字符为 `NaN` |
| 对象 | 先用 `valueOf`，再用 `toString` 的结果解析 |

| 值 | 转成 boolean |
| --- | --- |
| `false`、`0`、`-0`、`0n`、`""`、`null`、`undefined`、`NaN` | `false` |
| 其他值，含 `"0"`、`" "`、`[]`、`{}` | `true` |

```js
console.log(Number(undefined), Number(null));       // NaN 0
console.log(parseInt("12px", 10), Number("12px"));  // 12 NaN
console.log(Boolean("0"), Boolean(" "));            // true true：非空字符串全为真
```

## 3.2. === 与 ==

判等写 `a === b`（不等写 `a !== b`）：`===` 要求类型和值都相同，不做转换；`==` 按一组复杂规则先转换，`"0" == false`、`[] == false`、`undefined == null` 都为 `true`，读代码时必须把转换路径推一遍。`==` 唯一实用的场景是 `value == null` 同时覆盖 `null` 与 `undefined`，可读写法是 `value === null || value === undefined`。`NaN` 与任何值（含自身）用 `===` 都是 `false`，判断用 `Number.isNaN(value)`。

```js
console.log("1" === 1, "1" == 1); // false true
console.log(NaN === NaN, Number.isNaN(NaN)); // false true
```

# 4. 运算符

## 4.1. ?? 与 ||

给默认值有两种写法：`a ?? b` 只在左侧是 `null` 或 `undefined` 时取右侧，`||` 把所有 falsy 值（`0`、`""`、`NaN`、`false`）都当作没有值。表单里的数字、计数器、分页参数、允许为空的字符串都可能合法地取 `0` 或 `""`，用 `||` 会把这些输入替换掉。`??` 与 `||` 或 `&&` 不加括号混写是语法错误。

```js
const page = 0;
console.log(page || 1, page ?? 1);   // 1 0
const a = 0, b = null, c = "fallback";
// const bad = a ?? b || c;          // SyntaxError：混用必须加括号
console.log((a ?? b) || c, a ?? (b || c)); // "fallback" 0
```

# 5. 控制流

## 5.1. if 与三元表达式

`if (cond) { ... } else { ... }` 是语句，不能放在赋值右侧或参数位置；三元表达式 `cond ? a : b` 有返回值，适合在赋值和参数处二选一，嵌套超过一层就改用 `if` 或提前 `return` 展平。条件位置按布尔规则转换，`if ([])` 会进入分支，所以判数组为空要看 `length`。

```js
const items = [];
if (items.length === 0) console.log("empty");
console.log(items.length ? `${items.length} items` : "empty"); // "empty"
```

## 5.2. 三种循环

需要索引、倒序或步长时用计数 `for (let i = 0; i < n; i++)`，循环变量用 `let`；`for (const v of list)` 按顺序取出可迭代对象的值，拿不到索引也不能遍历普通对象；`while` 用于次数未知、由条件决定退出的场景，循环变量在体外声明并在体内改变，忘记改变就是死循环。

```js
const colors = ["red", "green"];
for (let i = 0; i < colors.length; i++) console.log(i, colors[i]);
for (const [i, color] of colors.entries()) console.log(i, color);
let n = 10;
while (n > 1) n = n % 2 === 0 ? n / 2 : 3 * n + 1; // 最终落到 1
```

## 5.3. break、continue 与标签

`break` 结束整个循环，`continue` 结束本轮进入下一轮，两者默认只作用于最内层；`continue` 在 `while` 里直接跳到条件判断，位置写错容易死循环。多层循环要一次跳出时给外层加标签，`break 标签` 从带该标签的循环跳出，`continue 标签` 让该循环进入下一轮。

```js
outer:
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i * j > 2) break outer; // 一次跳出两层
    if (j === 1) continue;      // 只跳过本轮 j === 1
    console.log(i, j);
  }
}
```

# 6. 函数

## 6.1. 函数声明与函数表达式

函数声明 `function f(a, b) { ... }` 整体提升，可以在定义之前的代码里调用；函数表达式 `const f = function (a, b) { ... };` 赋给 `const` 时只在赋值之后可用，提前调用抛 `ReferenceError`。两种写法产生的都是值，可以赋给变量、当作参数传递，接收函数的函数把参数称为回调。没有 `return` 的函数返回 `undefined`；函数能读外层变量，外层读不到函数内部的局部变量。

```js
console.log(double(3));   // 6：声明可以在定义前调用
function double(n) { return n * 2; }
// console.log(triple(3)); // ReferenceError：表达式在赋值前不可用
const triple = (n) => n * 3;
[1, 2].forEach((n) => console.log(triple(n))); // 回调由 forEach 决定执行时机
```

## 6.2. 箭头函数

箭头函数写成 `(a, b) => a + b`：单表达式省略 `{}` 与 `return`，写了花括号就必须显式 `return`，漏写返回 `undefined`；一个参数可省略括号，零个参数必须写 `()`。箭头函数没有自己的 `this`，沿用它定义位置外层的 `this`，所以不能用 `new` 调用（抛 `TypeError`）。对象方法需要随调用者变化的 `this`，用方法简写而不是箭头函数。

```js
const double = (n) => { return n * 2; };  // 有大括号必须写 return
console.log(double(3));
const Counter = () => {};
// new Counter();  // TypeError: Counter is not a constructor
```

# 7. Rest 与 Spread

## 7.1. rest 参数

在参数表末尾写 `function f(first, ...rest) { ... }`，`...rest` 把剩余实参收进真数组，可以直接调用数组方法；它必须写在参数表最后，后面再写参数是语法错误；它不参与 `fn.length` 计数；收集到的值仍按值或按引用传入。同一个符号出现在别的位置含义相反：

| 出现位置 | 名称 | 作用 |
| --- | --- | --- |
| 函数参数表末尾 | rest 参数 | 把剩余实参收集成数组 |
| 函数调用、数组或对象字面量中 | spread 语法 | 把数组、可迭代对象或对象展开成列表 |

```js
function sum(first, ...rest) {
  return rest.reduce((acc, n) => acc + n, first);
}
console.log(sum(1, 2, 3)); // 6
// function broken(...rest, last) {} // SyntaxError：rest 必须在最后
```

## 7.2. spread 展开

`fn(...args)` 在调用位置把可迭代对象展开成实参，`[...arr]` 与 `{ ...obj }` 在字面量里做浅拷贝与合并，对象里后写的同名属性覆盖前面的。展开只复制一层，嵌套对象仍与原对象共享引用。在函数调用与数组字面量里展开 `undefined` 或 `null` 会抛 `TypeError`，要兜底就写 `...(arr ?? [])`；对象字面量里展开 `null` 或 `undefined` 被忽略，结果只保留其余属性。

```js
const nums = [3, 5, 1];
console.log(Math.max(...nums));       // 5：展开成参数列表
console.log([0, ...nums, 2]);         // [0, 3, 5, 1, 2]
console.log({ ...{ a: 1, b: 2 }, b: 9 }); // { a: 1, b: 9 }
```

# 8. 对象

## 8.1. 对象字面量与属性访问

对象用字面量写 `const obj = { key: value };`，属性名是字符串，数字键会被自动转成字符串，所以 `obj[1]` 与 `obj["1"]` 是同一条记录。点符号用于固定的属性名，方括号用于动态名字或含空格、连字符的名字，方括号里可以写任意表达式，字面量中也能用 `[expr]: value` 直接算键。`delete obj.key` 删除属性；`"key" in obj` 判断属性是否存在，原型链上的属性也算命中；`obj.key === undefined` 分不清 "属性不存在" 与 "属性值就是 undefined"，要区分就用 `Object.hasOwn`。

```js
const key = "like birds";
const box = { 1: "one", [key]: true };
console.log(box[1] === box["1"]);  // true：数字键转成字符串
delete box[1];
console.log("1" in box, Object.hasOwn(box, key)); // false true
```

## 8.2. 属性简写

属性名与变量名相同时只写一次 `const obj = { name, age };`，省去 `name: name`。函数属性可以写方法简写 `sayHi() {}`，其中的 `this` 指向调用它的对象。箭头函数属性没有自己的 `this`，不适合当对象方法。返回值是对象字面量时要用小括号包住，否则 `{}` 被解析成函数体。

```js
function makeUser(name, age) {
  return { name, age, label() { return `${this.name}:${this.age}`; } };
}
console.log(makeUser("Ada", 36).label());  // "Ada:36"
const bad = (name) => { name };            // 这行的大括号是函数体
console.log(bad("Ada"));                   // undefined
```

## 8.3. 解构

对象解构写 `const { a, b: alias, c = 3 } = obj;`，按属性名取值，可以改名并给默认值，默认值只在取到 `undefined` 时生效，`null` 不触发默认值；数组解构写 `const [x, , y] = arr;`，按位置取值，空位跳过。解构只顾取值，原对象不变。嵌套解构要求中间层存在，否则抛 `TypeError`，常见写法是给中间层默认 `{}` 或改用可选链；函数参数位置也能解构，传 `undefined` 才用默认值。

```js
const user = { name: "Ada", address: { city: "London" } };
const { name, address: { city } = {}, role = "guest" } = user;
console.log(name, city, role);        // Ada London guest
const [first, ...others] = [1, 2, 3];
console.log(first, others);           // 1 [2, 3]
```

## 8.4. 浅拷贝

浅拷贝用 `{ ...obj }`、`[...arr]` 或 `Object.assign(target, source)`，它产生新对象，但属性值是对象时复制的是引用，改嵌套数据两边都变。`Object.assign` 的返回值和第一个参数是同一个对象，所以 `Object.assign({}, defaults, input)` 不改 `defaults`，直接写 `Object.assign(defaults, input)` 会改掉它。

```js
const defaults = { retries: 3, timeout: { ms: 1000 } };
const copy = { ...defaults };
copy.timeout.ms = 5000;            // 改的是共享的嵌套对象
console.log(defaults.timeout.ms);  // 5000
console.log(copy.retries === defaults.retries); // true：这一层是复制的
```

## 8.5. 深拷贝

深拷贝首选 `structuredClone(value)`，它递归复制普通对象、数组、`Date`、`Map`、`Set`、`ArrayBuffer`，保留循环引用与内部的引用共享关系；遇到函数抛 `DataCloneError`；类实例会被复制成普通对象，原型上的 getter 与方法都丢失，自身的可枚举属性照抄，其中用 getter 定义的属性按取值当时的结果写成数据属性。用 `JSON.parse(JSON.stringify(value))` 做 JSON 往返只在数据是纯对象、数组、字符串、数字、布尔、`null` 时可靠，代价见下表。

| 原值 | 经过 JSON 往返之后 |
| --- | --- |
| `undefined`、函数值 | 属性整个消失；出现在数组里变成 `null` |
| `NaN`、`Infinity` | 变成 `null` |
| `Date` | 变成 ISO 字符串，不再是 `Date` 对象 |
| `Map`、`Set`、类实例 | 变成普通对象，丢失原型方法 |
| 循环引用 | `JSON.stringify` 抛 `TypeError` |

```js
const source = { when: new Date(0), nested: { n: 1 } };
const clone = structuredClone(source);
clone.nested.n = 2;
console.log(source.nested.n, clone.when instanceof Date); // 1 true
// structuredClone({ fn() {} }); // DataCloneError：函数不能克隆
console.log(JSON.parse(JSON.stringify({ a: undefined, b: NaN }))); // { b: null }
```

## 8.6. this 与 bind

`this` 不在定义时确定，而由调用方式决定：`obj.method()` 指向 `obj`；把方法取出来直接调用，`this` 是 `undefined`（模块与严格模式下）；`fn.call(obj, x)`、`fn.apply(obj, [x])` 显式指定。把方法当回调传出去会丢掉调用者信息，这是 `this` 出问题的最常见来源，两种修法是 `bind` 或改成箭头函数。`bind` 返回新函数，`this` 永久固定为第一个参数，用 `call` 也改不回来，额外的绑定参数会前置到每次调用的实参之前，原函数不受影响。

```js
const user = { name: "Ada", sayHi() { console.log(this.name); } };
user.sayHi();                  // "Ada"
const detached = user.sayHi;
// detached();                 // TypeError：严格模式下 this 是 undefined
const greet = user.sayHi.bind(user);
setTimeout(greet, 0);          // 定时器回调里 this 仍指向 user
```

# 9. 可选链

## 9.1. 可选链的写法

可选链写成 `obj?.prop`、`obj?.[expr]`、`fn?.()`：左侧为 `null` 或 `undefined` 时整个表达式短路求值并返回 `undefined`，同一条链上后面的访问与调用都不执行，所以 `user.address?.street` 在 `address` 缺失时不抛 `TypeError`。`?.` 不能出现在赋值左侧，`obj?.x = 1` 是语法错误。只在逻辑上允许缺失的位置用：必填字段上滥用会把真正的错误变成 `undefined`，报错位置与出错位置分离。

```js
const user = {};
console.log(user.address?.street);  // undefined，不抛错
console.log(user.getName?.());      // undefined：方法缺失就跳过调用
console.log({ a: 1 }?.["b"]);       // undefined：方括号访问同样短路
```

# 10. 数组

## 10.1. 数组的声明与访问

数组用 `const arr = [1, 2, 3];` 声明，它是特殊对象，通过引用传递。`length` 是最大索引加一，不是元素个数：给高索引赋值会留下空位并撑大 `length`，`arr.length = 0` 清空数组最省事。`arr.at(-1)` 按负索引取尾部元素，`arr[-1]` 读的是名为 `"-1"` 的属性，结果是 `undefined`。

```js
const arr = [];
arr[3] = "x";
console.log(arr.length);  // 4：中间三个位置是空位
console.log(arr.at(-1), arr[-1]); // "x" undefined
arr.length = 0;
```

## 10.2. pop、push、shift 与 unshift

`arr.pop()`、`arr.push(v)`、`arr.shift()`、`arr.unshift(v)` 这四个方法都改原数组：`pop`、`push` 作用在尾部，`shift`、`unshift` 作用在头部。`push` 与 `unshift` 可一次加多个值并返回新的 `length`，`pop` 与 `shift` 返回被移除的元素，空数组上返回 `undefined`；`shift` 与 `unshift` 要移动后面所有元素的索引，大数组上比尾部操作慢。

```js
const stack = [1, 2];
console.log(stack.push(3), stack);    // 3 [1, 2, 3]
console.log(stack.pop(), stack);      // 3 [1, 2]
console.log(stack.unshift(0), stack); // 2 [0, 1]
```

## 10.3. splice、slice 与 concat

`arr.splice(start, deleteCount, ...items)` 直接改原数组，删除若干元素并就地插入新元素，返回被删除元素组成的数组；`deleteCount` 为 `0` 时只插入，`splice(0, arr.length)` 清空原数组。`slice` 返回新数组且不改原数组，`end` 位置的元素不含在内，负索引从尾部算。`concat` 也返回新数组，参数是数组时展开一层。需要保留原数据时不能用 `splice`。

```js
const arr = ["a", "b", "c", "d"];
console.log(arr.slice(1, 3));        // ["b", "c"]
console.log(arr.splice(1, 2, "X"));  // ["b", "c"]
console.log(arr);                    // ["a", "X", "d"]：原数组已被改
```

## 10.4. forEach、map 与 filter

`arr.forEach(fn)` 只做遍历，返回 `undefined`，后面接 `.map` 会抛 `TypeError`，也不能用 `break` 提前结束。`map` 返回等长新数组，回调的返回值成为新元素，漏写 `return` 会得到一排 `undefined`。`filter` 返回新数组，回调返回真值的元素才保留，全不匹配时得到空数组。三者都不修改原数组（回调自己改除外）。

```js
const nums = [1, 2, 3, 4];
console.log(nums.forEach((n) => n * 2));     // undefined
console.log(nums.map((n) => n * 2));         // [2, 4, 6, 8]
console.log(nums.map((n) => { n * 2; }));    // [undefined, undefined, undefined, undefined]
console.log(nums.filter((n) => n % 2 === 0)); // [2, 4]
```

## 10.5. find、findIndex、indexOf 与 includes

`arr.find(fn)` 返回第一个让回调为真的元素，没有则 `undefined`；`findIndex` 返回它的索引，没有则 `-1`；两者一找到就停止遍历，适合在对象数组里按 id 取一条。`indexOf` 与 `includes` 用 `===` 比较，对象与数组只能按引用命中，`NaN` 在 `indexOf` 里找不到而 `includes` 能命中。判断 "没找到" 要比较 `=== undefined` 或 `=== -1`，因为元素本身可能是 `0` 或 `""`。

```js
const users = [{ id: 1 }, { id: 2 }];
console.log(users.find((u) => u.id === 2), users.findIndex((u) => u.id === 3)); // {id:2} -1
const arr = [NaN, { a: 1 }];
console.log(arr.indexOf(NaN), arr.includes(NaN)); // -1 true
```

## 10.6. sort 与 reverse

`arr.sort(compareFn)` 与 `arr.reverse()` 都原地修改并返回同一个数组。`sort` 不传比较函数时把所有元素转成字符串按字典序比较，所以 `[10, 9, 1]` 得到 `[1, 10, 9]`；数字排序必须传 `(a, b) => a - b` 表示升序，比较函数返回负数表示 `a` 在前。按字段排序写 `(a, b) => a.id - b.id`，字符串字段用 `a.name.localeCompare(b.name)`。要保留原数组顺序就先 `[...arr].sort(...)`。

```js
const nums = [10, 9, 1];
console.log([...nums].sort());            // [1, 10, 9]：默认按字符串比较
console.log(nums.sort((a, b) => a - b));  // [1, 9, 10]
console.log(nums.reverse());              // [10, 9, 1]
```

## 10.7. split 与 join

`str.split(separator)` 把字符串切成数组：分隔符为空串时按字符切，不传参数时整串成为唯一元素，连续的分隔符会产生空串元素。`join` 用粘合符把数组拼成字符串，默认逗号，空数组得到空串。两者互逆只在不含连续分隔符的输入上成立。

```js
console.log("a,b,c".split(","));           // ["a", "b", "c"]
console.log("a,b,c".split(",").join(" | ")); // "a | b | c"
console.log("a,,b".split(","));            // ["a", "", "b"]：空字段保留
```

## 10.8. Array.isArray 与 Array.from

数组是对象，`typeof []` 返回 `"object"`，区分数组只能用 `Array.isArray(value)`。`Array.from` 把可迭代对象（数组、字符串、`Map`、`Set`）或类数组对象（有索引和 `length`）转成真数组，之后才能调用数组方法；第二个参数是映射函数，在转换时逐个应用。`for...of` 能直接遍历任何可迭代对象，但拿不到索引。

```js
console.log(Array.isArray([]), typeof []);            // true "object"
console.log(Array.from({ 0: "a", 1: "b", length: 2 })); // ["a", "b"]
console.log(Array.from("hi", (c) => c.toUpperCase()));  // ["H", "I"]
```

# 11. Map 与 Set

## 11.1. Map

`new Map(entries)` 建的键值集合里键保持原值，数字、对象、`NaN` 都能当键，`map.get(1)` 与 `map.get("1")` 是两条不同记录。迭代顺序等于插入顺序，不能按编号取元素，也不能重排。`get` 取不到时返回 `undefined`，判断存在必须用 `has`，因为值本身可能是 `undefined`；`set` 返回 map 本身，可以链式调用。

```js
const visits = new Map();
visits.set("a", 1).set(1, 2);       // 字符串键与数字键各占一条
console.log(visits.get(1), visits.size); // 2 2
console.log([...visits.keys()]);    // ["a", 1]：保持插入顺序
```

## 11.2. Set

`new Set(iterable)` 建的集合里同一个值重复 `add` 只留一份，去重就是 `[...new Set(arr)]`。比较规则与 `===` 一致：对象按引用去重，`NaN` 视为同一个值。可迭代，能配 `for...of` 与 `forEach`，用 `has` 判断某个值在不在集合里，但没有索引，也没有对外暴露的排序手段，需要顺序时先转数组再 `sort`。

```js
const tags = new Set(["js", "js", "web"]);
tags.add("dom");
console.log(tags.size, [...tags]);  // 3 ["js", "web", "dom"]
```

# 12. Object.keys 与 values 与 entries

## 12.1. 对象的遍历

`for (const key in obj)` 的 `key` 是字符串，它遍历自身与原型链上所有可枚举属性，继承来的属性和依赖往原型上添加的字段都会出现，遍历对象自己改用 `Object.keys` 或 `Object.entries`。数组也不要配 `for...in`：拿到的是索引字符串，还可能带上自定义属性。`Object.keys`、`values`、`entries` 都只处理对象自身的可枚举属性并返回真数组，能直接接 `map`、`filter`、`length`；`entries` 返回 `[key, value]`，可解构遍历，也能直接送给 `new Map()`，反向操作是 `Object.fromEntries`。

```js
const obj = Object.create({ inherited: true });
obj.own = 1;
for (const key in obj) console.log(key);  // "own" 然后 "inherited"
console.log(Object.keys(obj));            // ["own"]
console.log(Object.fromEntries([["x", 1]])); // { x: 1 }
```

# 13. JSON

## 13.1. JSON 的序列化与解析

`JSON.stringify(value)` 生成符合 JSON 规范的字符串，用于存储或传输；`JSON.parse(text)` 把它还原成普通对象。JSON 的键和字符串必须用双引号，不允许注释、尾随逗号和 `undefined`，所以 `JSON.parse("{'a':1}")` 抛 `SyntaxError`，处理外部数据要包 `try/catch`。合法 JSON 也包括数字、字符串、布尔与 `null`，`parse` 的返回值不一定是对象；往返过程中的数据损失见 8.5 的对照表。

```js
const text = JSON.stringify({ page: 0, tags: ["js"] });
console.log(text);                        // {"page":0,"tags":["js"]}
console.log(JSON.parse(text).page);       // 0
try { JSON.parse("{'a':1}"); } catch (e) { console.log(e.name); } // SyntaxError
```

# 14. Date

## 14.1. 创建 Date

`new Date()` 无参得到当前时刻，其他几种参数形式分别接受毫秒数、ISO 字符串或年月日字段：数字参数被当作自 UTC 1970-01-01 起经过的毫秒数，1970 年之前的日期是负数；字符串参数按 ISO 格式解析，格式不合法得到 `Invalid Date`，取时间戳为 `NaN`。多参数形式按本地时区解释，`month` 从 `0` 开始计数，只有年和月必填，其余默认 `1` 或 `0`。只写日期的字符串（`"2026-01-01"`）按 UTC 零点解析，在负时区会显示成前一天。

```js
console.log(new Date(0).toISOString());       // "1970-01-01T00:00:00.000Z"
console.log(new Date(2026, 0, 1).getMonth()); // 0：0 代表一月
console.log(new Date("bad").getTime());       // NaN
```

## 14.2. 读写时间组件

`date.getFullYear()`、`date.getMonth()` 这样的 `get*` 方法按本地时区读取组件，方法名里加 `UTC` 的按 UTC+0 读取；`set*` 修改原对象并返回新的时间戳，接受超范围的数值并自动校准，可以借此做日期加减。组件含义见下表：

| 方法 | 返回 |
| --- | --- |
| `getFullYear()` | 四位年份 |
| `getMonth()` | 月份，0 到 11 |
| `getDate()` | 当月第几天，1 到 31 |
| `getDay()` | 星期，0（周日）到 6（周六），不受区域设置影响 |
| `getTime()` | 时间戳毫秒数，没有 UTC 变体 |
| `getTimezoneOffset()` | UTC 与本地时区的分钟差，没有 UTC 变体 |

```js
const d = new Date(2026, 0, 31);
d.setDate(32);
console.log(d.getMonth(), d.getDate()); // 1 1：1 月 32 日校准成 2 月 1 日
```

## 14.3. 时间戳

时间戳是自 1970-01-01 UTC 起的毫秒数，1970 年之前的日期为负。`Date.now()` 直接返回当前时间戳，不必创建中间对象；`+date` 与 `date.getTime()` 等价。时间差直接相减再换算单位，比手工拆分年月日可靠，也自然处理跨月跨年。

```js
const start = Date.now();
for (let i = 0, sink = 0; i < 1000; i++) sink += i;  // 一段同步计算
console.log(Date.now() - start >= 0, +new Date(1000) === 1000); // true true
```
