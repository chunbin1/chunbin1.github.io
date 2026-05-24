---
title: kubectl
date: 2020-04-10 10:11:19
updated: 2020-04-10 10:11:19
tags: 前端
code_block_shrink: false
excerpt: kubectl是k8s的命令行工具
---
kubectl是k8s的命令行工具

## 安装
https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/

## 安装自动补全
这里比较推荐
https://github.com/c-bata/kube-prompt

## 使用
```
kube-prompt // 进入命令行交互界面 带补全
```

## 命令
注意：**以下命令 如果没有使用kube-prompt 前面要加上 kubectl**
### 查看帮助
```
help
```
### 获取pod
```
get pods // 获取所有pods

get pods --show-labels // 查看标签

get pods [podName] -o yaml // 查看生成该pod对应的yaml文件
```

### label相关
label由jianjian
```
label pods [podName] env=test --overwrite  // 修改标签

label pods [podName] env-  // 去除env标签

get pods --show-labels -l app=test-cloud-text-ds // 根据labels搜索标签

get pods --show-labels -l 'env in (test,env)' // 获得标签env为test或者env的pods
```

