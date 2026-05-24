---
title: umi-request源码分析
date: 2020-05-21 10:49:15
updated: 2020-05-21 10:49:15
tags: 前端
code_block_shrink: false
excerpt: "\"isomorphic-fetch\": \"^2.2.1\", // 同构的fetch请求库"
---
## 依赖分析
```json
"isomorphic-fetch": "^2.2.1", // 同构的fetch请求库
"qs": "^6.9.1" // 解析或格式化请求
```
使用isomorphic-fetch,底层使用fetch请求
## 从特性到源码
### request options定义
```ts
export interface RequestOptionsInit extends RequestInit {
  charset?: 'utf8' | 'gbk';
  requestType?: 'json' | 'form';
  data?: any;
  params?: object | URLSearchParams;
  paramsSerializer?: (params: object) => string;
  responseType?: ResponseType;
  useCache?: boolean;
  ttl?: number;
  timeout?: number;
  errorHandler?: (error: ResponseError) => void;
  prefix?: string;
  suffix?: string;
  throwErrIfParseFail?: boolean;
  parseResponse?: boolean;
  cancelToken?: CancelToken;
  getResponse?: boolean;
  validateCache?: (url: string, options: RequestOptionsInit) => boolean;
  __umiRequestCoreType__?: string;
}
```
### extend
一般我们项目中的请求很多处理都一样，所以可以使用extend来定制配置，然后调用只需要直接调用请求即可。
#### 创建一个request实例
```js
export const extend = initOptions => request(initOptions); 
```

### request实例
#### 创建实例
```js
const request = (initOptions = {}) => {
  const coreInstance = new Core(initOptions);
  const umiInstance = (url, options = {}) => {
    const mergeOptions = mergeRequestOptions(coreInstance.initOptions, options); //合并请求选项
    return coreInstance.request(url, mergeOptions); // 请求数据
  };
  ...
  return umiInstance; // 返回一个请求对象
};
```
#### .get、.post方法的语法糖
```js
  const METHODS = ['get', 'post', 'delete', 'put', 'patch', 'head', 'options', 'rpc'];
  METHODS.forEach(method => {
    umiInstance[method] = (url, options) => umiInstance(url, { ...options, method });
  });
```
#### 拓展实例
```js
export const extend = initOptions => request(initOptions);
```



### Core对象
#### 构造函数
```js
  constructor(initOptions) {
    this.onion = new Onion([]); // 中间件
    this.fetchIndex = 0; // 【即将废弃】请求中间件位置
    this.mapCache = new MapCache(initOptions);
    this.initOptions = initOptions;
    this.instanceRequestInterceptors = []; // 实例请求拦截器
    this.instanceResponseInterceptors = []; // 实例响应拦截器
  }
```
#### request方法
```js
  request(url, options) {
    const { onion } = this;
    const obj = {  // 这个就是ctx
      req: { url, options },
      res: null,
      cache: this.mapCache,  // 缓存
      responseInterceptors: [...Core.responseInterceptors, ...this.instanceResponseInterceptors],
    };
    if (typeof url !== 'string') {
      throw new Error('url MUST be a string');
    }

    return new Promise((resolve, reject) => {
      this.dealRequestInterceptors(obj) // 执行请求前拦截器
        .then(() => onion.execute(obj)) // 执行中间件
        .then(() => {
          resolve(obj.res);
        })
        .catch(error => {
          const { errorHandler } = obj.req.options;
          if (errorHandler) {
            try {
              const data = errorHandler(error); // 错误处理
              resolve(data);
            } catch (e) {
              reject(e);
            }
          } else {
            reject(error);
          }
        });
    });
  }
```

### 拦截器
#### 全局拦截器和实例拦截器
```js
class Core{
  constructor(initOptions) {
    // ...
    this.instanceRequestInterceptors = [];
    this.instanceResponseInterceptors = [];
  }
  static requestInterceptors = [addfixInterceptor];
  static responseInterceptors = [];
}
```
##### 注册拦截器
```js
  static requestUse(handler, opt = { global: true }) {
    if (typeof handler !== 'function') throw new TypeError('Interceptor must be function!');
    if (opt.global) { // 判断拦截器的层级
      Core.requestInterceptors.push(handler);
    } else {
      this.instanceRequestInterceptors.push(handler);
    }
  }
```
#### 请求拦截
```js
  dealRequestInterceptors(ctx) {
    const reducer = (p1, p2) =>
      p1.then((ret = {}) => {  // 这里其实return的是p1,Promise聚合所有拦截器的处理方法
        ctx.req.url = ret.url || ctx.req.url;
        ctx.req.options = ret.options || ctx.req.options;
        return p2(ctx.req.url, ctx.req.options);
      });
    const allInterceptors = [...Core.requestInterceptors, ...this.instanceRequestInterceptors]; // 实例拦截器和全局拦截器
    return allInterceptors.reduce(reducer, Promise.resolve()).then((ret = {}) => {
      ctx.req.url = ret.url || ctx.req.url;
      ctx.req.options = ret.options || ctx.req.options;
      return Promise.resolve();
    });
  }
```
值得注意的是，其中的reducer的用法相当于
```js
Promise.resolve()
.then(() => console.log(1))  // 执行任务1
.then(() => console.log(2))  // 执行任务2
.then(() => console.log(3))  // 执行任务3
```

### 中间件
umi-request有
1. 实例中间件
2. 全局中间件，不同实例共享全局中间件
3. 内核中间件，方便开发者拓展请求内核

#### 中间件对象
```js
class Onion{
  constructor(defaultMiddlewares) {
    if (!Array.isArray(defaultMiddlewares)) throw new TypeError('Default middlewares must be an array!');
    this.defaultMiddlewares = [...defaultMiddlewares];
    this.middlewares = [];
  }
  static globalMiddlewares = []; // 全局中间件 使用class的静态声明
  static defaultGlobalMiddlewaresLength = 0; // 内置全局中间件长度
  static coreMiddlewares = []; // 内核中间件
  static defaultCoreMiddlewaresLength = 0; // 内置内核中间件长度
}
```
#### 添加中间件
```js
use(newMiddleware, opts = { global: false, core: false, defaultInstance: false }) {}
```
通过global和core判断内核中间件和全局中间件
```js
    // 全局中间件
    if (global) {
      Onion.globalMiddlewares.splice(
        Onion.globalMiddlewares.length - Onion.defaultGlobalMiddlewaresLength,
        0,
        newMiddleware
      );
      return;
    }
    // 内核中间件
    if (core) {
      Onion.coreMiddlewares.splice(Onion.coreMiddlewares.length - Onion.defaultCoreMiddlewaresLength, 0, newMiddleware);
      return;
    }
```
#### 执行中间件
首先合并中间件
```js
    const fn = compose([
      ...this.middlewares,
      ...this.defaultMiddlewares,
      ...Onion.globalMiddlewares,
      ...Onion.coreMiddlewares,
    ]);
```
具体合并中间件逻辑
```js
export default function compose(middlewares) {
  if (!Array.isArray(middlewares)) throw new TypeError('Middlewares must be an array!');

  const middlewaresLen = middlewares.length;
  for (let i = 0; i < middlewaresLen; i++) {
    if (typeof middlewares[i] !== 'function') {
      throw new TypeError('Middleware must be componsed of function');
    }
  }

  return function wrapMiddlewares(params, next) {
    let index = -1;
    function dispatch(i) {
      if (i <= index) {
        return Promise.reject(new Error('next() should not be called multiple times in one middleware!'));
      }
      index = i;
      const fn = middlewares[i] || next;
      if (!fn) return Promise.resolve();
      try {
        return Promise.resolve(fn(params, () => dispatch(i + 1))); // 执行下一个中间件
      } catch (err) {
        return Promise.reject(err);
      }
    }

    return dispatch(0); // 执行第一个中间件
  };
}
```



### 请求过程
#### 默认中间件
umi-request的请求逻辑也是通过中间件实现的
```js
// 初始化全局和内核中间件
const globalMiddlewares = [simplePost, simpleGet, parseResponseMiddleware];
const coreMiddlewares = [fetchMiddleware];
```
#### simplePost中间件
对POST请求参数做处理，实现 query 简化、 post 简化

#### simpleGet中间件
对GET请求参数做处理，实现 query 简化、 post 简化

#### fetch请求中间件
处理请求的中间件


### 取消请求的实现
#### 如何取消请求
umi-request中取消请求  
你可以通过`CancelToken.source`来创建一个 cancel token
```js
import Request from 'umi-request';

const CancelToken = Request.CancelToken;
const { token, cancel } = CancelToken.source();

Request.get('/api/cancel', {
  cancelToken: token
}).catch(function(thrown) {
  if (Request.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理异常
  }
});

Request.post('/api/cancel', {
  name: 'hello world'
}, {
  cancelToken: token
})

// 取消请求(参数为非必填)
cancel('Operation canceled by the user.');
```
#### CancelToken
```js
function CancelToken(executor) {
  if (typeof executor !== 'function') {
    throw new TypeError('executor must be a function.');
  }

  var resolvePromise;
  this.promise = new Promise(function promiseExecutor(resolve) {
    resolvePromise = resolve;
  });

  var token = this;
  executor(function cancel(message) {
    if (token.reason) {
      // 取消操作已被调用过
      return;
    }

    token.reason = new Cancel(message);
    resolvePromise(token.reason); // cancel的时候结束这个Promise
  });
}


/**
 * 通过 source 来返回 CancelToken 实例和取消 CancelToken 的函数
 */
CancelToken.source = function source() {
  var cancel;
  var token = new CancelToken(function executor(c) {
    cancel = c; // 把function cancel赋值给cancel
  });
  return {
    token: token, // 这是一个CancelToken实例
    cancel: cancel,
  };
};

```

#### fetch中间件取消请求
```js
response = Promise.race([cancel2Throw(options, ctx), adapter(url, options)]);
```
使用Promise.race，当一个Promise完成，就不在等待别的Promise
`cancel2Throw`方式是用来取消请求的
```js
export function cancel2Throw(opt) {
  return new Promise((_, reject) => {
    if (opt.cancelToken) {
      opt.cancelToken.promise.then(cancel => {
        reject(cancel); // 这里的cancelToken.promise如果resole了 那这个promise就reject
      });
    }
  });
}
```
##### 超时取消
超时取消也是使用了`Promise.race`
```js
response = Promise.race([cancel2Throw(options, ctx), adapter(url, options), timeout2Throw(timeout, ctx.req)]);

function timeout2Throw(msec, request) {
  return new Promise((_, reject) => {
    setTimeout(() => { // 定时器
      reject(new RequestError(`timeout of ${msec}ms exceeded`, request, 'Timeout'));
    }, msec);
  });
}
```