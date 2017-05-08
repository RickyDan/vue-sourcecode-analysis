## Vue2.0 观察者模式中的Watcher对象

```
└── core
  └── observer
    └── watcher.js
```

```js
import { queueWatcher } from './scheduler'
import Dep, { pushTarget, popTarget } from './dep'

import {
  warn,
  remove,
  isObject,
  parsePath,
  _Set as Set,
  handleError
} from '../util/index'

let uid = 0

export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: Set;
  newDepIds: Set;
  getter: Function;
  value: any;

  // 构造函数
  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid
    this.active = true
    this.dirty = this.lazy
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)
      if (!this.getter) {
        this.getter = function () {}
        process.env.NODE_ENV !== 'production' && warn(
          `Failed watching path: "${expOrFn}" ` + 
          'Watcher only accepts simple dot-delimited paths. ' +
          'For full control, use a function instead.',
          vm
        )
      }
    }
    this.value = this.lazy ? undefined : this.get()
  }
}
```

Watcher 构造函数接收三个必选参数，第一个参数是组件实例，第二个参数为字符串或者，第三个参数是回调函数，第四个可选参数
可以为空类型或者是对象。

```js
this.vm = vm
vm._watchers.push(this)
// options
if (options) {
  this.deep = !!options.deep
  this.user = !!options.user
  this.lazy = !!options.lazy
  this.sync = !!options.sync
} else {
  this.deep = this.user = this.lazy = this.sync = false
}
```
把vm赋值给实例属性vm, 然后把Watcher实例对象添加进vm的_watchers数组中。如果传入了options对象，则把options对象的deep、user、lazy和sync属性值赋值给实例对象上对应的属性，如果没有传入该对象，全部赋值为false

```js
this.cb = cb
this.id = ++uid // uid for batching
this.active = true
this.dirty = this.lazy // for lazy watchers
this.deps = []
this.newDeps = []
this.depIds = new Set()
this.newDepIds = new Set()
this.expression = process.env.NODE_ENV !== 'production'
  ? expOrFn.toString()
  : ''
```
这里depIds和newDepIds都是Set类型的数组，这样就保证了Id的唯一性， expression属性的值如果在开发环境下的值为expOrFn的toString字符输出
，如果在生产环境则为空字符串

```js
if (typeof expOrFn === 'function') {
  this.getter = expOrFn
} else {
  this.getter = parsePath(expOrFn)
  if (!this.getter) {
    this.getter = function () {}
    process.env.NODE_ENV !== 'production' && warn(
      `Failed watching path: "${expOrFn}" ` +
      'Watcher only accepts simple dot-delimited paths. ' +
      'For full control, use a function instead.',
      vm
    )
  }
}
this.value = this.lazy
  ? undefined
  : this.get()
```
如果传入的expOrFn是函数，就将它赋值给实例属性getter。如果传入的expOrFn是字符串，就将它传给util模块里的parsePath方法，返回值
赋值给getter属性，如果parsePath方法返回了undefined, getter属性将重新被赋值为一个空函数。如果是非生产环境下，会调用util模块
的warn方法在控制台输出警告信息。
实例属性lazy的值如果为 false, this.value的值会被赋值为undefined, 否则会调用Watcher对象的get方法


Watcher实例对象内的方法依赖于以下两个方法
```js
const seenObjects = new Set()
function traverse (val: any) {
  seenObjects.clear()
  _traverse(val, seenObjects)
}

function _traverse (val: any, seen: Set) {
  let i, keys
  const isA = Array.isArray(val)
  if ((!isA && !isObject(val)) || !Object.isExtensible(val)) {
    return
  }
  if (val.__ob__) {
    const depId = val.__ob__.dep.id
    if (seen.has(depId)) {
      return
    }
    seen.add(depId)
  }
  if (isA) {
    i = val.length
    while (i--) _traverse(val[i], seen)
  } else {
    keys = Object.keys(val)
    i = keys.length
    while (i--) _traverse(val[keys[i]], seen)
  }
}
```
