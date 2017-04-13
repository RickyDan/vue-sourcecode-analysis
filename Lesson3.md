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
