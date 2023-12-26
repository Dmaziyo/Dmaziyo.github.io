---
title : '手写一个bind函数'
date : 2023-12-26T18:39:34+08:00
draft : false
---
背景: 今天面试面到了，说不能使用apply和call来实现，当时脑袋空空，就放弃了,西八
```js
Function.prototype.myBind=function(self,...prePendArgs){
    // 在目标this对象上将当前函数添加上去，并且调用它
    let fn = Symbol('fn');
    self[fn] = this;
    return function(...args){
        let finalArgs =[...prePendArgs,...args]
        return self[fn](finalArgs);
    }
}

let obj1 = {
  name: '1',
  sayHi(target) {
    console.log('hi', target, this.name)
  }
}
obj1.sayHi()

let obj2 = {
  name: '2'
}

let bindFn = obj1.sayHi.myBind(obj2,'默认小马')

bindFn()
bindFn('非小马')
```