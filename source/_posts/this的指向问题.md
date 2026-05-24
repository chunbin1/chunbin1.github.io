---
title: this的指向
date: 2020-05-13 09:50:56
updated: 2020-05-13 09:50:56
tags: 前端
code_block_shrink: false
excerpt: 目的：指向函数运行时所在的环境
---
## 为何要设计this关键字
目的：指向函数运行时所在的环境
```js
var obj = { getName:function(){return this.name} }
```
解释：
 1. 当对象的属性是一个`函数`的时候，会把函数单独保存在`堆`中,然后把函数的`地址`赋值给对象属性。
 2. 而js是允许`在函数体内引用当前环境的其他变量`，所以当函数在不同的运行环境执行的时候，需要`this来获取函数内的当前运行环境`。
## 如何判断this所指的环境变量
### 默认绑定
#### 非严格模式
非严格模式下this指向全局对象
```js
var name = 'lcb';
function foo() {
  console.log(this.name);
}

foo(); // lcb
```
#### 严格模式
严格模式下，不能将全局对象`window`作为默认绑定,`this`会绑定到undefined
```js
'use strict';
function foo(){
  console.log(this.name)
}
var name = 'lcb'
foo()
// Uncaught TypeError: Cannot read property 'name' of undefined at foo
```
严格模式调用不会影响绑定
```js
function foo(){
  console.log(this.name)
}
var name = 'lcb'
'use strict'
foo()  
// lcb
```

#### node环境下
node环境下不会把变量挂载到`全局对象`上。因为每个js文件都有自己的作用域
```js
var name = 'lcb';
function foo() {
  console.log(this); // Global
  console.log(this.name);  // undefined
}

foo();
```

### 隐式绑定
函数作为`对象的属性`,通过对象属性执行函数时候，此时`隐式绑定规则会将this绑定到对象`上:
```js
var name = 'lcb';
function foo() {
  console.log(this.name);
}

var obj = {
  name: 'lql',
  foo,
  user:{
    name:'userName',
    foo
  }
}

foo(); // lcb

var foo2 = obj.foo; 

foo2(); // lcb


obj.foo(); // lql
obj.user.foo() // userName
```
结论：只有当作为对象属性调用的时候才会指向该对象

### 显式绑定
通过`call apply bind`显式绑定
```js
var name = 'lcb';
function foo() {
  console.log(this.name);
}

const obj = {
  name:'objName'
}

const obj2 = {
  name:'objName2'
}

foo.call(obj) // objName
foo.call(obj2) // objName2
foo.call()  // 浏览器环境为lcb node为undefined
```

### new绑定
分四种情况:
- new了一个对象
- 返回了一个对象
- 返回了基本数据类型
#### new了一个对象
```js
var name = 'global'

// new了一个对象
function Foo(name){
  this.name = name
  this.getName = function(){
    console.log(this.name)
  }
}

Foo.prototype.getName2 = function(){
  console.log(this.name)
}

const newFoo = new Foo('new lcb')
newFoo.getName() // new lcb
newFoo.getName2() // new lcb
```
#### 返回了一个对象
```js
// 返回了一个对象
function Foo1(name){
  this.name = name 
  return {
    name:`${name}aaa`,
    getName:function(){
      return this.name
    }
  }
}

const a = new Foo1('lcb')
a.getName() // lcbaaa  this指向返回的对象
```

#### 返回了基本数据类型
与返回对象的不同点：new过程中，如果construtor返回的是对象，则返回该对象，否则`返回new出来的对象`，所以返回基本数据类型可以拿到name
```js
function Foo(name) {
  this.name = name;
  return 'hi'
}

const a = new Foo('lcb')
console.log(a.name) // 'lcb' 
```

### 箭头函数
一句话：箭头函数this和外层的this`（全局/函数作用域）`作用域一致。
例1:
```js
const a = {
  name:'a',
  getName:()=>{
    console.log(this.name)
  }
}

a.getName() // undefined

const b = {
  name:"b"
}

a.getName.call(b) // undefined
```
this被绑在了全局作用域下

```js
function foo() {
  setTimeout(() => {
    console.log(this.name);
  }, 0);
}

function foo2(){
  setTimeout(function(){
    console.log(this.name)
  },0)
}

var name = 'global';

const obj = {
  name:'lcb'
}

foo.call(obj); // lcb
foo2.call(obj) // 浏览器为global node为undefined
```
分析：箭头函数this和`外层的函数作用域中`的this保持一致,其中`foo中的this指向了obj对象`，所以箭头函数中的this指向obj对象。
把foo转化为ES5
```js
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log(_this.name);
  }, 0);
}
```



