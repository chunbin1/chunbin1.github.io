---
title: js错题本
date: 2020-06-28 18:48:41
updated: 2020-06-28 18:48:41
tags: 前端
code_block_shrink: false
excerpt: const m = [1, 2]
---

## js中的加法
```js
const m = [1, 2]
const b = (n, ...m) => {
  return n + m
}
const a = b(1)

console.log(a) // '1'
```
### 解析
这道题有几个知识点
1. b中获取的是形参，不会获取`m`的值
2. `...m`,如果传参为空，则为`[]`
3. `1+[]`,`[].toString()`，得到`'1'`

### 拓展
#### js的加法转化
针对第3个知识点,js中加法运算规则，只做`数字和字符串的加法操作`,`所有不是这两种类型的都会被转化这两种原始数据类型再进行操作`。
[具体](https://juejin.im/post/5a9bd5cf51882555784d675b#heading-1)
`关键词`:`ToPrimitive`、`toString`、`toValue`
```js
// 各种转化情况
// 对象
const a1 = {
  a: 1
};
console.log(a1.valueOf()); // {a:1}
console.log(a1.toString()); // '[object Object]'
// 数组
const a2 = [1,2,3];
console.log(a2.valueOf()); // [1,2,3]
console.log(a2.toString()); // "1,2,3"
// 方法
const a3 = function() {
  const a = 1;
  return 1;
};
console.log(a3.valueOf()); // f(){const a = 1 return 1} 函数，可以执行
console.log(a3.toString()); // 'function(){const a = 1 return 1}'
```

