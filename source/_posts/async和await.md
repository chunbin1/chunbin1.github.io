---
title: async/await
date: 2020-04-28 09:57:03
updated: 2020-04-28 09:57:03
tags: 前端
code_block_shrink: false
excerpt: function* genDemo() {
---
## 生成器（Generator）
```js
function* genDemo() {
    console.log("开始执行第一段")
    yield 'generator 2'

    console.log("开始执行第二段")
    yield 'generator 2'

    console.log("开始执行第三段")
    yield 'generator 2'

    console.log("执行结束")
    return 'generator 2'
}

console.log('main 0')
let gen = genDemo()
console.log(gen.next().value)
console.log('main 1')
console.log(gen.next().value)
console.log('main 2')
console.log(gen.next().value)
console.log('main 3')
console.log(gen.next().value)
console.log('main 4')
```
为啥能暂停和恢复？
### 协程
定义：协程是一种比线程更加轻量级的存在
特点：
- 一个线程可以存在多个协程，但是线程上同时只能执行一个协程
- 协程不会被操作系统内核管理，完全由程序所控制（也就是在用户态执行）
### 如何运行
1. 调用生成器函数getDemo来创建一个协程gen,创建后并没有立刻执行
2. 调用gen.next(),执行携程
3. 协程执行，通过yield关键字暂停gen协程的执行，并返回主要信息给父协程。
4. 如果协程在执行期间，遇到return 关键字，那么js引擎会结束当前协程，并将return后面的内容返回给父协程
### 如何恢复
1. gen协程和父协程是在主线程上交互执行的
2. 在gen协程中调用yield方法时，js引擎会保存gen协程当前的调用栈信息，并恢复父协程的调用栈信息。
3. 父协程中执行gen.next()时，js引擎会保存父协程的调用栈信息，并恢复gen协程的调用栈信息。

<img src="./image/协程切换.png" />

## async/await
底层技术：Promise和生成器即微任务和协程

### async
定义：通过`异步执行`并`隐式返回Promise`作为结果的函数

### await
```js
async function foo() {
  console.log(1)
  let a = await 100 //
  console.log(a)
  console.log(2)
}
console.log(0)
foo()
console.log(3)
```
输出顺序：0=>1=>3=>100=>2
过程如下
<img src="./image/async和await之间的配合.png" />

### 一道题目
```js
async function foo() {
    console.log('foo')
}
async function bar() {
    console.log('bar start')
    await foo()
    console.log('bar end')
}
console.log('script start')
setTimeout(function () {
    console.log('setTimeout')
}, 0)
bar();
new Promise(function (resolve) {
    console.log('promise executor')
    resolve();
}).then(function () {
    console.log('promise then')
})
console.log('script end')
```
结果：
script start => bar start => foo => promise executor => script end => bar end => promise then => setTimeout
分析：

1. 主进程运行，主进程到seTimeout时候，会把任务放入宏任务对队列  
输出  script start 

2. 执行bar()时候,await相当于返回一个promise给父协程,相当于在父协程`new Promise((resolve,reject)=>{console.log('foo'); resole()})`，所以会先打印foo,然后，bar中剩下的内容放入微任务队列  
输出 bar start => foo
3. 主进程运行了Promise，把then中的内容加入了微任务队列，然后主进程执行完毕  
输出 promise then => script end
3. 执行微任务队列，按顺序来
输出 bar end => promise then   
4. 执行宏任务队列  
输出 setTimeout

