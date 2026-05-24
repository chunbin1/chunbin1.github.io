---
title: LazyMan
date: 2020-05-29 08:35:08
updated: 2020-05-29 08:35:08
tags: 前端
code_block_shrink: false
excerpt: 最近做了一道LazyMan题目,觉得十分有趣
---
最近做了一道LazyMan题目,觉得十分有趣
## 描述
```js
LazyMan.sleep(5).eat('somthing') // 先睡5s再打印 eat somthing

LazyMan.eat('something').sleepFirst(4) // 调用sleepFirst再调用eat
```
## 思路
该题目的调用有几个特点，一是链式调用，二是异步调用，三是调用优先级可以变。  
我采用类似中间件模式，新建了LazyMan对象后，先要注册方法，注册方法会返回调用下个方法的函数。
这样解耦了各种处理函数和LazyMan，添加新的LazyMan方法不需要更改LazyMan类

## 实现
```js
class LazyMan {
  constructor() {
    this.taskQueue = [];
    new Promise((resole) => {
      resole();
    }).then(() => {
      this.trigger();
    });
    this.next = this.next.bind(this);
  }
  // order 1 表示在后面插入 order -1 表示插入在前面
  use(key, fnc, order = 1) {
    const _this = this;
    if (!this[key]) {
      this[key] = function (...args) {
        const fn = function () {
          fnc(_this.next, ...args);
        };
        if (order === 1) {
          this.taskQueue.push(fn);
        } else if (order === -1) {
          this.taskQueue.unshift(fn);
        }
        return this;
      };
    }
  }

  next() {
    if (this.taskQueue.length !== 0) {
      const fn = this.taskQueue.shift();
      fn();
    }
  }
  trigger() {
    this.next();
  }
}

const lazyMan = new LazyMan();

lazyMan.use("eat", (next, something) => {
  console.log(`eat ${something}`);
  next();
});

lazyMan.use("sleep", (next, time) => {
  const s = time * 1000;
  setTimeout(() => {
    console.log(`sleep ${time}s`);
    next();
  }, s);
});

lazyMan.use(
  "sleepFirst",
  (next, time) => {
    const s = time * 1000;
    setTimeout(() => {
      console.log(`sleepFirst ${time}s`);
      next();
    }, s);
  },
  -1
);

lazyMan.sleep(5).eat("chicken").sleepFirst(2);
```