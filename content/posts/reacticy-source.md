---
title : 'Reacticy Source'
date : 2024-02-20T11:15:14+08:00
draft : true
---


**目标**:通过测试案例，来了解学习reactivity的设计实现


#### 测试使用到的plugin
```js
https://github.com/vitest-dev/vscode 虽说是deprecated，但能用就行
```

- [功能实现](#功能实现)
    - [使用jest](#使用jest)
    - [实现一个简单的响应式](#实现一个简单的响应式)

## 功能实现 

#### 使用jest
```js
方法1:打包生成es5的代码，可以使用babel
方法2:node配置esm
{
  "type": "module",
  "scripts": {
    "test": "cross-env NODE_OPTIONS=--experimental-vm-modules jest"
  },
  "devDependencies": {
    "cross-env": "^7.0.3",
    "jest": "^29.7.0"
  }
}
如果是ts文件的话，就不需要写type
使用pnpm i ts-jest -D   
```

#### 实现一个简单的响应式
```ts
// 通过reactive将对象代理变成响应式，再使用effect将响应式的值与回调函数关联起来

源码mvp核心过程：调用effect时利用全局变量存储当前执行函数，如果在函数内部调用reactive值，就利用proxy存储，并且在修改时回调
// mvp实现
//effect.ts
interface ReactiveEffect {
  (): any
  _isEffect: true
  raw: Function
  deps: Array<any>
}
export type Dep = Set<ReactiveEffect>
export type KeyToDepMap = Map<string | symbol, Dep>
export const targetMap: WeakMap<any, KeyToDepMap> = new WeakMap()

export let activeEffect: ReactiveEffect | null
interface ReactiveEffectOptions {
  lazy: boolean
}
export function effect(fn: Function, options: ReactiveEffectOptions = { lazy: false }): ReactiveEffect {
  if ((fn as ReactiveEffect)._isEffect) {
    fn = (fn as ReactiveEffect).raw
  }
  const runner = createEffect(fn, options)
  if (!options.lazy) {
    runner()
  }
  return runner
}

export function createEffect(fn: Function, options: ReactiveEffectOptions) {
  const effect = function Effect(...args) {
    activeEffect = effect //存储当前执行effect
    return fn(...args)
  } as ReactiveEffect
  effect._isEffect = true
  effect.raw = fn
  effect.deps = []
  return effect
}

// 将当前activeEffect添加至reactive的回调队列中
export function track(target: any, key?: string | symbol) {
  let effect = activeEffect
  if (effect) {
    // 获取该对象所有属性的回调队列
    let keyToDepMap = targetMap.get(target)
    if (!keyToDepMap) {
      targetMap.set(target, (keyToDepMap = new Map()))
    }
    let dep = keyToDepMap.get(key as string | symbol)
    if (!dep) {
      keyToDepMap.set(key as string | symbol, (dep = new Set()))
    }
    if (!dep.has(effect)) {
      dep.add(effect)
    }
  }
}

export function trigger(target: any, key?: string | symbol) {
  const keyToDepMap = targetMap.get(target)
  if (!keyToDepMap) {
    // 说明还没被track
    return
  }
  if (key) {
    keyToDepMap.get(key)?.forEach(effect => effect())
  }
}

//reactive.ts
import { targetMap, track, trigger } from './effect'
import { hasOwnProperty, isObject, unwrap } from './utils'

export const rawToObserved: WeakMap<any, any> = new WeakMap()
export const observedToRaw: WeakMap<any, any> = new WeakMap()
export type identity = <T>(target?: T) => T

export const reactive = ((target: any = {}): any => {
  return createObservable(target, rawToObserved, observedToRaw)
}) as identity

function createObservable(target: any, toProxy: WeakMap<any, any>, toRaw: WeakMap<any, any>) {
  if (!isObject(target)) {
    console.warn(`value is not observable:${String(target)}`)
    return target
  }
  //  判断是否已经被代理
  let proxy = toProxy.get(target)
  if (proxy) {
    return proxy
  }
  //  判断传入的是否为代理
  if (toRaw.has(target)) {
    return target
  }

  //  代理target
  proxy = new Proxy(target, {
    get(target: any, key: string | symbol, receiver: any) {
      const res = Reflect.get(target, key, receiver)
      //  存储effect
      track(target, key)
      //  实现一个懒加载以及nested reactive
      return isObject(res) ? reactive(res) : res
    },
    set(target: any, key: string | symbol, value: any, receiver: any): boolean {
      value = unwrap(value) // 避免设置的值为proxy
      const oldValue = target[key]
      const result = Reflect.set(target, key, value, receiver)
      //   如果是通过原型链访问到了这个属性的，那么不触发trigger回调
      if (target === unwrap(receiver)) {
        trigger(target, key)
      }
      return result
    }
  })
  //   双向映射
  toProxy.set(target, proxy)
  toRaw.set(proxy, target)

  if (!targetMap.get(target)) {
    // 初始化target所有key的回调依赖
    targetMap.set(target, new Map())
  }
  return proxy
}
function isObservable(value: any): boolean {
  return rawToObserved.has(value)
}
```
   