---
title: git相关疑难杂症
date: 2020-11-30 10:22:17
updated: 2020-11-30 10:22:17
tags: 前端
code_block_shrink: false
excerpt: 使用git submodule分散项目的大小，[具体](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B
---
## nextjs部署k8s过程中，因为项目文件过大，上传超时
### 解决方法
使用git submodule分散项目的大小，[具体](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
然后打包的时候在DockerFile中使用
```
    git submodule init && \
    git submodule update && \
```

## 本地有改动，无法拉取远程代码
### git stash
```
git stash // 暂存，把本地变更压入栈
git pull // 领取代码
git stash pop // 把本地变更弹出栈
...继续合并等操作
```

## 某个feature、bugfix提交想要同时应用在不兼容的v1,v2分支
### git cherry-pick
[阮一峰教程](http://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)

## 如何合作开发
使用git flow,需要自己安装
develop分支代表即将上线的下个版本，master代表线上版本，每次有新的bugfix或者feature，从develop分支新建分支，完成开发后再合并回develop分支

## fork仓库后如何保持和原仓库同步
[参考文章](https://cloud.tencent.com/developer/article/1398502)
1. 添加一个将被同步的给fork远程的上游
```
git remote add upstream xxx.git 
```
2. 获取远程upstream
```
git fetch upstream
```
3. 合并upstream
```
git merge upstream/master
```