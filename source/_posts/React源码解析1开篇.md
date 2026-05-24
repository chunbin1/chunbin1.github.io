---
title: React源码解析 -- 开篇
date: 2021-04-25 14:30:39
updated: 2021-04-25 14:30:39
tags: 前端
code_block_shrink: false
excerpt: 版本：17.0.1
---
版本：17.0.1

## 前置知识

### 虚拟DOM
使用js来表示dom树
以react为例子，比如
```html
<div id="app">
  <p class="text">hello world!!!</p>
</div>
---

```
转化为
```js
const virtualDOM = {
  tag: 'div',
  props: {
    id: 'app'
  },
  chidren: [
    {
      tag: 'p',
      props: {
        className: 'text'
      },
      chidren: [
        'hello world!!!'
      ]
    }
  ]
}
```
这样当虚拟DOM改变后再改变真实DOM，可以减少真实DOM的操作（注意：不一定快）


### React.createElement、jsx和babel
React是通过React.createElement产生虚拟DOM的
```js
<div id="app">
  <p class="text">hello world!!!</p>
</div>
// 通过react表示等于
React.createElement("div", {
  id: "app"
}, React.createElement("p", {
  className: "text"
}, "hello world!!!"));
```
但是这样的写法写程序非常不友好，所以有了jsx
```js
function Test(props){
  return <div>
    你好
  </div>
}
```
然后通过babel插件`@babel/plugin-transform-react-jsx`转换为`React.createElement`的型式

[可以这里尝试下](https://babel.docschina.org/repl/#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&spec=false&loose=false&code_lz=DwEwlgbgBGILwCICGAHFCB8AoKVgqgGMAbJAZzIDkkBbAU0QBc6APRzACzuOIHsoA7rwBOxEAEJJwAPQpsM8BAxA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2%2Cenv&prettier=false&targets=&version=7.12.9&externalPlugins=%40babel%2Fplugin-transform-react-jsx%407.12.7)



## 拓展
[babel原理](https://juejin.cn/post/6844903760603398151#heading-5)