---
title: JavaScript Basics
tags:
  - Frontend
  - JavaScript
categories:
  - Frontend
date: 2026-06-24 00:00:00
katex: true
---

# JavaScript Basics

## 易错点先记

| 点 | 结论 |
|---|---|
| `let` / `const` / `var` | 现代 JS 优先 `const`，需要重新赋值时用 `let`，尽量不用 `var`。 |
| `typeof null` | 返回 `'object'`，这是历史遗留问题，不表示 `null` 真的是普通对象。 |
| `Number(undefined)` | 得到 `NaN`。 |
| `Boolean("0")` 和 `Boolean(" ")` | 都是 `true`，因为非空字符串为真。 |
| `??` | 只把 `null` / `undefined` 当作“没有值”。 |
| 函数声明 vs 函数表达式 | 函数也是值，可以赋给变量，也可以作为回调传入另一个函数。 |
| 箭头函数 | 适合短回调，但没有自己的 `this`。涉及对象方法或构造器时要小心。 |

## 基础内容

## 1. hello, world!

在.html中：
- 我们可以使用一个 `<script>` 标签将 JavaScript 代码添加到页面中。
- 外部的脚本可以通过 `<script src="path/to/script.js"></script>` 的方式插入。

## 2. variables

我们可以使用 `var`、`let` 或 `const` 声明变量来存储数据。

- `let` — 现代的变量声明方式。
- `const` — 类似于 `let`，但是变量的值无法被修改。

## 3. types

JavaScript 中有八种基本的数据类型。

- 七种原始数据类型（基本数据类型）：
    - `number` 用于任何类型的数字：整数或浮点数，在 `±(253-1)` 范围内的整数。
    - `bigint` 用于任意长度的整数。可以通过将 `n` 附加到整数字段的末尾来创建 `BigInt` 值。
    - `string` 用于字符串：一个字符串可以包含 0 个或多个字符，所以没有单独的单字符类型。
    - `boolean` 用于 `true` 和 `false`。
    - `null` 用于未知的值 —— 只有一个 `null` 值的独立类型。JavaScript 中的 `null` 仅仅是一个代表“无”、“空”或“值未知”的特殊值。
    - `undefined` 用于未定义的值 —— 只有一个 `undefined` 值的独立类型。如果一个变量已被声明，但未被赋值，那么它的值就是 `undefined`
    - `symbol` 用于唯一的标识符。
- 以及一种非原始数据类型（复杂数据类型）：
    - `object` 用于更复杂的数据结构。

(`object` 会在 [对象](https://zh.javascript.info/object) 一章中介绍 。
 `symbol` 要在学完 `object` 类型后再学习。)

反引号\`\`是 **功能扩展** 引号。它们允许我们通过将变量和表达式包装在 `${…}` 中，来将它们嵌入到字符串中。例如

```javascript
let name = "John";
// 嵌入一个变量
alert(`Hello,${name}!`);
// Hello, John!

// 嵌入一个表达式` `
alert(`the result is ${1+2}`);
// the result is 3
```

`typeof` 运算符返回参数的类型。当我们想要分别处理不同类型值的时候，或者想快速进行数据类型检验时，非常有用。

对 `typeof x` 的调用会以字符串的形式返回数据类型。

## 4. alert-prompt-confirm

### 4.1 alert
### 4.2 prompt

`prompt(*title*,[*default*]);`

浏览器会显示一个带有文本消息的模态窗口，还有 input 框和确定/取消按钮。

`title`显示给用户的文本

`default`可选的第二个参数，指定 input 框的初始值

上述语法中 `default` 周围的方括号表示该参数是可选的，不是必需的。

`prompt` 将返回用户在 `input` 框内输入的文本，即`string`类型，如果用户取消了输入，则返回 `null`。

`let a=prompt("input",0);`

### 4.3 confirm

`confirm` 函数显示一个带有 `question` 以及确定和取消两个按钮的模态窗口。

点击确定返回 `true`，点击取消返回 `false`。

## 5. type-conversions

有三种常用的类型转换：转换为 string 类型、转换为 number 类型和转换为 boolean 类型。

**字符串转换** —— 转换发生在输出内容的时候，也可以通过 `String(*value*)` 进行显式转换。原始类型值的 string 类型转换通常是很明显的。

**数字型转换** —— 转换发生在进行算术操作时，
如`'6'/'2'`，不同类型的比较操作（`===` 除外），如`'2'>1`。也可以通过 `Number(*value*)` 进行显式转换。或者`+*value*`

数字型转换遵循以下规则：

| 值              | 变成……                                                                     |
| -------------- | ------------------------------------------------------------------------ |
| `undefined`    | `NaN`                                                                    |
| `null`         | `0`                                                                      |
| `true / false` | `1 / 0`                                                                  |
| `string`       | “按原样读取”字符串，两端的空白字符（空格、换行符 `\n`、制表符 `\t` 等）会被忽略。空字符串变成 `0`。转换出错则输出 `NaN`。 |

**布尔型转换** —— 转换发生在进行逻辑操作时，也可以通过 `Boolean(value)` 进行显式转换。

布尔型转换遵循以下规则：

| 值                                     | 变成……    |
| ------------------------------------- | ------- |
| `0`, `null`, `undefined`, `NaN`, `""` | `false` |
| 其他值                                   | `true`  |

上述的大多数规则都容易理解和记忆。人们通常会犯错误的值得注意的例子有以下几个：

- 对 `undefined` 进行数字型转换时，输出结果为 `NaN`，而非 `0`。
- 对 `"0"` 和只有空格的字符串（比如：`" "`）进行布尔型转换时，输出结果为 `true`。

## 6. nullish-coalescing-operator

`a??b`
- 如果 `a` 是已定义的，则结果为 `a`，
- 如果 `a` 不是已定义的，则结果为 `b`。

可以使用 `??` 序列从一系列的值中选择出第一个非 `null/undefined` 的值

## 7. while-for

### 7.6 [break/continue 标签](https://zh.javascript.info/while-for#breakcontinue-biao-qian)

有时候我们需要一次从多层嵌套的循环中跳出来。

`break <labelName>` 语句跳出循环：

```js
outer: 
for(;;){
	for(;;){
		//...
		break outer;
	}
}

```

`continue` 指令也可以与标签一起使用。在这种情况下，执行跳转到标记进行下一次循环。

## 8. function-basics

函数声明方式如下所示：

`function *name*(*param1*,*param2*,...,*paramN*){/*code*/}`

- 作为参数传递给函数的值，会被复制到函数的局部变量。
- 函数可以访问外部变量。但它只能从内到外起作用。函数外部的代码看不到函数内的局部变量。
- 函数可以返回值。如果没有返回值，则其返回的结果是 `undefined`。

## 9. function-expressions

### 9.1 **函数是一个值**

在 JavaScript 中，函数是一种特殊的值。

函数的类型是对象。

一个容易理解的方式是把函数想象成可被调用的“行为对象（action object）”。我们不仅可以调用它们，还能把它们当作对象来处理：增/删属性，按引用传递等

一些属性：

- `name` —— 函数的名字。通常取自函数定义，但如果函数定义时没设定函数名，JavaScript 会尝试通过函数的上下文猜一个函数名（例如把赋值的变量名取为函数名）。
- `length` —— 函数定义时的入参的个数。Rest 参数不参与计数。

前面章节使用的语法称为 **函数声明**：

例如：

```js
function f(){
	alert("hello");
	}
```

另一种创建函数的语法称为 **函数表达式**。

它允许我们在任何表达式的中间创建一个新函数。

例如：

```js
let f=function(){
	alert("hello");
}
```

无论函数是如何创建的，函数都是一个**值**。上面的两个示例都在 `f` 变量中存储了一个函数。

一个函数是表示一个“行为”的值。字符串或数字等常规值代表 **数据**。函数可以被视为一个 **行为（action）**。我们可以在变量之间传递它们，并在需要时运行。

`alert(f)//显示函数代码，但并不会运行函数`

的确，在某种意义上说一个函数是一个特殊值，我们可以像 `f()` 这样调用它。

但它依然是一个值，所以我们可以像使用其他类型的值一样使用它，比如复制函数到其他变量。

### 9.2 回调函数

我们写一个包含三个参数的函数 `ask(question, yes, no)`：

`question`

关于问题的文本

`yes`

当回答为 “Yes” 时，要运行的脚本

`no`

当回答为 “No” 时，要运行的脚本

函数需要提出 `question`（问题），并根据用户的回答，调用 `yes()` 或 `no()`：

```js
let ask=function(question,yes,no){
	if(confirm(question)){
		yes();
	}
	else{
		no();
	}
};

let ok=function(){
	alert("OK");
}
let cancel=function(){
	alert("Cancel");
}

//使用
ask("Do you agree?",ok,cancel);
```

`ask` 的两个参数值 `ok` 和 `cancel` 可以被称为 **回调函数** 或简称 **回调**。

主要思想是我们传递一个函数，并期望在稍后必要时将其“回调”。在我们的例子中，`ok` 是回答 “yes” 的回调，`cancel` 是回答 “no” 的回调。

我们可以使用函数表达式来编写一个等价的、更简洁的函数：

```js
let ask=function(question,yes,no){
	if(confirm(question)){
		yes();
	}
	else{
		no();
	}
};

//使用
ask("Do you agree?",function(){alert("OK");},function(){alert("Cancel");});
```

这里直接在 `ask(...)` 调用内进行函数声明。这两个函数没有名字，所以叫 **匿名函数**。这样的函数在 `ask` 外无法访问（因为没有对它们分配变量）。

## 10. arrow-function-basics

创建函数还有另外一种非常简单的语法，并且这种方法通常比函数表达式更好。

它被称为“箭头函数”。

例如：

```js
let sum= function(a,b){
	return a+b;
}

```

可以写为：

```js
let sum=(a,b)=>a+b;
```

`let func=(arg1,arg2,...,argN)=>expression`

这里创建了一个函数 `func`，它接受参数 `arg1..argN`，然后使用参数对右侧的 `expression` 求值并返回其结果。

如果我们只有一个参数，还可以省略掉参数外的圆括号，使代码更短。

如果没有参数，括号则是空的（但括号必须保留）

有时我们需要更复杂一点的函数，比如带有多行的表达式或语句。在这种情况下，我们可以使用花括号将它们括起来。主要区别在于，用花括号括起来之后，需要包含 `return` 才能返回值（就像常规函数一样）。

`(...args) => { body }`

```js
let sum=(a,b)=>{
 let res=a+b;
return res;
}

```

## `??` 与 `||` 的区别

`??` 只在左侧为 `null` 或 `undefined` 时使用右侧；`||` 会把所有 falsy 值都当作“没有”。

```js
0 || 10;      // 10
0 ?? 10;      // 0

"" || "x";   // "x"
"" ?? "x";   // ""
```

这对表单值、计数器和分页参数很重要：`0`、空字符串可能是合法输入，不应该被误替换。
