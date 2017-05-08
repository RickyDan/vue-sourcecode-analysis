# 使用window.performance 属性测试 Web 应用的性能

Vue2.0 封装了一个用于测试网页性能的模块, 该模块位于util目录下的 perf.js文件

```
└── core
  └── util
    └── perf.js
```

perf.js的源码如下

```js
import { inBrowser } from './env'

export let mark
export let measure

if (process.env.NODE_ENV !== 'production') {
  const perf = inBrowser && window.performance
  /* istanbul ignore if */
  if (
    perf &&
    perf.mark &&
    perf.measure &&
    perf.clearMarks &&
    perf.clearMeasures
  ) {
    mark = tag => perf.mark(tag)
    measure = (name, startTag, endTag) => {
      perf.measure(name, startTag, endTag)
      perf.clearMarks(startTag)
      perf.clearMarks(endTag)
      perf.clearMeasures(name)
    }
  }
}
```

window对象有一个performance属性，performance属性是一个对象，performance.mark方法接收一个字符串参数, mark方法是标注的意思，调用 mark方法会创建一个时间戳，类似于在地图上标注一个位置, 通过测算两个点之间的距离就能算出两个地理位置的实际距离是多少。比如js 函数执行前使用mark方法做一个标注，执行完之后做一个标注，通过计算两个两个时间戳的差值可以算出该函数的运行时间，性能越高，执行时间越少。 measure 方法就是用于测算两个标注之间的时间差值。关于 window.performance 属性的有关 API 可以自行查阅 MDN的文档
   