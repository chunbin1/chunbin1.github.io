---
title: react事件原理
code_block_shrink: false
date: 2024-07-22 00:00:25
updated: 2024-07-22 00:00:25
excerpt: 简单的react事件原理
tags:
---
读了React事件源码，简单讲下React事件的原理。

1. React事件都是绑定在root节点的
React把所有原生事件绑定在了root节点上，然后处理方法使用了dispatchDiscreteEvent回到了React内部。

2. React如何把dom节点映射到Fiber节点
dom上面会有个React的内部属性，这个属性的值就是对应的Fiber节点

3. React怎么收集事件的
通过Fiber节点的return一步一步收集事件到listeners中。然后把该事件推到dispatchQueue中。

4. React是通过listeners的清空顺序来模拟捕获和冒泡的。
