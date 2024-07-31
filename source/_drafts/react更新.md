---
title: react更新
code_block_shrink: false
date: 2024-07-22 20:57:16
updated: 2024-07-22 20:57:16
excerpt: 介绍React的更新机制
tags:
---
setState发生了什么？
调用了updater的enqueueSetState方法，然后将update对象加入到队列updateQueue中。
然后再启动调度

