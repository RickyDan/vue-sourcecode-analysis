```
└── core
  └── observer
    └── dep.js
```

```js
import type Watcher from './watcher'
import { remove } from '../util/index'

let uid = 0

// 通知类
export default class Dep {
  // 静态属性 target 指向 watcher 对象
  static target: ? Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  // 将 watcher 对象添加进栈
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  // 将 watcher 对象移除出栈
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  /* 当数据发生变更后，通知 Vue 去重新 render 视图的方法
  通过遍历 watcher 数组，循环调用数组中 watcher 对象的 update 方法
  */
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


