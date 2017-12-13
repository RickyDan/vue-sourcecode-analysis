# Vue 如何定义一个属性

```
└── core
  └── util
    └── lang.js
```

这个模块暴露了三个方法和一个不可变的空对象

```js
export const emptyObject = Object.freeze({})
```

emptyObject 就是一个不可变对象，不能往该对象里添加或修改属性，也就是这个对象是只读的

```js
export function isReserved (str: string): boolean {
  const c = (str + '').charCodeAt(0)
  return c === 0x24 || c === 0x5F
}
```
isReserved 方法接收一个字符串参数， 返回一个布尔值。该方法的作用是检查一个字符串是否是一个 $ 符号或者 _ 开头，如果是则返回true, 否则返回 false
Vue 自身很多关键字或API都是以$符号和_开头的，这个方法可以防止自定义的变量名称与Vue内置的API名称冲突

```js
export function def (obj: Object, key: string, val: any, enumerable?: boolean) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  })
}
```
def 方法是Vue 实现 MVVM 的核心方法。该方法接收一个对象，一个 key 值， 一个 value 值 和一个布尔值为参数, 最后一个参数布尔值可以不传。def 方法
实际是调用了 Object 对象上的 defineProperty 方法来定义一个对象的属性和值，以及该属性是否是可枚举的


```js
/*
bailRE 会解析简单的路径，比如 a.b.c 是合法的，不会被匹配， 而 a.b[0] 是不合法的，会被正则捕获到，直接return
Vue 中 data 会返回一个对象，Vue 会去遍历这个对象，对每个属性调用 def 方法去定义对象的属性和值，因为数组是没有defineProperty方法的，所以数组的变化无法变监听到
*/
const bailRE = /[^\w.$]/
export function parsePath (path: string): any {
  if (bailRE.test(path)) {
    return
  }
  const segments = path.split('.')
  return function (obj) {
    for (let i = 0; i < segments.length; i++) {
      if (!obj) return
      obj = obj[segments[i]]
    }
    return obj
  }
}
```
