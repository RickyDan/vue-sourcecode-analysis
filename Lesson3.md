# Vue dev 模式下的调试信息
utils目录主要是存放一些公共的基础方法
```
 └── core
   └── util
    └── debug.js
    └── env.js
    └── error.js
    └── lang.js
    └── index.js
    └── options.js
    └── perf.js
    └── props.js
```
debug.js下主要是存放一些在开发模式下输出一些调试信息给开发者提示，调试Vue变得更容易些
该模块主要暴露了三个方法, warn, tips, formatComponentName, 另外有一个内部共用的formatLocation方法

```js
 const formatLocation = str => {
    if (str === `<Anonymous>`) {
      str += ` - use the "name" option for better debugging messages.`
    }
    return `\n(found in ${str})`
}
```
formatLocation方法是在Vue开发过程中如果引入了未声明的标签时的报错提示信息


```js
formatComponentName = (vm, includeFile) => {
    if (vm.$root === vm) {
      return '<Root>'
    }
    let name = typeof vm === 'string'
      ? vm
      : typeof vm === 'function' && vm.options
        ? vm.options.name
        : vm._isVue
          ? vm.$options.name || vm.$options._componentTag
          : vm.name

    const file = vm._isVue && vm.$options.__file
    if (!name && file) {
      const match = file.match(/([^/\\]+)\.vue$/)
      name = match && match[1]
    }

    return (
      (name ? `<${classify(name)}>` : `<Anonymous>`) +
      (file && includeFile !== false ? ` at ${file}` : '')
    )
  }
  ```
  

formatComponentName是调试时候更加友好地输出组件的名称以便开发人员更快的定位到问题组件中的方法

#### env.js主要是存放一些检测vue开发环境的代码

```js
import { noop } from 'shared/util'
import { handleError } from './error'

// can we use __proto__?
export const hasProto = '__proto__' in {}

// Browser environment sniffing
export const inBrowser = typeof window !== 'undefined'
export const UA = inBrowser && window.navigator.userAgent.toLowerCase()
export const isIE = UA && /msie|trident/.test(UA)
export const isIE9 = UA && UA.indexOf('msie 9.0') > 0
export const isEdge = UA && UA.indexOf('edge/') > 0
export const isAndroid = UA && UA.indexOf('android') > 0
export const isIOS = UA && /iphone|ipad|ipod|ios/.test(UA)
export const isChrome = UA && /chrome\/\d+/.test(UA) && !isEdge
```
这段代码主要是用于检测浏览器环境的，包括各大主流浏览器平台的检测, IE的版本至少要9以上才支持Vue

 Vue2.0版本开始提供服务端渲染的支持，下面的这段代码是用于检测当前的js执行环境是否是在nodejs中, _isServerRendering会返回一个Boolean, 为true表示执行环境在NodeJS中
 
 ```js
 // this needs to be lazy-evaled because vue may be required before
// vue-server-renderer can set VUE_ENV
let _isServer
export const isServerRendering = () => {
  if (_isServer === undefined) {
    /* istanbul ignore if */
    if (!inBrowser && typeof global !== 'undefined') {
      // detect presence of vue-server-renderer and avoid
      // Webpack shimming the process
      _isServer = global['process'].env.VUE_ENV === 'server'
    } else {
      _isServer = false
    }
  }
  return _isServer
}
 ```

 另外在开发模式下为了更好的调试 Vue, 推荐安装一款chrome插件 vue-devtool, 下面这段代码是检测浏览器是否安装了 vue-devtool 插件
 ```js
 export const devtools = inBrowser && window.__VUE_DEVTOOLS_GLOBAL_HOOK__
```

 ```js
 /* istanbul ignore next */
export function isNative (Ctor: any): boolean {
  return typeof Ctor === 'function' && /native code/.test(Ctor.toString())
```
isNative函数用于检测一个值是否是 JS 内置的函数, 如果是则返回 true, 否则返回 false


```js
export const hasSymbol =
  typeof Symbol !== 'undefined' && isNative(Symbol) &&
  typeof Reflect !== 'undefined' && isNative(Reflect.ownKeys)
```
Symbol是ES6新增的一种基本数据类型，Reflect也是ES6新引入为操作对象提供的新的 API


```js
// nextTick方法在需要直接操作DOM的情况下，在修改数据后，能够在数据更新到DOM后才执行对应的函数
export const nextTick= (function () {
  // 回调函数数组
  const callbacks = []
  let pending = false
  let timerFunc

  // 回调函数的轮询调用， 类似于EventLoop事件机制
  // nextTickHandler 在DOM节点更新后， 取出回调函数数组中的函数逐一执行
  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callback.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }

  // 如果支持Promise，则优先使用Promise.then 来异步执行nextTickHandler
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      if (iOS) setTimeout(noop)
    }
  /* 如果不支持Promise, 则检测浏览器是否内置了MutationObserver 这个对象
  MutationObserver(MO） 的主要作用是可以通过它创建一个观察者对象，这个对象会监
  听给定的某个DOM元素，并在它的DOM树发生变化时执行到我们提供的回调函数。MO接受一个
  回调函数作为参数，返回一个observer对象，observer对象的observe方法接收一个DOM
  元素和一个观察配置对象作为参数。具体的使用方法可以查看 MDN 对应的MutationObserver文档
  */
  } else if (typeof MutationObserver !== 'undefined' && (isNative(MutationObserver) ||
    MutationObserver.toString() === '[object MutationObserverConstructor]')) {
      var counter = 1
      var observer = new MutationObserver(nextTickHandler)
      var textNode = document.createTextNode(String(counter))
      observer.observe(textNode, {
        characterData: true
      })
      timerFunc = () => {
        counter = (counter + 1) % 2
        textNode.data = String(counter)
      }
    } else {
      timerFunc = () => {
        // setTimeout 方法是在前面两种方法都不支持的情况下才用来异步执行nextTickHandler
        setTimeout(nextTickHandler, 0)
      }
    }

    return function queueNextTick (cb?: Function, ctx?: Object) {
      let _resolve
      callbacks.push(() => {
        if (cb) cb.call(ctx)
        if (_resolve) _resolve(ctx)
      })
      if (!pending) {
        pending = true
        timerFunc()
      }
      if (!cb && typeof Promise !== 'undefined') {
        return new Promise((resolve, reject) => {
          _resolve = resolve
        })
      }
    }
})()
```

```js
let _Set
/* istanbul ignore if */
if (typeof Set !== 'undefined' && isNative(Set)) {
  // use native Set when available.
  _Set = Set
} else {
  // a non-standard Set polyfill that only works with primitive keys.
  _Set = class Set {
    set: Object;
    constructor () {
      this.set = Object.create(null)
    }
    has (key: string | number) {
      return this.set[key] === true
    }
    add (key: string | number) {
      this.set[key] = true
    }
    clear () {
      this.set = Object.create(null)
    }
  }
}

export { _Set }
```

Set是 ES6 新增的一种数据结构，类似于数组，但是Set内的值都是唯一的, 没有重复的值，下面这段代码中首先检测环境中是否已经内置了 Set 构造函数，如果存在则将其缓存在变量 _Set中，否则就自己实现一个 Set 类, 这个类暴露了 has、add和 clear方法



error.js 模块只有一个简单的报错提示方法, 代码如下

```js
import config from '../config'
import { warn } from './debug'
import { inBrowser } from './env'

export function handleError (err, vm, info) {
  if (config.errorHandler) {
    config.errorHandler.call(null, err, vm, info)
  } else {
    if (process.env.NODE_ENV !== 'production') {
      warn(`Error in ${info}: "${err.toString()}"`, vm)
    }
    /* istanbul ignore else */
    if (inBrowser && typeof console !== 'undefined') {
      console.error(err)
    } else {
      throw err
    }
  }
}
```

handleError 方法首先检测 config 模块下是否配置了 errorHandler 处理方法，如果有就直接调用, 否则先检测是否是在生产环境中，生产环境中则用
warn 方法输出警告信息，如果是在浏览器端渲染并且console模块存在的话则使用console.error() 方法输出错误信息, 否则把错误抛出