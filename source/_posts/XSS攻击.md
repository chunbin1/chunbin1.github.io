---
title: XSS攻击
date: 2020-06-04 11:42:08
updated: 2020-06-04 11:42:08
tags: 前端
code_block_shrink: false
excerpt: 读了[如何防止XSS攻击](https://juejin.im/post/5bad9140e51d450e935c6d64#heading-26),有了一些笔记
---
读了[如何防止XSS攻击](https://juejin.im/post/5bad9140e51d450e935c6d64#heading-26),有了一些笔记和思考
## 什么是XSS攻击
XSS 攻击是页面被注入了恶意的代码

### 从一个实际例子出发
```json
"data":"<div style='color:green'>你好</div>"
```
这是客户传上来的，进行文字识别的字符串，后端不加转码就存储起来。
而此时前端有了`展示需求`,他采用了以下写法
```js
// react
<div dangerouslySetInnerHTML={{__html:data}}></div>
```
这样，前端就会渲染出<div style="color:green;">绿色你好</div>
不只是，黑客还能拿到你的cookie
点击拿到cookie
```js
const xss = '<a href=javascript:alert(document.cookie)>测试</a>';
```

值得注意的是 插入`script`标签不会执行,这是因为react的dangerouslySetInnerHTML使用了innerHTML来设置，H5不会执行innerHTML的内容，具体查看这里[MDN innerHTML](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/innerHTML)
```js
const xss = '<script>alert(document.cookie)</script>';
```

但是不使用script标签运行js的方法非常多
```js
const xss = "<img src='x' onerror='alert(document.cookie)'>";
```

## XSS攻击分类
### 存储型
1. 攻击者把恶意代码提交到目标的数据库中
2. 网站服务端把恶意代码拼接到HTML中返回给浏览器
3. 浏览器接收到响应后解析执行
4. 恶意代码窃取用户数据，并发送到攻击者网站

### 反射型
1. 攻击者构造出特殊的url,其中包含恶意代码
2. 用户打开有恶意代码的URL，网站服务端将恶意代码从URL中取出，拼接在HTML中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站

### DOM型XSS
1. 攻击者构造出特殊的URL，包含恶意代码
2. 用户打开带恶意代码的URL
3. 用户浏览器收到响应后解析执行，前端Js取出URL中的恶意代码并执行
4. 恶意代码窃取用户数据并发送到攻击者的网站

## React中针对XSS攻击做的处理
### 使用textContent来做内容插入
innerHTML会有XSS风险，而textContent不会有这个风险，textContent会把内容解析为纯文本文本。
### 当遇到script标签和dangerouslySetInnerHTML时
使用innerHTML插入节点，可是仍然有风险
### $$typeof
$$typeof为Symobol.for('react.element'),通过`Symbol无法被json格式化`，所以服务端`不能使用json来伪造虚拟DOM`  
具体可以看[$$typeof]('https://overreacted.io/why-do-react-elements-have-typeof-property/')

## 如何防范XSS攻击
### 输入过滤
数据库存储的时候对关键字符进行转码`5 < 7`转为`5 &lt; 7`，但是这会导致一个问题：
比如渲染环境不是浏览器，而是`手机端`，则&lt；不会被正确的转吗,在提交阶段，我们并不确定内容要数据到哪里，所以`这种方法应该避免`。  
不过可以对明确的数据类型，如`数字、URL、电话号码、邮件地址`等，可以过滤

### 转义HTML
在模版引擎中，使用转义库，如`org.owasp.encoder`

### 预防DOM型XSS攻击
- 尽量使用`.textContent`、`.setAttribute`，来把不可信的数据作为HTML插到页面上。
- 使用成熟的框架,Vue/React等，且不要使用v-html/dangerouslySetInnerHTML
- DOM中的内联事件监听器，如`location`、`onclick`、`onerror`、`onload`、`onmouseover`等，`<a>`标签的`href`属性，js的`eval()`、`setTimeout()`、`setInterval()`等，都能把字符串当作代码运行，不要把`不可信`的数据拼接到字符串传递给这些API

### 其他方法
- `HTTP-only`,Cookie禁止js读取某些敏感Cookie
- `eslint`,避免一些存在安全隐患的写法

#### Content Security Policy
- 禁止加载外域代码，防止复杂的攻击逻辑。
- 禁止外域提交，网站被攻击后，用户的数据不会泄露到外域。
- 禁止内联脚本执行（规则较严格，目前发现 GitHub 使用）。
- 禁止未授权的脚本执行（新特性，Google Map 移动版在使用）。
- 合理使用上报可以及时发现 XSS，利于尽快修复问题。


