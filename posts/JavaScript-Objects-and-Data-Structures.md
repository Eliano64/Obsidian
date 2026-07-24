---
title: JavaScript Objects and Data Structures
tags:
categories:
  - Frontend
date: 2026-06-26 00:00:00
katex: true
---

# JavaScript Objects and Data Structures

## 结构总览

| 主题 | 关键问题 | 常用场景 |
|---|---|---|
| Object | 键值对、引用、拷贝、属性访问 | 配置对象、实体建模、接口数据 |
| Array | 有序集合及数组方法 | 列表渲染、批量转换、筛选、排序 |
| Iterable | `for..of` 与迭代协议 | 自定义可遍历对象、把类数组转数组 |
| Map / Set | 更明确的键值集合 / 去重集合 | 缓存、计数、去重、保留插入顺序 |
| WeakMap / WeakSet | 弱引用集合 | 与对象生命周期绑定的缓存/标记 |
| Date | 时间戳、日期读写、自动校准 | 时间展示、时间计算 |
| JSON | 对象与字符串互转 | 存储、网络传输、请求/响应 |

## 1. Object

## 11. object

### 11.1 属性

对象用来存储键值对和更复杂的实体。

我们可以通过使用带有可选 **属性列表** 的花括号 `{…}` 来创建对象。一个属性就是一个键值对（“key: value”），其中键（`key`）是一个字符串（也叫做属性名），值（`value`）可以是任何值。

创建一个空的对象:

```js
let usr=new Object();
let usr={};
```

我们可以在创建对象的时候，立即将一些属性以键值对的形式放到 `{...}` 中。

```js
let usr={
	name:"John",
	age:30,
	getUsrMsg(){
		alert(this.name);
	},
	//在对象方法中避免使用箭头函数，因为`this`指向的对象会有不同
};
```

列表中的最后一个属性应以逗号结尾，便于添加、删除和移动。

使用点符号`.`或`[]`访问属性值。

可以直接添加属性

```js
usr.isAdmin=true;
```

可以用 `delete` 操作符移除属性：

```js
delete user.age;
```

我们也可以用多字词语来作为属性名，但必须给它们加上引号：

```js
let usr={
	"like birds":true,
};

alert(usr["like birds"]);
```

通过`[]`访问还支持通过任意表达式来获取属性名：

```js
let key="like birds";
usr[key]=true;
//等价于usr["like birds"]=true;
```

这个也等价于

```js
let usr={
	[key]:true;
}
```

大部分时间里，当属性名是已知且简单的时候，就使用点符号。如果我们需要一些更复杂的内容，那么就用方括号。

### 11.2 属性值简写

```js
let initUser=(name,age)=>{
	return {
		name,
		age,
	}
};

let usr=initUser("John",30);
alert(usr.name+" "+usr.age); //John 30
```

### 11.3 `in` 

`"*keyName*" in *object*`:返回`true`/`false`

`in` 的左边必须是 **属性名**。通常是一个带引号的字符串。

如果我们省略引号，就意味着左边是一个变量

为了遍历一个对象的所有键（key），可以使用一个特殊形式的循环：`for..in`

`key`是变量名，类型是string

```js
for (let key in object){

}
```


JavaScript 中还有很多其他类型的对象：

- `Array` 用于存储有序数据集合，
- `Date` 用于存储时间日期，
- `Error` 用于存储错误信息。
- ……等等。



## 12. 对象的拷贝

对象与原始类型的根本区别之一是，对象是“通过引用”存储和复制的，而原始类型：字符串、数字、布尔值等 —— 总是“作为一个整体”复制。

所以，使用 const 声明的对象，它的属性的值是可以被修改的。

`=`:原始类的复制，对象的引用复制。

### 12.1 复制一个对象

通过遍历已有对象的属性，并在原始类型值的层面复制它们，以实现对已有对象结构的复制。

```js
Object.assign(dest,[src1,src2,...,srcN])
```

- 第一个参数 `dest` 是指目标对象。
- 更后面的参数 `src1, ..., srcN`（可按需传递多个参数）是源对象。
- 该方法将所有源对象的属性拷贝到目标对象 `dest` 中。换句话说，从第二个开始的所有参数的属性都被拷贝到第一个参数的对象中。
- 调用结果返回 `dest`。
- 如果被拷贝的属性的属性名已经存在，那么它会被覆盖：

```js
let usr=new Object();
let src1={canView:true};
let src2={canEdit:true};

Object.assign(usr,src1,src2);
//src1 src2 的属性全部拷贝到usr中
```

### 12.2 深拷贝

 `user` 的所有属性均为原始类型，用上面的浅拷贝即可。但属性可以是对其他对象的引用。

我们应该使用一个拷贝循环来检查对象的每个值。如果它是一个对象，那么也复制它的结构。这就是所谓的“深拷贝”。

我们可以使用递归来实现它。

```js
let usr={
	sizes:{
		height:182,
		width:50,
	},
};

let deepClone=(src)=>{
	let clone={};
	for( let key in src){
		if (typeof src[key] === "object" && src[key] !== null){
			clone[key]=deepClone(src[key]);
		}
		else{
			clone[key]=src[key];
		}
	}
	return clone;
}

clone=deepClone(usr);
```

或者

`let clone=_.cloneDeep(usr)`



## 13. constructor-new

构造函数在技术上是常规函数（除了箭头函数，它没有自己的 `this`）。不过有两个约定：

1. 它们的命名以大写字母开头。
2. 它们只能由 `"new"` 操作符来执行。

```js
let Usr=function(name){
	this.name=name;
	this.isAdmin=false;
}//在构造函数中避免使用箭头函数，因为它们不会正确绑定 `this`。

let usr1=new Usr("Jack");
```

## 14. optional-chaining

### 14.1 `?.`

可选链 `?.` 是一种访问嵌套对象属性的安全的方式。即使中间的属性不存在，也不会出现错误。

如果可选链 `?.` 前面的值为 `undefined` 或者 `null`，它会停止运算并返回 `undefined`，否则返回值

```js
let usr={};
alert(usr.address?.street); // undefined,但不报错
```

我们可以使用 `?.` 来安全地读取或删除，但不能写入。

可选链 `?.` 不能用在赋值语句的左侧。

>我们应该只将 `?.` 使用在一些东西可以不存在的地方。
>
 例如，如果根据我们的代码逻辑，`user` 对象必须存在，但 `address` 是可选的，那么我们应该这样写 `user.address?.street`，而不是这样 `user?.address?.street`。
>
 那么，如果 `user` 恰巧为 undefined，我们会看到一个编程错误并修复它。否则，如果我们滥用 `?.`，会导致代码中的错误在不应该被消除的地方消除了，这会导致调试更加困难。

### 14.2 `?.()`

`?.()` 用于调用一个可能不存在的方法

```js
let usr={
 admin(){
	alert("1");
 },
} ;

let guest={};

usr.admin?.();//1
guest.admin?.();//(无事发生)
```

### 14.3 `?.[]`

使用方括号 `[]` 而不是点符号 `.` 来访问属性，语法 `?.[]` 也可以使用

## 15. symbol

根据规范，只有两种原始类型可以用作对象属性键：

- 字符串类型
- symbol 类型

否则，如果使用另一种类型，例如数字，它会被自动转换为字符串。所以 `obj[1]` 与 `obj["1"]` 相同，而 `obj[true]` 与 `obj["true"]` 相同。

### 15.1 symbol

“symbol” 值表示唯一的标识符。

可以使用 `Symbol()` 来创建这种类型的值：

```js
let id=Symbol();
```

创建时，我们可以给 symbol 一个描述（也称为 symbol 名），这在代码调试时非常有用：

```js
let id=Symbol("id");
```

symbol 保证是唯一的。即使我们创建了许多具有相同描述的 symbol，它们的值也是不同。描述只是一个标签，不影响任何东西。

symbol不会被自动转换（例如调用`alert()`）为string，需要`.toString()`或`.description`

### 15.2 "隐藏"属性

symbol 允许我们创建对象的“隐藏”属性，代码的任何其他部分都不能意外访问或重写这些属性。

```js
let usr={};
let id=Symbol("id");
usr[id]=1;
```

如果我们要在对象字面量 `{...}` 中使用 symbol，则需要使用方括号把它括起来。

```js
let usr={
	[id]:1,
};
```

这是因为我们需要变量 `id` 的值作为键，而不是字符串 “id”。

symbol 在 for…in 中会被跳过

但相反，[Object.assign](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 会同时复制字符串和 symbol 属性

### 15.3 全局symbol

想要名字相同的 symbol 具有相同的实体。例如，应用程序的不同部分想要访问的 symbol `"id"` 指的是完全相同的属性。

为了实现这一点，这里有一个 **全局 symbol 注册表**。我们可以在其中创建 symbol 并在稍后访问它们，它可以确保每次访问相同名字的 symbol 时，返回的都是相同的 symbol。

要从注册表中读取（不存在则创建）symbol，请使用 `Symbol.for(key)`。

该调用会检查全局注册表，如果有一个描述为 `key` 的 symbol，则返回该 symbol，否则将创建一个新 symbol（`Symbol(key)`），并通过给定的 `key` 将其存储在注册表中。

```js
let id=Symbol.for("id");

let id2=Symbol.for("id");

alert(id===id2);
//true
```

对于全局 symbol，`Symbol.for(key)` 按名字返回一个 symbol。相反，通过全局 symbol 返回`key`的名字，我们可以使用 `Symbol.keyFor(sym)`

```js
alert(Symbol.keyFor(id2));//id
```

`Symbol.keyFor` 内部使用全局 symbol 注册表来查找 symbol 的键。所以它不适用于非全局 symbol。如果 symbol 不是全局的，它将无法找到它并返回 `undefined`。

## 2. Array

## 16. array

数组是一种特殊的对象，是通过引用来复制的。

### 16.1 声明

创建一个空数组：

```js
let arr = new Array();
let arr= [];
```

`let arr=[elem1,elem2,...,elemN,];`

通过`[]`下标访问

`length` 属性的值是数组中元素的总个数

准确来说，它实际上不是数组里元素的个数，而是最大的数字索引值加一。

```js
let arr=[];
arr[123]=1;

alert(arr.length);//124
```

清空数组最简单的方法就是：`arr.length = 0;`

也可以用 `alert` 来显示整个数组。即数组有自己的 `toString` 方法的实现，会返回以逗号隔开的元素列表。

数组可以存储任何类型的元素。

数组里的项也可以是数组。我们可以将其用于多维数组

 `fruits.at(i)`：
 - 如果 `i >= 0`，则与 `arr[i]` 完全相同。
- 对于 `i` 为负数的情况，它则从数组的尾部向前数。



### 16.2 [pop/push, shift/unshift 方法](https://zh.javascript.info/array#poppushshiftunshift-fang-fa)

**作用于数组末端的方法：**

`pop` 取出并返回数组的最后一个元素
`push` 在数组末端添加元素

**作用于数组首端的方法：**

`shift` 取出数组的第一个元素并返回它
`unshift` 在数组的首端添加元素

`push` 和 `unshift` 方法都可以一次添加多个元素

### 16.3 循环

对于数组，除了`for(let i=0;i<arr.length;i++)`，还有一种循环：

```js
for(let elem of arr){

}
```

`for..of` 不能获取当前元素的索引，只是获取元素值

**通常来说，我们不应该用 `for..in` 来处理数组。**

## 17. array-methods

### 17.1 `splice`

```js
let arr = ["I", "study", "JavaScript"];  
  
arr.splice(1, 1); // 从索引 1 开始删除 1 个元素  
  
alert( arr ); // ["I", "JavaScript"]
```

```js
let arr = ["I", "study", "JavaScript", "right", "now"];  
  
// 删除数组的前三项，并使用其他内容代替它们  
arr.splice(0, 3, "Let's", "dance");  
  
alert( arr ) // 现在 ["Let's", "dance", "right", "now"]
```

```js
let arr = ["I", "study", "JavaScript", "right", "now"];  
  
// 删除前两个元素  
let removed = arr.splice(0, 2);  
  
alert( removed ); // "I", "study" <-- 被从数组中删除了的元素
```

```js
let arr = ["I", "study", "JavaScript"];  
  
// 从索引 2 开始  
// 删除 0 个元素  
// 然后插入 "complex" 和 "language"  
arr.splice(2, 0, "complex", "language");  
  
alert( arr ); // "I", "study", "complex", "language", "JavaScript"
```

### 17.2 `slice`

`arr.slice([start], [end])`

它会返回一个新数组，将所有从索引 `start` （默认为0）到 `end`（不包括 `end`）（默认为`arr.length`）的数组项复制到一个新的数组。`start` 和 `end` 都可以是负数，在这种情况下，从末尾计算索引。

### 17.3 `concat`

`let newArr=arr.concat(arg1, arg2...)`

如果参数 `argN` 是一个数组，那么复制其中的所有元素。否则，将复制参数本身。

但是，如果对象具有 `Symbol.isConcatSpreadable` 属性，那么它就会被 `concat` 当作一个数组来处理：此对象中的元素将被逐一添加

### 17.4 `forEach`

[arr.forEach](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 方法允许为数组的每个元素都运行一个函数


`arr.forEach(*function*);`

```js
["Bilbo", "Gandalf", "Nazgul"].forEach((item, index, array) => {  
alert(`${item} is at index ${index} in ${array}`);  
});//数组的每个元素在目标数组中的位置
```

### 17.5 `indexOf`/`lastIndexOf`/`includes`

- `arr.indexOf(item, from)` —— 从索引 `from` 开始搜索 `item`，如果找到则返回索引，否则返回 `-1`。
- `arr.includes(item, from)` —— 从索引 `from` 开始搜索 `item`，如果找到则返回 `true`（译注：如果没找到，则返回 `false`）。

`indexOf` 和 `includes` 使用严格相等 `===` 进行比较

如果我们想检查数组中是否包含元素 `item`，并且不需要知道其确切的索引，那么 `arr.includes` 是首选。

方法 [arr.lastIndexOf](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/lastIndexOf) 与 `indexOf` 相同，但从右向左查找。

### 17.6 `find` / `findIndex` / `findLastIndex`

找到具有特定条件的对象

```js
let result = arr.find(function(item, index, array) {  
//... 
});
```

```java
let users = [  
{id: 1, name: "John"},  
{id: 2, name: "Pete"},  
{id: 3, name: "Mary"}  
];  
  
let user = users.find(item => item.id == 1);  
  
alert(user.name); // John
```

[arr.findIndex](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex) 方法（与 `arr.find`）具有相同的语法，但它返回找到的元素的索引，而不是元素本身。如果没找到，则返回 `-1`。

[arr.findLastIndex](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/findLastIndex) 方法类似于 `findIndex`，但从右向左搜索，类似于 `lastIndexOf`。

### 17.7 `filter`

`find` 方法搜索的是使函数返回 `true` 的第一个（单个）元素。

如果需要匹配的有很多，我们可以使用 [arr.filter(fn)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)。

语法与 `find` 大致相同，但是 `filter` 返回的是所有匹配元素组成的数组

```js
let result = arr.filter(function(item, index, array) {  
// 如果 true item 被 push 到 results，迭代继续 // 如果什么都没找到，则返回空数组
});
```

### 17.8 `map`

[arr.map](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 方法是最有用和经常使用的方法之一。

它对数组的每个元素都调用函数，并返回结果数组。

语法：

```js
let result = arr.map(function(item, index, array) {  
// 返回新值而不是当前元素  
})
```

### 17.9 `sort`

[arr.sort](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) 方法对数组进行 **原位（in-place）** 排序，更改元素的顺序。(译注：原位是指在此数组内，而非生成一个新数组。)

它还返回排序后的数组，但是返回值通常会被忽略，因为修改了 `arr` 本身

**这些元素默认情况下被按字符串进行排序。**

要使用我们自己的排序顺序，我们需要提供一个函数作为 `arr.sort()` 的参数。

```js
function compareNumeric(a, b) {  
	return a-b;
}  
  
let arr = [ 1, 2, 15 ];  
  
arr.sort(compareNumeric);  
  
alert(arr);  // 1, 2, 15
```

比较函数只需要返回一个正数表示“大于”，一个负数表示“小于”。

```js
arr.sort((a,b)=>a-b);
```

### 17.10 `reverse`

[arr.reverse](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse) 方法用于原位颠倒 `arr` 中元素的顺序。

### 17.11 `split`/`join`

[str.split(delim)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/String/split) 方法通过给定的分隔符 `delim` 将字符串分割成一个数组。

```js
let names = 'Bilbo, Gandalf, Nazgul';  
  
let arr = names.split(', ');  
  
for (let name of arr) {  
alert( `A message to ${name}.` ); // A message to Bilbo（和其他名字）  
}
```

调用带有空参数 `s` 的 `split(s)`，会将字符串拆分为字母数组

[arr.join(glue)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/join) 与 `split` 相反。它会在它们之间创建一串由 `glue` 粘合的 `arr` 项，返回一个string

```js
let arr = ['Bilbo', 'Gandalf', 'Nazgul'];  
  
let str = arr.join(';'); // 使用分号 ; 将数组粘合成字符串  
  
alert( str ); // Bilbo;Gandalf;Nazgul
```

### 17.12 `Array.isArray`

数组是基于对象的，不构成单独的语言类型。

所以 `typeof` 不能帮助从数组中区分出普通对象

但是数组经常被使用，因此有一种特殊的方法用于判断：[Array.isArray(value)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray)。如果 `value` 是一个数组，则返回 `true`；否则返回 `false`

## 3. Iterable

## 17. Iterable

### 17.1 `Symbol.iterator`

**可迭代（Iterable）** 对象是数组的泛化。这个概念是说任何可以被定制为可在 `for..of` 循环中使用的对象。

数组和字符串是使用最广泛的内建可迭代对象。

自定义：

```js
let range = {  
	from: 1,  
	to: 5  
};  
  
// 1. for..of 调用首先会调用这个：  
range[Symbol.iterator] = function() {  
  
// ……它返回迭代器对象（iterator object）：  
// 2. 接下来，for..of 仅与下面的迭代器对象一起工作，要求它提供下一个值  
	return {  
		current: this.from,  
		last: this.to,  
  
		// 3. next() 在 for..of 的每一轮循环迭代中被调用  
		next() {  
		// 4. 它将会返回 {done:.., value :...} 格式的对象  
			if (this.current <= this.last) {  
			return { done: false, value: this.current++ };  
			} else {  
				return { done: true };  
				}  
		}  
	};  
};  
  
// 现在它可以运行了！  
for (let num of range) {  
	alert(num); // 1, 然后是 2, 3, 4, 5  
}
```

或者

```js
let range = {  
	from: 1,  
	to: 5,  
	[Symbol.iterator]() {  
		this.current = this.from;  
		return this;  
	},  
  
	next() {  
		if (this.current <= this.to) {  
			return { done: false, value: this.current++ };  
		} else {  
			return { done: true };  
			}  
		}  
};  
  
for (let num of range) {  
	alert(num); // 1, 然后是 2, 3, 4, 5  
}
```

底层原理：

```js
let iterator = str[Symbol.iterator]();  
  
while (true) {  
	let result = iterator.next();  
	if (result.done) break;  
	alert(result.value); // 一个接一个地输出字符  
}
```

### 17.2 iterable & array-like&`Array.from`

- **Iterable** 如上所述，是实现了 `Symbol.iterator` 方法的对象。
- **Array-like** 是有索引和 `length` 属性的对象，所以它们看起来很像数组。

```js
let arrayLike = { // 有索引和 length 属性 => 类数组对象  
	0: "Hello",  
	1: "World",  
	length: 2  
};
```

一个可迭代对象也许不是类数组对象。反之亦然，类数组对象可能不可迭代。

全局方法 [Array.from](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Array/from) 可以接受一个可迭代或类数组的值，并从中获取一个“真正的”数组。然后我们就可以对其调用数组方法了。

```js
let arr = Array.from(arrayLike); // (*)  
alert(arr.pop()); // World（pop 方法有效）
```

```js
let arr = Array.from(range);  
alert(arr); // 1,2,3,4,5 （数组的 toString 转化方法生效）

```

## 4. Map 与 Set

## 18 map-set

在 `Map` 和 `Set` 中迭代总是按照值插入的顺序进行的，所以我们不能说这些集合是无序的，但是我们不能对元素进行重新排序，也不能直接按其编号来获取元素。

### 18.1 Map

[Map](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Map) 是一个带键的数据项的集合。

它的方法和属性如下：

- `new Map()` —— 创建 map。
- `map.set(key, value)` —— 根据键存储值。返回 map 本身，所以我们可以进行“链式”调用
- `map.get(key)` —— 根据键来返回值，如果 `map` 中不存在对应的 `key`，则返回 `undefined`。
- `map.has(key)` —— 如果 `key` 存在则返回 `true`，否则返回 `false`。
- `map.delete(key)` —— 删除指定键值对。
- `map.clear()` —— 清空 map。
- `map.size` —— 返回当前元素个数。

与对象不同，**键不会被转换成字符串**。

**Map 还可以使用对象作为键。**

如果要在 `map` 里使用循环，可以使用以下三个方法：

- `map.keys()` —— 遍历并返回一个包含所有键的可迭代对象，
- `map.values()` —— 遍历并返回一个包含所有值的可迭代对象，
- `map.entries()` —— 遍历并返回一个包含所有实体 `[key, value]` 的可迭代对象，`for..of` 在默认情况下使用的就是这个。

迭代的顺序与插入值的顺序相同。与普通的 `Object` 不同，`Map` 保留了此顺序。

`Map` 有内建的 `forEach` 方法，与 `Array` 类似

如果我们想从一个已有的普通对象（plain object）来创建一个 `Map`，那么我们可以使用内建方法 [Object.entries(obj)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/entries)，该方法返回对象的键/值对数组，该数组格式完全按照 `Map` 所需的格式。

```js
let obj = {  
	name: "John",  
	age: 30  
};  
  
let map = new Map(Object.entries(obj));  
  
alert( map.get('name') ); // John
```

`Object.fromEntries` 方法的作用是相反的：给定一个具有 `[key, value]` 键值对的数组，它会根据给定数组创建一个对象

```js
let prices = Object.fromEntries([  
	['banana', 1],  
	['orange', 2],  
	['meat', 4]  
]);  
  
// 现在 prices = { banana: 1, orange: 2, meat: 4 }  
  
alert(prices.orange); // 2
```

### 18.2 WeakMap

将数据放到 `WeakMap` 中，那么当该键为`null`后，它以及它对应的值会被自动清除。

`WeakMap` 和 `Map` 的第一个不同点就是，`WeakMap` 的键必须是对象

`WeakMap` 不支持迭代以及 `keys()`，`values()` 和 `entries()` 方法。所以**没有办法获取 `WeakMap` 的所有键或值**

一个常见的例子是缓存。

```js
// 📁 cache.js  
let cache = new WeakMap();  
  
// 计算并记结果  
function process(obj) {  
	if (!cache.has(obj)) {  
		let result = /* calculate the result for */ obj;  
		cache.set(obj, result);  
	}  
  
	return cache.get(obj);  
}  
  
// 📁 main.js  
let obj = {/* some object */};  
  
let result1 = process(obj);  
let result2 = process(obj);  
  
// ……稍后，我们不再需要这个对象时：  
obj = null;  
  
// 无法获取 cache.size，因为它是一个 WeakMap，  
// 要么是 0，或即将变为 0  
// 当 obj 被垃圾回收，缓存的数据也会被清除
```


### 18.3 Set

`Set` 是一个特殊的类型集合 —— “值的集合”（没有键），它的每一个值只能出现一次。

它的主要方法如下：

- `new Set(iterable)` —— 创建一个 `set`，如果提供了一个 `iterable` 对象（通常是数组），将会从数组里面复制值到 `set` 中。
- `set.add(value)` —— 添加一个值，返回 set 本身
- `set.delete(value)` —— 删除值，如果 `value` 在这个方法调用的时候存在则返回 `true` ，否则返回 `false`。
- `set.has(value)` —— 如果 `value` 在 set 中，返回 `true`，否则返回 `false`。
- `set.clear()` —— 清空 set。
- `set.size` —— 返回元素个数。

它的主要特点是，重复使用同一个值调用 `set.add(value)` 并不会发生什么改变。这就是 `Set` 里面的每一个值只出现一次的原因。

我们可以使用 `for..of` 或 `.forEach` 来遍历 Set

注意一件有趣的事儿。`forEach` 的回调函数有三个参数：一个 `value`，然后是 **同一个值** `valueAgain`，最后是目标对象。没错，同一个值在参数里出现了两次。

`forEach` 的回调函数有三个参数，是为了与 `Map` 兼容。

`Map` 中用于迭代的方法在 `Set` 中也同样支持：

- `set.keys()` —— 遍历并返回一个包含所有值的可迭代对象，
- `set.values()` —— 与 `set.keys()` 作用相同，这是为了兼容 `Map`，
- `set.entries()` —— 遍历并返回一个包含所有的实体 `[value, value]` 的可迭代对象，它的存在也是为了兼容 `Map`。


### 18.4 WeakSet

`WeakSet` 的表现类似：

- 与 `Set` 类似，但是我们只能向 `WeakSet` 添加对象（而不能是原始值）。
- 对象只有在其它某个（些）地方能被访问的时候，才能留在 `WeakSet` 中。
- 跟 `Set` 一样，`WeakSet` 支持 `add`，`has` 和 `delete` 方法，但不支持 `size` 和 `keys()`，并且不可迭代。

### 18.5 Object. Keys, values, entries

对于普通对象，下列这些方法是可用的：

- [Object.keys(obj)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) —— 返回一个包含该对象所有的键的数组。
- [Object.values(obj)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/values) —— 返回一个包含该对象所有的值的数组。
- [Object.entries(obj)](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Object/entries) —— 返回一个包含该对象所有 [key, value] 键值对的数组。

Object.keys/values/entries 会忽略 symbol 属性 就像 `for..in` 循环一样

对数组使用 `Object.fromEntries(array)` 方法，转回成对象

## 5. Date

## 19. 日期和时间

### 19.1 创建

调用 `new Date()` 来创建一个新的 `Date` 对象。在调用时可以带有一些参数

`new Date()` 不带参数 —— 创建一个表示当前日期和时间的 `Date` 对象

`new Date(milliseconds)` 创建一个 `Date` 对象，其时间等于 1970 年 1 月 1 日 UTC+0 之后经过的毫秒数（1/1000 秒）。**时间戳**

在 01.01.1970 之前的日期带有负的时间戳

可以使用 `date.getTime()` 将现有的 `Date` 对象转化为时间戳

`new Date(datestring)` 如果只有一个参数，并且是字符串，那么它会被自动解析

字符串的格式应该为：`YYYY-MM-DDTHH:mm:ss.sssZ`，其中：

- `YYYY-MM-DD` —— 日期：年-月-日。
- 字符 `"T"` 是一个分隔符。
- `HH:mm:ss.sss` —— 时间：小时，分钟，秒，毫秒。
- 可选字符 `'Z'` 为 `+-hh:mm` 格式的时区。单个字符 `Z` 代表 UTC+0 时区。

简短形式也是可以的，比如 `YYYY-MM-DD` 或 `YYYY-MM`，甚至可以是 `YYYY`。

如果给定字符串的格式不正确，则返回 `NaN`。

`new Date(year, month, date, hours, minutes, seconds, ms)` 使用当前时区中的给定组件创建日期。只有前两个参数是必须的。

- `year` 应该是四位数。为了兼容性，也接受 2 位数，并将其视为 `19xx`，例如 `98` 与 `1998` 相同，但强烈建议始终使用 4 位数。
- `month` 计数从 `0`（一月）开始，到 `11`（十二月）结束。
- `date` 是当月的具体某一天，如果缺失，则为默认值 `1`。
- 如果 `hours/minutes/seconds/ms` 缺失，则均为默认值 `0`。

### 19.2 访问日期

从 `Date` 对象中访问年、月等信息有多种方式：

[getFullYear()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getFullYear)

获取年份（4 位数）

[getMonth()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getMonth)

获取月份，**从 0 到 11**。

[getDate()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getDate)

获取当月的具体日期，从 1 到 31，这个方法名称可能看起来有些令人疑惑。

[getHours()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getHours)，[getMinutes()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getMinutes)，[getSeconds()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getSeconds)，[getMilliseconds()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getMilliseconds)

获取相应的时间组件。

另外，我们还可以获取一周中的第几天：

[getDay()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getDay)

获取一周中的第几天，从 `0`（星期日）到 `6`（星期六）。第一天始终是星期日，在某些国家可能不是这样的习惯，但是这不能被改变。

**以上的所有方法返回的组件都是基于当地时区的。**

当然，也有与当地时区的 UTC 对应项，它们会返回基于 UTC+0 时区的日、月、年等：[getUTCFullYear()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getUTCFullYear)，[getUTCMonth()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getUTCMonth)，[getUTCDay()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getUTCDay)。只需要在 `"get"` 之后插入 `"UTC"` 即可。

除了上述给定的方法，还有两个没有 UTC 变体的特殊方法：

[getTime()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getTime)

返回日期的时间戳 —— 从 1970-1-1 00:00:00 UTC+0 开始到现在所经过的毫秒数。

有一个特殊的方法 `Date.now()`，它会返回当前的时间戳。

它相当于 `new Date().getTime()`，但它不会创建中间的 `Date` 对象。因此它更快，而且不会对垃圾回收造成额外的压力。

当 `Date` 对象被转化为数字时，得到的是对应的时间戳，即`+date`等价于`date.getTime()`

[getTimezoneOffset()](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/getTimezoneOffset)

返回 UTC 与本地时区之间的时差，以分钟为单位

### 19.3. 设置时间

下列方法可以设置日期/时间组件：

- [`setFullYear(year, [month], [date])`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setFullYear)
- [`setMonth(month, [date])`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setMonth)
- [`setDate(date)`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setDate)
- [`setHours(hour, [min], [sec], [ms])`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setHours)
- [`setMinutes(min, [sec], [ms])`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setMinutes)
- [`setSeconds(sec, [ms])`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setSeconds)
- [`setMilliseconds(ms)`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setMilliseconds)
- [`setTime(milliseconds)`](https://developer.mozilla.org/zh/docs/Web/JavaScript/Reference/Global_Objects/Date/setTime)（使用自 1970-01-01 00:00:00 UTC+0 以来的毫秒数来设置整个日期）

以上方法除了 `setTime()` 都有 UTC 变体，例如：`setUTCHours()`。

### 19.4 [自动校准（Autocorrection）](https://zh.javascript.info/date#zi-dong-jiao-zhun-autocorrection)

**自动校准** 是 `Date` 对象的一个非常方便的特性。我们可以设置超范围的数值，它会自动校准。

这个特性经常被用来获取给定时间段后的日期。例如，我们想获取“现在 70 秒后”的日期

## 6. JSON

Convert objects into strings and vice versa!

```js

let x = {1: 2, foo: {uwu: 'owo'}}
let y = JSON.stringify(x) // y is '{"1":2,"foo":{"uwu":"owo"}}'
let z = JSON.parse(y) // retrieves x

```

● Useful if you need to store info as a string (e.g. in a database) and then

● Our utilities.js get/post request code does this
retrieve it later

## 浅拷贝与深拷贝的安全写法

[上文](JavaScript-Objects-and-Data-Structures.md#12.%20对象的拷贝)里已经区分了对象引用复制、浅拷贝和深拷贝。需要特别注意：`typeof` 返回的是小写字符串，所以判断对象时应写：

```js
if (typeof value === "object" && value !== null) {
  // value 是对象或数组等引用类型
}
```

一个简化版递归深拷贝：

```js
function deepClone(src) {
  if (typeof src !== "object" || src === null) {
    return src;
  }

  const clone = Array.isArray(src) ? [] : {};
  for (const key in src) {
    clone[key] = deepClone(src[key]);
  }
  return clone;
}
```

现代环境中，如果只需要复制可结构化克隆的数据，也可以使用：

```js
const clone = structuredClone(obj);
```

但函数、DOM 节点等并不适合用 `structuredClone` 直接处理。
