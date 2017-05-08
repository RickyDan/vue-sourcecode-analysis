# Vue2.0 中的观察者模式

```js
import type Watcher from './watcher'
import { remove } from '../util/index'

let uid = 0

export default class Dep {
  static target: ? Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

Dep.target = null
const targetStack = []

export function pushTarget (_target: Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}
```

Dep 类有三个属性，静态属性target, 值为 watcher对象或者为null。实例属性id 和 subs属性为watcher对象数组
每次实例化Dep 类时，id 属性会自增，addSub、removeSub方法是往subs数组里添加或删除Watcher对象，depend方法
首先判断target属性是否为空，若不为空，则调用watcher对象的addDep方法，该方法接收的参数为Dep实例, notify方法
首先拷贝一份subs对象数组，然后遍历该数组，依次调用数组中watcher对象的update方法，另外还暴露了两个pushTarget
和popTarget方法，作用是往栈里添加和删除watcher对象


