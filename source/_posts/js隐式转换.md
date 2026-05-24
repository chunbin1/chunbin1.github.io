---
title: js隐式类型转换
date: 2021-08-23 21:19:24
updated: 2021-08-23 21:19:24
tags: 前端
code_block_shrink: false
excerpt: 最近做了一些题目[js转换](https://www.eveningwater.com/my-web-projects/react/5/)，做完一头雾水，应该有
---
最近做了一些题目[js转换](https://www.eveningwater.com/my-web-projects/react/5/)，做完一头雾水，应该有一些规律可以让我全作对这些这些问题、

## 什么是隐式类型转换
隐式转换是相对于`显式类型转换`的,通常是由`+`、`-`或者`==`等操作运算符导致的类型转换，他不明显，但是有会副作用。
比如：
```js
let a = 42 + '' // a经过转换，变成了'42'字符串类型
```
经过加法运算，a变成了字符串类型。

## 抽象值操作
对于不同的类型，抽象操作的结果其实是不一致的
### ToPrimitive
把不同类型转化为原始类型，
原始类型直接返回
对象的转换规则：
    1. 转换为相应的基本类型值
        - 检查是否有valueOf()方法，如果有并且返回基本类型值，就用该值进行强制类型转换
        - 如果没有valueOf()，检查是否有toString()方法并且返回基本类型值，使用toString()返回值来进行强制类型转换
        - 如果均不返回基本类型值，会产生TypeError错误
    2. 返回非数字的基本类型值，再遵循以上规则将其强制转换为数字
可以通过修改@@toPrimitive来修改规则，转换规则。
```js
const a = []
// 数组默认是没有@@toPrimitive的
a[Symbol.toPrimitive] = function(){
    return 3
}

// 而日期类型是有的
const b = new Date()
b[Symbol.toPrimitive]  // fn
```
### ToString
其中null=>'null',undefined=>'undefined',true=>'true'
```js
// number
const a = 42
a.toString() // '42'
const b = 42*1000**7
b.toString() // '4.2e+22'

// obj
const obj = {}
obj.toString() // "[object Object]"

// array
const arr = [1,2]
arr.toString() // '1,2'
```

### ToNumber
字符串转换规则：基本遵循数字常量的相关规则和语法，处理失败返回NaN，ToNumber对以0开头的八进制数并不按照八进制数处理，而是采用十进制。
```js
Number(010) // 8
Number('010') // 10
```
对象的转化规则：调用ToPrimitive规则
```js
Number(['LCB']); // NaN
```
过程：
 1. ['lcb']的valueOf方法返回的是['lcb'],不是基本类型，
 2. 调用toString，返回了'lcb'字符串，是基本类型
 3. 根据字符串转换规则，得出`NaN`



## 字符串与数字的隐式强制类型转换
### 一元形式的类型转换
```js
1+ +"13" // 14
1- -"13" // 14

+new Date() // 1629711851817
```
其中+X、-X是+运算符的一元形式，会调用ToNumber操作，而且-操作还会改变数字的符号位。
ps：值得注意的是，这种一元形式转换大部分被归为显式类型转换，但是称之为隐式也没问题（不是我说的，是《你不知道的js》说的，概念问题不纠结）。

### +运算符
+运算符能用于数字加法和字符串拼接
`+运算符的转换规则：如果是对象则调用ToPrimitive规则，如果其中一个经过ToPrimitive被转换为了字符串类型或者就是字符串类型，那么就执行拼接操作。否则调用ToNumber操作进行数字相加`
```js
const a = [1,2]
const b = [3,4]
a+b // 1,23,4
```
过程分析：
    1. a和b都为数组，先调用ToPrimitive规则
    2. a可以通过toString转换为`'1,2'`, b通过toString转换为`'3,4'`，满足拼接的操作
    4. 拼接得到`1,23,4`
举一反三验证我们的想法
```js
[1,2]+4 // 1,24

// 修改valueOf
const a = [3,4]
const b = [1,2]
a.valueOf = ()=>{
    console.log('valueOf被调用了')
    return 4
}
a + b // 41,2
```
a,b经过ToPrimitive后，b为字符串，进行拼接操作
```js
const a = [3,4]
a[Symbol.toPrimitive] = function(){
    return 3
}
a + 4 // 7
```
修改Symbol.toPrimitive可以改变`ToPrimitive`行为。

```js
undefined + false // Number(undefined) + Number(false) =  NaN + 0
```

#### 坑
```js
{} + [] // 0
[] + {} // '[object Object]'
```
解释:
第一个`{}+[]`其中，`{}`被js引擎解释为代码块，只是里面没有代码。所以`{}+[]`等于0。
而第二个执行ToPrimitive操作，得到'[object Object]'。

## 宽松相等
虽然从来不用=。=
### 字符串和数字之间的相等比较
ES5规范规定，如果其中之一是数字x，另一是字符串y，返回x == ToNumber(y)
```js
var a = 42
var b = '42'

a == b
```

### 其他类型和布尔类型的相等比较
根据规范，如果其中之一是布尔类型x,另一是任意类型y，返回ToNumber(x) == y的结果
```js
var x = true
var y = '42'

x == y // false
```
过程：
1. x为true，转换为1，求 1 == '42'的结果
2. 1为Number类型，‘42’为字符串类型，依照字符串和数字之间的相等比较
3. ‘42’被转换为42
4. 1==42得出false

### null和undefined的相等比较
根据规范，其中之一的x为null，另一y为undefined，则结果为true，在==中，null和undefined相等且与自身相等，除此之外其他值不和它们两相等。

```js
var a = undefined
var b = null

a == b // true
a == undefined // true

a == '12' // false
a == false // false
```

### 对象与非对象之间的相等比较
规矩ES5规范，其中之一x为字符串或数字，另一y为对象，则返回 x == ToPrimitive(y)的结果

···
var a = 42
var b = [42]

a == b // true
···
过程分析：
1. b调用ToPrimitive操作，返回'42'
2. 得 42 == ‘42’,根据对象和字符串的==规则,调用ToNumber操作
3. 得 42 == 42, 返回true

### 困难例子的过程分析
1. “0” == false
 - 因为有boolean类型，先调用 ToNumber(false)
 - 得到 "0" == "0", 返回true

2. false == 0
 - 因为有boolean类型，先调用 ToNumber(false)
 - 得到 "0" == 0, 调用字符串和数字类型的==规则
 - 得到 0 == 0， 返回true

3. 0 == []
 - 因为有对象类型，调用 ToPrimitive
 - 得到 0 == "", 调用字符串和数字类型的==规则
 - 得到 0 == 0，返回true

4. [] == ![]
 - !运算符强制类型转换，得 [] == true
 - 因为有对象类型，调用 ToPrimitive, 得 "" == false
 - 因为有boolean，根据布尔类型和其他类型的转换，得到 "" == 0
 - 根据字符串类型和数字类型的转换规则，得到 0 == 0 , 返回true

## 抽象关系比较
 首先调用ToPrimitive，如果出现非字符串，根据ToNumber规则把双方转换为数字类型进行比较。
 ```js
 var a = [42]
 var b = ['43']

 a<b // true
 ```
 如果双方都是字符串，则按照字母顺序返回布尔值
 ```js
 var a = ['42']
 var b = ['043']

 a < b // false
 ```
 过程：
    1. ToPrimitive转换为字符串‘42’、‘043’。
    2. 因为'4'在字母顺序上小于'0',所以返回false
应该是按照ASCII码做比较, 所以 字母小写 > 字母大写 > 数字