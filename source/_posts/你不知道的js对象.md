---
title: 你不知道的js对象
date: 2020-04-13 09:39:36
updated: 2020-04-13 09:39:36
tags: 前端
code_block_shrink: false
excerpt: 读《你不知道的JavaScript》笔记
---
读《你不知道的JavaScript》笔记
## [[GET]]
在对象的访问中，如果使用obj.a实际上是对obj使用了[[GET]]操作，对象默认的[[GET]]操作：
1. 在对象中查找是否有名字相同属性，有则返回值
2. 否则去原型链中寻找，有则返回值
3. 没有 返回undefined

## [[PUT]]
在对象已经存在这个属性的情况下，赋值会触发[[PUT]]，他的过程如下:
1. 属性是否是访问描述符(定义了getter或者setter)，是并且存在setter就调用setter，结束
2. 属性的数据描述符中的writable是否为false，如果是，在非严格模式下静默失败，在严格模式下抛出TypeError异常，结束。
3. 如果都不是，将该值设置为属性的值

## getter和setter
对象默认的[[PUT]]和[[Get]]操作分别可以控制属性值的设置和获取。
ES5中可以使用getter和setter部分改写默认操作，只能应用到单个属性上，无法应用到整个对象上。
getter、setter是隐藏函数，分别在获取属性值时调用、在设置属性值时调用
使用：
```js
var obj={}
Object.defineProperty(obj,'a',{
  get(){
    return this._a_;
  },
  set(val){
    this._a_ = val*2 // 把数据存在_a_中
  }
})

obj.a = 2
obj.a // 4
```


##  属性描述符
从es5开始，所有的属性都具备了属性描述符
```javascript
{
  value: 2,
  writable: true,
  enumerable: true,
  configurable: true,
}
```
属性描述符由4个属性组成value,writable(可写),enumerable(可枚举),configurable(可配置)
- writable
```javascript
var Obj = { }
Object.defineProperty(Obj,"a",{
  value: 2,
  writable: false, // 无法修改
})

Obj.a = 3 // 修改Obj
Obj.a;  // 2 任然为2
```
结论：writable为false的时候，属性值的修改会静默的失败, ps:在严格模式下，修改会抛出错误
我们可以使用Object.defineProperty(...)添加一个属性或者修改一个已有属性(如果他是configurable)
- enumerable
如果enumerable为false,for...in循环中、Object.keys不会出现
但是可以使用Object.getOwnPropertyNames获得
```javascript
var obj = {a:1,b:2}
Object.defineProperty(obj,'c',{
  value:12,
  enumerable:false,
})
for(key in obj){
  console.log(key)
} // a,b
Object.keys(obj) // ['a', 'b']
Object.getOwnPropertyNames(obj) // ['a', 'b', 'c']
```
ps: for...in会遍历原型链上的可遍历对象，Object.keys、getOwnPropertyNames只会遍历自身
- configurable
如果configurable为false，修改配置时候会产生一个TypeError错误，把configurable修改为false是单向操作，无法撤销！

##拓展: 使用es5实现const的效果
思路:使用writable控制window对象上的变量无法修改
所有声明的所有全局作用域中声明的var变量、函数都会变成window对象的属性和方法
```javascript
function _const(key,value){
  window[key] = value;
  Object.defineProperty(window,key,{
    value:value,
    writable:false,
    enumerable:false,  // const是不会挂载 不能枚举的
    configurable:false,
  })
}

_const("c",12)
c =  13
c // 12
```
修改c的值会静默的失败，那为什么不使用setter去抛出错误呢？
因为**当给一个属性定义getter、setter或者两者的时候，这个属性会被定义为“访问描述符”，对于访问描述符来说，js会忽略他们的value和writable特性，取而代之关心set和get特性**，如果定义setter，会报错，也印证了上面[[PUT]]的过程

所以如果要修改c的时候抛出错误，可以使用getter和setter
```javascript
function _const2(key,value){
  window[key] =  value;
  Object.defineProperty(window,key,{
    enumerable:false,  // const是不会挂载 不能枚举的
    configurable:false,
    get:function(){
      return value
    },
    set:function(data){
      if(data===value){
        return value
      }else{
        throw new TypeError('can not change const')
      }
    }
  })
}

_const2('bc',13)

bc= 14 // TypeError: can not change const
```
## 遍历
for...in循环可以用来遍历对象的可枚举属性列表(包括[[prototype]])。
for...of循环首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的next()方法来遍历所有返回值。
数组中内置了@@iterator，所以for...of可以直接应用到数组上
```javascript
var arr = [1,2,3]

for(var v of arr){
  console.log(v)
}

// 上等同于
var it = arr[Symbol.iterator]()

it.next();
it.next();
it.next(); // {value:3,done:false}
it.next(); // {done:true}
```

普通对象没有内置的@@iterator,所以无法自动完成for...of遍历。我们可以给任何想遍历的对象定义@@iterator
```javascript
var myObj = { a:2,b:3}

Object.defineProperty(myObj,Symbol.iterator,{
    enumerable:false,
    writable:false,
    configurable:true,
    value:function(){
      var o = this;
      var idx = 0;
      var ks = Object.keys(o);
      return {
        next:function(){
          return {
            value:o[ks[idx++]],
            done:( idx > ks.length )
          }
        }
      }
    }
})

for(v of myObj){
  console.log(v)
}
```