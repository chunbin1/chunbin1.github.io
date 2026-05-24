---
title: 使用nginx优化前端资源
date: 2020-04-20 08:26:52
updated: 2020-04-20 08:26:52
tags: 前端
code_block_shrink: false
excerpt: "if ($request_uri ~* .(js|css)$) {"
---
## 强缓存
```
    if ($request_uri ~* .(js|css)$) {
        add_header Cache-Control  "public,max-age=31536000,immutable";
    }
```
### 适用场景
所有的js和css打包的时候都使用hash唯一标识
### 原理
- 浏览器使用Cache-Control做缓存控制，`max-age=31536000,immutable`告诉浏览器可以缓存，下次加载资源浏览器先从本地缓存拿，返回`200(from memory cache)`从内存或者`200(from dist cache)`从本地
- 前端打包时候改变的文件使用hash标识，只有文件改变才修改hash
### 拓展
1. 缓存控制策略：使用webpack的`optimization`里的`splitChunks`把`公共的`、`不常变化`的库提取出来，这样通过强缓存，可以让依赖于该库的界面只需要加载少量资源
2. [webpack的hash策略](https://juejin.im/post/5d7eedf0e51d4562165535ae#heading-1)
3. Expires和Cache-control的区别
  Expires 如：Thu, 01 Dec 2019 16:00:00 GMT  
  表示资源的具体过期时间，过期了就得向服务端发请求，指定的是一个时间，可能存在服务器资源和电脑本地时间不同步的问题。  
  Cache-control，指定从请求的时间开始，允许获取的响应被重用的最长时间（秒）  
  若俩者同时存在Expires则被Cache-Control的max-age覆盖


## gzip
```
    gzip on; # 开启gzip
    gzip_min_length 1k; # 大于该值 gzip
    gzip_comp_level 2; # gzip的力度,越大越消耗服务器性能
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/json application/xml+rss application/rss+xml application/atom+xml image/svg+xml; # 标识gzip的类型
```

### 适用场景
几乎所有,但是照片等不适合gzip,会变得更大。  
`gzip_min_length 1k;`这个配置最好要开，太小的资源gzip后可能比原来大

### 原理
- 请求时候`accept-encoding`字段，告诉服务器能接受的编码方式，Response Headers中 `coding-encoding`告诉浏览器解码方式
- [gzip的算法原理](https://juejin.im/post/5b793126f265da43351d5125#heading-3) 拓展: [gzip原理](https://luyuhuang.github.io/2020/04/28/gzip-and-deflate.html)

### 优化
gzip会消耗服务器性能，而以nginx为例，它会先搜寻.gz文件并返回，所以我们只要提前压缩好放在服务器中，可以减少一步服务器压缩的过程，使用`compression-webpack-plugin`即可完成

## 好文推荐
[玩转Nginx](http://blog.hszofficial.site/recommend/2019/03/20/%E7%8E%A9%E8%BD%ACNginx/)