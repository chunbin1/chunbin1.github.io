---
title: is关键字
date: 2024-07-05 16:18:35
tags: [TypeScript]
code_block_shrink:  false
excerpt: TypeScript中is关键字的使用
---
常用于函数的返回值，判断类型是否是某种类型
```typescript
function isString(test: any): test is string {
  return typeof test === 'string'
}

function example(foo: any) {
  if (isString(foo)) {
    console.log('it is a string' + foo)
    // 在这里 foo的类型已经变成了string,ts编译器不会报错
    console.log(foo.length) // string function
  } else {
    console.log('not string')
  }
}
example('hello world')
example(2)
```

is关键字只会在判断后的块级作用域中生效
E.g.2
```typescript
function example(foo: any) {
  if (isString(foo)) {
    console.log('it is a string' + foo)
    console.log(foo.length)
  }
  // 这里可能会产生运行错误，因为此时类型为any，不会产生编译错误
  console.log(foo.toExponential(2))
}
```
这说明，is关键字，只会在判断后的块作用域中生效，所以离开了作用域的foo在ts编译器中又变回any类型