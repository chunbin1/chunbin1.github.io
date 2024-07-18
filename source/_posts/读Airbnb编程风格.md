---
title: 读Airbnb编程风格
date: 2024-07-10 18:10:42
tags: [js]
---
读了Airbnb的编程风格，记录一些有意思的书写习惯

- 3.2 使用计算出来的属性创建对象
```js
function getKey(k){
  return `a key named ${k}`
}
// good 
const obj={
  id:5,
  name : 'San Francisco',
  [getKey('enabled')] : true,
}
```

- 4.5 使用Array.from把类数组对象转变为数组
```js
const arrLike = { 0: 'foo', 1: 'bar', 2: 'baz', length: 3 };

// bad
const arr = Array.prototype.slice.call(arrLike);

// good
const arr = Array.from(arrLike);
```

- 7.1 使用赋值函数表达式而不是函数声明方式
why: 
1. 函数声明是会提升到全文的，很容易在未声明之前就使用了，降低程序的可维护性和可读性。
2. 可以用短的名字来简写长的函数 

```js
const short = function longUniqueMoreDescriptiveLexicalFoo(){

}
```

- 7.4 在非函数的{}中声明函数的时候，如(if,while,etc)中，使用箭头表达式
```js
if(currentUser){
  test = () =>{
    console.log('Yup.')
  }
}
```

- 7.6 不用使用arguments，使用...代替
why: ...args是一个真正的数组，而arguments是一个类数组对象
```js
function concatenateAll(...args){
  return args.join('')
}
```

- 7.7 使用默认参数语法，而不是改变函数参数
```js
// really bad
function handleThings(opts) {
  // No! We shouldn’t mutate function arguments.
  // Double bad: if opts is falsy it'll be set to an object which may
  // be what you want but it can introduce subtle bugs.
  opts = opts || {};
  // ...
}

// good
function handleThings(opts = {}) {
  // ...
}
```

- 8.5 不要把箭头表达式(=>)和比较运算符一起使用(<=,>=)
```js
// bad
const itemHeight = item => item.height <= 256 ? item.largeSize : item.smallSize;

// bad
const itemHeight = (item) => item.height >= 256 ? item.largeSize : item.smallSize;

// good
const itemHeight = item => (item.height <= 256 ? item.largeSize : item.smallSize);

// good
const itemHeight = (item) => {
  const { height, largeSize, smallSize } = item;
  return height <= 256 ? largeSize : smallSize;
};
```

- 9.1 使用class，避免直接操控原型
why: class语法更简洁
```js
// bad
function Queue(contents = []) {
  this.queue = [...contents];
}
Queue.prototype.pop = function () {
  const value = this.queue[0];
  this.queue.splice(0, 1);
  return value;
};

// good
class Queue {
  constructor(contents = []) {
    this.queue = [...contents];
  }
  pop() {
    const value = this.queue[0];
    this.queue.splice(0, 1);
    return value;
  }
}
```

- 9.3 方法中可以return this 让函数可以链式的调用
```js
// good
class Jedi {
  jump() {
    this.jumping = true;
    return this;
  }

  setHeight(height) {
    this.height = height;
    return this;
  }
}

const luke = new Jedi();

luke.jump()
  .setHeight(20);
```

- 9.5 class中有默认的构造函数
```js
class Rey extends Jedi{
  constructor(...args){
    super(...args);
    this.name = 'Rey'  //如果没有这个 不要指定构造函数
  }
}
```

- 11.1 不要使用迭代器。最好使用JavaScript的高阶函数，而不是像for-in或for-of
why:这遵循了不变的规则，纯函数的返回值比副作用更容易推断
```js
  const numbers = [1, 2, 3, 4, 5];

// bad
let sum = 0;
for (let num of numbers) {
  sum += num;
}
sum === 15;

// good
let sum = 0;
numbers.forEach((num) => {
  sum += num;
});
sum === 15;

// best (use the functional force)
const sum = numbers.reduce((total, num) => total + num, 0);
sum === 15;

// bad
const increasedByOne = [];
for (let i = 0; i < numbers.length; i++) {
  increasedByOne.push(numbers[i] + 1);
}

// good
const increasedByOne = [];
numbers.forEach((num) => {
  increasedByOne.push(num + 1);
});

// best (keeping it functional)
const increasedByOne = numbers.map(num => num + 1);
```

- 13.6 使用num+=1 代替 num++
  使用num-=1 代替 num--

- 13.7 有超长的函数或者函数名时，用括号括起来
```js
// bad
const foo =
  superLongLongLongLongLongLongLongLongFunctionName();

// bad
const foo
  = 'superLongLongLongLongLongLongLongLongString';

// good
const foo = (
  superLongLongLongLongLongLongLongLongFunctionName()
);

// good
const foo = 'superLongLongLongLongLongLongLongLongString';
```

- 15.5 使用{}在包含变量声明的语句中，例如:(let,const,function, class)
```js
switch(foo){
  case 1: {
    let x = 1;
    break;
  }
  case 2: {
    const y = 2;
    break;
  }
  case 3: {
    function f(){
      // ...
    }
    break;
  }
  case 4: 
    bar();
    break;
  default:{
    class C {}
  }
}
```

- 15.7 不要使用不需要的三元表达式
```js
// bad
const foo = a ? a : b;
const bar = c ? true : false;
const baz = c ? false : true;

// good
const foo = a || b;
const bar = !!c;
const baz = !c;
```
