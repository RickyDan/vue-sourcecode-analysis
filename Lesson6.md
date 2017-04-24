# Vue2.0 中的观察者模式

```
└── core
  └── util
    └── lang.js
```
#### 注意这个文件并没有使用 flow 做类型检测， 因为 flow 对于 Array.prototype 属性上的一些接收不定参数的函数支持并不是特别好。
#### 这个模块主要是重写和封装了一些 Array prototype 上的方法。


```js
const arrayProto = Array.prototype
export arrayMethods = Object.create(arrayProto)
```
首先缓存 Array 原型的一个副本， 然后创建一个空对象， 该对象的原型即是 Array 的原型， 然后将该对象暴露出去


```js
import { def } from '../util/index'
[
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
]
.forEach(function (method) {
  // cache original method
  const original = arrayProto[method]
  // def 方法在这里被调用
  def(arrayMethods, method, function mutator () {
    // avoid leaking arguments:
    // http://jsperf.com/closure-with-arguments
    let i = arguments.length
    const args = new Array(i)
    while (i--) {
      args[i] = arguments[i]
    }
    const result = original.apply(this, args)
    const ob = this.__ob__
    let inserted
    switch (method) {
      case 'push':
        inserted = args
        break
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    if (inserted) ob.observeArray(inserted)
    // notify change
    ob.dep.notify()
    return result
  })
})
```
这里引入了 util 目录下定义的 def 方法，该方法正是基于 Object.defineProperty的封装。首先遍历定义好的字符串
数组，遍历过程中使用 original 变量缓存 Array 原型上对应的方法。跟着调用 def 去定义  arrayMethods 这个对象
上的属性，属性名称是字符串数组的子元素，属性值为一个匿名函数。

```js
let i = arguments.length
const args = new Array(i)
while (i--) {
  args[i] = arguments[i]
}
const result = original.apply(this, args)
```
当 arrayMethods 对象上的方法被调用时，首先将传入的参数缓存到一个args数组中，最后把调用对象和参数传入原生 Array 
原型上对应的方法, 把结果缓存起来 (result 变量)

```js
const ob = this.__ob__
let inserted
switch (method) {
  case 'push':
    inserted = args
    break
  case 'unshift':
    inserted = args
    break
  case 'splice':
    inserted = args.slice(2)
    break
}
if (inserted) ob.observeArray(inserted)
// notify change
ob.dep.notify()
return result
```

这里的 this 应该是指向 观察者对象, __ob__ 是该对象的一个属性。如果调用的是push、unshift或者splice方法，则将args数组
传入 this.__ob__ 上的observeArray 方法中, 然后调用 this.__ob__ 对象上的 dep.notify 方法把变更告知观察者, 最后将
结果(result变量) 返回

观察者模式是常用的设计模式，其主要作用是将业务逻辑通过事件订阅发布的方式解耦，对于处理某一个对象状态变更后其他对象
需要更新相应的状态的情况下非常实用。 另外作者在这个模块里多次使用了变量去缓存原生对象上的属性和方法 (arrayProto 和 original),
这样做的好处是能够在不破坏原生对象上方法的同时又可以扩展自己所需的方法。
