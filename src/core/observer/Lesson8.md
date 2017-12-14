## Vue2.0 观察者模式中的Watcher对象

```
└── core
  └── observer
    └── watcher.js
```

```js
/* flow */
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
  /* 每个组件实例都有相应的 watcher 实例对象，
  它会在组件渲染的过程中把属性记录为依赖，
  之后当依赖项的 setter 被调用时，会通知 watcher 重新计算，
  从而致使它关联的组件得以更新 */
  vm: Component;
  expression: string;
  cb: Function; // 回调函数
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>; // Dep类的实例数组
  newDeps: Array<Dep>; // Dep类的实例数组
  depIds: Set; // dep实例id属性的集合
  newDepIds: Set; // dep实例id属性的集合
  getter: Function;
  value: any;

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
    this.value = this.lazy
      ? undefined
      : this.get()
  }

  /* Evaluate the getter, and re-collect dependencies */
  get () {
    // 将 watcher 对象添加进栈
    pushTarget(this)
    let value
    const vm = this.vm // 指向组件的属性
    if (this.user) {
      try {
        // 调用 getter 方法去计算出 value 的值
        value = this.getter.call(vm, vm)
      } catch (e) {
        handleError(e, vm, `getter for watcher "${this.expression}"`)
      }
    } else {
      value = this.getter.call(vm, vm)
    }
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      traverse(value)
    }
    // 将 watcher 对象移除出栈
    popTarget()
    this.cleanDeps()
    return value
  }

  // Add a dependency to this directive
  addDep (dep: Dep) {
    const id = dep.id
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        dep.addSub(this)
      }
    }
  }

  // Clean up for dependency collection
  cleanupDeps () {
    let i = this.deps.length
    while (i--) {
      const dep = this.deps[i]
      if (!this.newDepIds.has(dep.id)) {
        dep.removeSub(this)
      }
    }
    let tmp = this.depIds
    this.depIds = this.newDepIds
    this.newDepIds = tmp
    this.newDepIds.clear()
    tmp = this.deps
    this.deps = this.newDeps
    this.newDeps = tmp
    this.newDeps.length = 0
  }

  // Subscriber intergace Will be called when a dependency changes
  update () {
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      this.run()
    } else {
      queueWatcher(this)
    }
  }

  // Scheduler job interface Will be called by the scheduler
  run () {
    if (this.active) {
      const value = this.get()
      if (
        value !== this.value ||
        // Depp watchers and watchers on Object/Arrays should fire even
        // When the value is the same, because the value map
        // have mutated
        isObject(value) ||
        this.deep
      ) {
        // set the value
        const oldValue = this.value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }

  // Evaluate the value of the watcher This only gets called lazy watchers
  evaluate () {
    this.value = this.get()
    this.dirty = false
  }

  // Depend on all deps collected by this watcher
  depend () {
    let i = this.deps.length
    while (i--) {
      this.deps[i].depend()
    }
  }

  // Remove self from all dependencies' subscriber list
  teardown () {
    if (this.active) {
      // remove self from vm's watcher list
      // this is a somewhat expensive operation so wo skip it
      // if the vm is being destroyed
      if (!this.vm._isBeingDestroyed) {
        remove(this.vm._watchers, this)
      }
      let i = this.deps.length
      while (i--) {
        this.deps[i].removeSub(this)
      }
      this.active = false
    }
  }

  // Recursively traverse an object to evoke all converted
  // getters, so that every nested property inside the object
  // is collected as a "deep" dependency
  const seenObjects = new Set()
  // 接收一个任意类型的值，调用集合的 clear() 方法，保证传入 _traverse 方法的是一个空集
  function traverse (val: any) {
    seenObjects.clear()
    _traverse(val, seenObjects)
  }
  // _traverse 接收一个任意类型的数值和一个集合类型
  function _traverse (val: any, seen: Set) {
    let i, key
    const isA = Array.isArray(val)
    // 如果传入的 val 不是数组也不是对象或者是一个不可扩展的对象，直接return
    if ((!isA && !isObject(val)) || !Object.isExtensible(val)) {
      return
    }
    if (val.__ob__) {
      // 如果 __ob__ 属性存在，则val.__ob__.dep.id 的值提取出来
      const depId = val.__ob__.dep.id
      // 判断 seen 集合中是否已经存在相同的 depId，如果存在则直接返回，否则把 depId 添加进 seen 集合中
      if (seen.has(depId)) {
        return
      }
      seen.add(depId)
    }
    if (isA) {
      // 如果val是数组，遍历该数组，对数组中的每一个元素递归调用 _traverse 自身
      i = val.length
      while (i--) _traverse(val[i], seen)
    } else {
      // 如果 val 是对象，遍历该对象，对对象中的每一个属性递归调用 _traverse 自身
      keys = Object.keys(val)
      i = keys.length
      while (i--) _traverse(val[keys[i]], seen)
    }
  }
}
```