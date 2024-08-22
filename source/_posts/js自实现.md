---
title: js自实现
code_block_shrink: false
date: 2024-08-22 12:30:45
updated: 2024-08-22 12:30:45
excerpt: js的一些api的自实现
tags: [js]
---
# js的自实现
## new的实现
### 过程
1. 新建一个对象p，把构造函数中的this设置为对象的变量
2. 把p的__proto__指向其构造器的原型对象P.prototype
3. 如果构造函数中有`返回值，且返回值为对象，则返回该对象`，否者返回新建的对象

{% asset_img 原型链图.png 原型链图 %}

### 代码实现
```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayYourName = function() {
  console.log("i am" + this.name);
};

function New(constructor, ...args) {
  let obj = {};
  if (constructor.prototype !== null) {
    obj.__proto__ = constructor.prototype;
  }
  // 执行构造函数 把属性或者方法添加到创建的空对象上
  let ret = constructor.apply(obj, args);
  return ret instanceof Object ? ret : obj
}

const p = New(Person, "sdfs", 12);
p.sayYourName();
```

## call的实现
### 思路
1. 将函数设置为对象的方法(this的指向为该对象)
2. 调用该方法
3. 删除该方法

```js
Function.prototype.call2 = function(content = window,...args) {
  // this为该方法
  content.fn = this;
  let result = content.fn(...args);
  delete content.fn;
  return result;
}

const obj = {
  name: 'lcb',
  age: 24,
}

function bar(name, age) {
  console.(this.name)
  console.log(this.age)
}

console.log(bar.call2(obj));
```

## apply
### 思路
同call
### 实现
```js
Function.prototype.apply2 = function(context = window,args) {
  context.fn = this;
  let obj;
  obj = context.fn(args);
  delete obj.fn;
  return obj;
};
```

## 函数科里化
### 思路
1. 如果传入的参数没有到原函数需要的数量 则继续执行curry函数接收参数
2. 如果参数达到了，则执行科里化了的函数

### 实现
```js
function curry(fn, ...args) {
  let length = fn.length; // Function.length 为 函数需要几个参数
  return function(...params){
      newArgs = args.concat(params);
      if (newArgs.length < length) {
          return curry.call(this,fn,...newArgs);
      }else{
          return fn.apply(this,newArgs);
      }
  }
}

function multiFn(a, b, c) {
  return a * b * c;
}

let multi = curry(multiFn);
console.log(multi(2)(3)(4));
console.log(multi(2,3,4));
console.log(multi(2)(3,4));
console.log(multi(2,3)(4));
```

## JSON.stringfy
### 思路
转化类型:  
1. Boolean | Number | String 类型会转换为对应的原始类型  
2. undefined、任意函数以及symbol，会被忽略（出现在非数组对象的属性值中时），或者被转换成 null（出现在数组中时）  
3. 不可枚举的属性会被忽略  
4. 如果一个对象的属性值通过某种间接的方式指回该对象本身，即循环引用，属性也会被忽略。// todo  
5. 递归/循环

### 实现
```js
function jsonStringify(obj) {
  let type = typeof obj;
  // 判断是否为对象
  if (type === "object") {
    let json = [];
    let arr = Array.isArray(obj); // 判断数组
    for (let k in obj) {
      let v = obj[k];
      let type = typeof v;
      if (/string|undefined|function/.test(type)) {
        v = '"' + v + '"';
      } else if (type === "object") {
        v = jsonStringify(v);
      }
      json.push((arr ? "" : '"' + k + '":') + String(v));
    }
  } else {
    if (/string|undefined|function/.test(type)) {
      obj = '"' + obj + '"';
    }
    return String(obj);
  }
  return (arr ? "[" : "{") + String(json) + (arr ? "]" : "}");
}

```
