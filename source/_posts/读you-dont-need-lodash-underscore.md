---
title: 读you-dont-need-lodash-underscore
date: 2024-07-09 17:03:36
tags: [javascript]
---
# 读you-dont-need-lodash-underscore
[原文](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore)

记录一些可能会用到或者是从来没见过的

## Array
### reduce
累计器
#### callback
执行数组中每个值的函数，包含四个参数：
- accumulator
累计器累计回调的返回值; 它是上一次调用回调时返回的累积值，或initialValue（见于下方）。
- currentValue
数组中正在处理的元素。
- currentIndex可选
数组中正在处理的当前元素的索引。 如果提供了initialValue，则起始索引号为0，否则为1。
- array可选
调用reduce()的数组
#### initialValue可选
作为第一次调用 callback函数时的第一个参数的值。 如果没有提供初始值，则将使用数组中的第一个元素。 在没有初始值的空数组上调用 reduce 将报错。

### concat合并数组
```javascript
// Native
var array = [1]
var other = array.concat(2, [3], [[4]])

// output: [1, 2, 3, [4]]
```

### a不在b存在的数组 _.difference
```javascript
const a = [1, 2, 3, 4, 5]
const b = [5, 2, 10]

a.filter(c=>!b.includes(c))
// output: [1, 3, 4]
```

### .fill
```javascript
const a = [1,2,3]
array.fill('a')
// output: ['a', 'a', 'a']

Array(3).fill(2)
// output: [2, 2, 2]

[4, 6, 8, 10].fill('*', 1, 3)
// output: [4, '*', '*', 10]
```
### .find与.findIndex
```javascript
// Native
const users = [
  { 'user': 'barney',  'age': 36, 'active': true },
  { 'user': 'fred',    'age': 40, 'active': false },
  { 'user': 'pebbles', 'age': 1,  'active': true }
]

users.find(function (o) { return o.age < 40; })
// output: object for 'barney'

var index = users.findIndex(function (o) { return o.age >= 40; })
console.log(index)
// output: 1
```

### 简单扁平化数组
```javascript
// Native
const flatten = [1, [2, [3, [4]], 5]].reduce( (a, b) => a.concat(b), [])
// => [1, 2, [3, [4]], 5]

// Native(ES2019)  兼容性暂时不好
const flatten = [1, [2, [3, [4]], 5]].flat()
// => [1, 2, [3, [4]], 5]
```
### 深沉次扁平化数组
```javascript
// Native
const flattenDeep = (arr) => Array.isArray(arr)
  ? arr.reduce( (a, b) => a.concat(flattenDeep(b)) , [])
  : [arr]

flattenDeep([1, [[2], [3, [4]], 5]])
// => [1, 2, 3, 4, 5]

// Native(ES2019)
[1, [2, [3, [4]], 5]].flat(Infinity)
// => [1, 2, 3, 4, 5]
```

### .indexOf
```javascript
// Native
var array = [2, 9, 9]
var result = array.indexOf(2)
console.log(result)
// output: 0
```

### 寻找交集
```javascript
// ES6
let arrays = [[1, 2, 3], [101, 2,3, 1, 10], [2, 1,3]];
console.log(arrays.reduce((a, b) => a.filter(c => b.includes(c))));
// output: [1, 2]
```

### 从右切割数组
```javascript
// Native
[1, 2, 3].slice(-1);
// => [3]

[1, 2, 3].slice(-2);
// => [2, 3]

[1, 2, 3].slice(-5);
// => [1, 2, 3]
```

### .isArray,isArrayBuffer
```javascript
// Native
var array = []
console.log(Array.isArray(array));
// output: true

// Native
console.log(new ArrayBuffer(2) instanceof ArrayBuffer);
// output: true
```

### .lastIndexOf
```javascript
// Native
var array = [2, 9, 9, 4, 3, 6]
var result = array.lastIndexOf(9)
console.log(result)
// output: 2
```

### .every
```javascript
// Native
function isLargerThanTen (element, index, array) {
  return element >= 10
}

var array = [10, 20, 30]
var result = array.every(isLargerThanTen)
console.log(result)
// output: true
```

### 对数组分类
```javascript
// 对长度进行分类 话说这个范例的可读性真的没问题吗...
var grouped = ['one', 'two', 'three'].reduce((r, v, i, a, k = v.length) => ((r[k] || (r[k] = [])).push(v), r), {})
```

### 把数组转换为对象
需要保证作为key的唯一性
```javascript
// keyBy for array only
const keyBy = (array, key) => (array || []).reduce((r, x) => ({ ...r, [key ? x[key] : x]: x }), {});

// Native
console.log(keyBy(['a', 'b', 'c']))
// output: { a: 'a', b: 'b', c: 'c' }
console.log(keyBy([{ id: 'a1', title: 'abc' }, { id: 'b2', title: 'def' }], 'id')
// output: { a1: { id: 'a1', title: 'abc' }, b2: { id: 'b2', title: 'def' } }
console.log(keyBy(Object.values({ data: { id: 'a1', title: 'abc' }}), 'id')
// output: { a1: { id: 'a1', title: 'abc' }}

// keyBy for array and object
const collectionKeyBy = (collection, key) => { 
  const c = collection || {};
  return c.isArray() ? keyBy(c, key) : Object.values(keyBy(c, key));
}
```

### 初始化数组 .from
这个操作是真的骚
```javascript
// Native ( solution with Array.from )
Array.from({length: 4}, (_, i) => i)  // output: [0, 1, 2, 3]
Array.from({length: 4}, (_, i) => -i) // output: [-0, -1, -2, -3]
Array.from({length: 4}, (_, i) => i + 1) // output: [1, 2, 3, 4]
Array.from({length: 4}, (_, i) => i * 5) // output: [0, 5, 10, 15]

// Native ( solution with keys() and spread )
[...Array(4).keys()]  // output: [0, 1, 2, 3]
[...Array(4).keys()].map(k => -k) // output: [-0, -1, -2, -3]
[...Array(4).keys()].map(k => k + 1)  // output: [1, 2, 3, 4]
[...Array(4).keys()].map(k => k * 5)  // output: [0, 5, 10, 15]
```

### .reduceRight从右开始遍历 用法与.reduce一致

### .some 与.every类似 any的意思

## function

## Lang
### isEmpty
```javascript
// Native
const isEmpty = obj => [Object, Array].includes((obj || {}).constructor) && !Object.entries((obj || {})).length;

console.log(isEmpty(null)
// output: true
console.log(isEmpty('')
// output: true
console.log(isEmpty({})
// output: true
console.log(isEmpty([])
// output: true
console.log(isEmpty({a: '1'})
// output: false
```

### Number.isFinite()
```javascript
// Native
console.log(Number.isFinite('3'))
// output: false
console.log(Number.isFinite(3))
// output: true
```

### Number.isNaN()
```javascript
// Native
console.log(isNaN(NaN))
// output: true

// ES6
console.log(Number.isNaN(NaN))
// output: true
```
使用Number.isNaN()不会强制将参数转换为数字
二者的区别
```javascript
Number.isNaN('asd') // false
isNaN('asd')  // true 会转换为数字再比较
```

## Object
.assign

### 在兑现中选中key为提供列表的元素
```javascript
// Native
  return keys.reduce((obj, key) => {
function pick(object, keys) {
     if (object[key]) {
        obj[key] = object[key];
     }
     return obj;
   }, {});
}
var result = pick(object, ['a', 'c']);
console.log(result)
// output: {a: 1, c: 3}
```

### Object.entries()
Object.entries()方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in 循环遍历该对象时返回的顺序一致（区别在于 for-in 循环也枚举原型链中的属性）。

```javascript
const obj = { foo: 'bar', baz: 42 };
console.log(Object.entries(obj)); // [ ['foo', 'bar'], ['baz', 42] ]
```

### Object.values()
```javascript
// Native
var result2 = Object.values({one: 1, two: 2, three: 3})
console.log(result2)
// output: [1, 2, 3]
```

## String
### str.startsWith(searchString[, position])
searchString
要搜索的子字符串。
position 可选
在 str 中搜索 searchString 的开始位置，默认值为 0，也就是真正的字符串开头处。
```javascript
// Native
var result = 'abc'.startsWith('b', 1)
console.log(result)
// output: true
```

### repeat
```javascript
// Native
var result = 'abc'.repeat(2)
console.log(result)
// output: 'abcabc'
```
