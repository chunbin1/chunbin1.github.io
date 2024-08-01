---
title: dom树
code_block_shrink: false
date: 2024-08-01 10:42:56
updated: 2024-08-01 10:42:56
excerpt: dom树
tags: [浏览器]
---
## 什么是DOM
从网络传给渲染引擎的HTML文件字节流是无法直接被渲染引擎渲染的，需要把它转化为渲染引擎能够理解的内部结构,这个结构就是DOM。  
提供了HTML文档`结构化`的表述。

- 从页面的视角来看，DOM 是生成页面的基础数据结构。  
- 从 JavaScript 脚本视角来看，DOM 提供给 JavaScript 脚本操作的接口，通过这套接口，JavaScript 可以对 DOM 结构进行访问，从而改变文档的结构、样式和内容。
- 从安全视角来看，DOM 是一道安全防护线，一些不安全的内容在 DOM 解析阶段就被拒之门外了。

DOM是`表述HTML的内部数据结构`，它会将 Web 页面和 JavaScript 脚本连接起来，并过滤一些不安全的内容

## DOM树是如何生成的
渲染引擎内部有一个`HTML解析器HTMLParser`，值得注意的是HTML解析器并不是等整个文档加载完成后解析的，而是`网络进程加载了多少数据，HTML解析器便解析多少数据`。

### HTMLParser流程
1. 网络进程接收到响应头之后，会根据响应头中的 content-type 字段来判断文件的类型，比如 content-type 的值是“text/html”，那么浏览器就会判断这是一个 HTML 类型的文件，然后为该请求选择或者创建一个渲染进程。  
2. 渲染进程准备好后，和网络进程之间会建立一个`共享数据的管道`。
3. 网络进程接收到数据后就往管道中放，`渲染进程则从管道另一边获取数据`，并推给`HTML解析器`。
4. HTML解析器动态接收字节流，并将其`解析为DOM`。

{% asset_img 字节流生成DOM树.png 字节流生成DOM树 %}

### 解析字节流的具体流程
1. 通过分词器将字节流转换为Token,大概分为`文本TOKEN`、`Start Tag`和`End Tag`，分别对应文本标签、开始标签和结束标签
2. HTML解析器维护了一个Token栈
- 如果压入到栈中的是`StartTag Token`，HTML 解析器会`为该 Token 创建一个 DOM 节点`，然后将该节点加入到 DOM 树中，它的父节点就是`栈中相邻的那个元素生成的节点`。
- 如果分词器解析出来是`文本Token`，那么会生成一个文本节点，然后将`该节点加入到 DOM 树中`，文本Token 是`不需要压入到栈中`，它的父节点就是当前栈顶 Token 所对应的 DOM 节点。
- 如果分词器解析出来的是`EndTag 标签`，比如是EndTag div，HTML 解析器会查看 Token 栈顶的元素是否是 StarTag div，如果是，就将 StartTag div 从栈中弹出，表示该 div 元素解析完成。

## js是如何影响DOM生成的
### HTML解析器遇到js脚本时候会先暂停
HTML解析器遇到js脚本会先下载运行js脚本。
不过浏览器做了很多优化，`预解析操作`，当渲染引擎收到了字节流后，会开启一个预解析线程，用来分析HTML文件中包含的js、css等相关文件，解析到相关文件之后，预解析线程会提前下载这些文件。

### 优化的策略
- 开启CDN来加速js文件的加载
- 压缩js文件体积
- 如果js文件中没有操作DOM相关代码，就可以将该js脚本设置为异步，通过`async或者defer`属性

### js依赖CSSDOM
```js
div.style.color = 'yellow'
```
因为js引擎在解析js之前，是不知道js是否操控了CSSDOM的，所以渲染引擎在遇到js脚本时，不管该脚本是否操控了CSSDOM,都会执行`CSS文件下载，解析操作，再执行js脚本`。

Js脚本是依赖样式表的
### 优化手段
- style文件放在头部

## 总结
js的执行会阻塞DOM树的生成，css会阻碍js的执行

### 拓展
#### async和defer的区别
async会在加载完后立即执行,defer会在DOMContentLoaded事件之前执行
{% asset_image js的async和defer的区别.png js的async和defer的区别 %}

#### load事件和DOMContentLoaded事件的区别
load是当页面的html、css、js、图片等资源全部加载完毕后触发。
DOMContentLoaded是html文档被完全加载和解析完成之后触发。

## 从浏览器的渲染角度分析图片会加载吗？
关键点--浏览器渲染顺序：
1. 解析HTML，遇到img标签加载图片，构建dom树
2. 加载样式=>解析样式（遇到背景图片链接不加载）=>构建样式树
3. 把DOM树和样式树合成为渲染树(遍历DOM树时加载样式规则上的背景图片)
4. 布局
5. 绘制
### img标签display:none
```html
<img src="../image/test.png" style="display:none">
<div class="img-test" style="display:none"></div>

<style>
.img-test {
    background-image: url(../image/test.png);
}
</style>
```
结果分析：
img标签会被加载，而css背景图不会，`因为display为none,不会把该元素加入渲染树`

### 伪类元素的背景图片
```html
<div class="img-green"></div>
<style>
.img-green {
    background-image: url(../image/green.png);
}
.img-green:hover{
    background-image: url(../image/red.png);
}</style>
```
结果：hover后加载red.png
分析：hover后该样式才被加载到渲染树，才会加载背景图片

## 好文推荐
[网页性能管理详解](https://www.ruanyifeng.com/blog/2015/09/web-page-performance-in-depth.html)