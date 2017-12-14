
```
└── core
  └── observer
    └── scheduler.js
```

```js
/* flow */
import type Watcher from './watcher'
import config from '../config'
import { callHook } from '../instance/lifecycle'
import {
  warn,
  nextTick,
  devtools
} from '../util/index'

const queue: Array<Watcher> = []
let has: { [key: number]: ?true } = {}
let circular: { [key: number]: number } = {}
let waiting = false
let flushing = false
let index = 0

// 重置调度器的状态
function resetSchedulerState () {
  queue.length = 0
  has = {}
  if (process.env.NODE_ENV !== 'production') {
    // 循环初始值
    circular = {}
  }
  waiting = flushing = false
}

// 清除队列并启动 watchers
// flushSchedulerQueue 函数会依次从watcher queue 里取出watcher对象，并执行run方法
function flushSchedulerQueue () {
  flushing = true
  let watcher, id, vm
  // 对队列进行排序，id 属性值小的排在队列前面
  queue.sort((a, b) => a.id - b.id)

  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    id = watcher.id
    has[id] = null
    watcher.run()
    // 在开发环境中，检查并停止无限循环更新
    if (process.env.NODE_ENV !== 'production' && has[id] !== null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > config._maxUpdateCount) {
        warn(
          'You have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }
  // 调用更新数据的钩子函数
  index = queue.length
  while (index--) {
    watcher = queue[index]
    vm = watcher.vm
    if (vm._watcher === watcher && vm._isMounted) {
      callHook(vm, 'updated')
    }
  }

  if (devtools && config.devtools) {
    devtools.emit('flush')
  }

  resetSchedulerState()
}

/* 添加一个 watcher 对象到 watcher 队列中

*/

export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // queu the flush
    if (!waiting) {
      waiting = true
      nextTick(flushSchedulerQueue)
    }
  }
}
```
