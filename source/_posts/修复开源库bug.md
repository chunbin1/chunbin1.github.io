---
title: 修复开源库bug
code_block_shrink: false
date: 2024-08-01 11:50:44
updated: 2024-08-01 11:50:44
excerpt: 以preact为例子介绍如何修复开源库bug
tags:
---
以preact为例子
## 问题描述
preact@10.4.7中useRef出现了与react不符合的返回，导致使用了useRef的库报错，提MR太慢了，我们需要快速上线，需要一套`优雅`的解决方案

## 解决方案
### patch-package
#### 安装
```powershell
yarn add patch-package -D
```

#### 修改node_modules/preact
1. 修改package.json中的main字段,不使用打包后的文件，打包后的代码我们无法修改
```js
diff --git a/node_modules/preact/hooks/package.json b/node_modules/preact/hooks/package.json
index 1e8d66a..cc0dab6 100644
--- a/node_modules/preact/hooks/package.json
+++ b/node_modules/preact/hooks/package.json
@@ -4,7 +4,7 @@
   "version": "0.1.0",
   "private": true,
   "description": "Hook addon for Preact",
-  "main": "dist/hooks.js",
+  "main": "src/index.js",
   "module": "dist/hooks.module.js",
   "umd:main": "dist/hooks.umd.js",
   "source": "src/index.js",
```
拓展:为啥是修改main字段,这是是由打包工具webpack决定的,现象：[package.json中,browser，module，main 优先级](https://segmentfault.com/a/1190000019438150)

2. 修改bug
```js

diff --git a/node_modules/preact/hooks/src/index.js b/node_modules/preact/hooks/src/index.js
index 72d1467..394e3fc 100644
--- a/node_modules/preact/hooks/src/index.js
+++ b/node_modules/preact/hooks/src/index.js
@@ -174,10 +174,7 @@ export function useLayoutEffect(callback, args) {
 
 export function useRef(initialValue) {
 	currentHook = 5;
-	return useMemo(
-		() => ({ current: initialValue === undefined ? null : initialValue }),
-		[]
-	);
+	return useMemo(() => ({ current: initialValue }), []);
 }
```

#### 使用

##### 生成补丁
```shell
npx patch-package preact --exclude '^$' 
```
ps：默认不会记录package.json，所以使用--exclude覆盖

##### 使用补丁
在ci中打包前也要使用这个命令
```shell
git apply --ignore-whitespace patches/preact+10.4.7.patch
```

##### 撤销所有修改
如果以后preact修复了bug,可以更新preact然后撤销修改

```shell
npx patch-package --reverse 
```

#### 优势

1. 可以快速、优雅的解决一些开源库的bug
2. 可以减少复制粘贴的工作量


