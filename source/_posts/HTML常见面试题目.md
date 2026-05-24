---
title: HTML常见面试题目
date: 2021-07-19 21:52:25
updated: 2021-07-19 21:52:25
tags: 前端
code_block_shrink: false
excerpt: width只包含中间的content
---
## 盒子模型
### 标准盒子模型
width只包含中间的content
### IE盒子模型
```css
box-sizing:border-box;
```
width包含了content+padding+border

## 浏览器解析dom的流程
## 两栏布局
## 控制显示、隐藏
```css
display:none;
visibility: hidden
```
区别visibility元素不会消失,会占住位置
## 行内元素和块级元素
## css中的content元素
可以和伪元素:after、:before一起使用，给某些元素前后加上文字
## 冒泡、捕获
冒泡是指从内到外。
捕获是从外到内。
## 事件循环
```js
setTimeout(function () {
  console.log("1");
}, 0);
async function async1() {
  console.log("2");
  const data = await async2();
  console.log("3");
  return data;
}
async function async2() {
  return new Promise((resolve) => {
    console.log("4");
    resolve("async2的结果");
  }).then((data) => {
    console.log("5");
    return data;
  });
}
async1().then((data) => {
  console.log("6");
  console.log(data);
});
new Promise(function (resolve) {
  console.log("7");
  //   resolve()
}).then(function () {
  console.log("8");
});
```
输出顺序：
主进程执行输出：
2 -> 4 -> 7 -> 5 -> 3 -> 6 -> async2的结果 -> 1