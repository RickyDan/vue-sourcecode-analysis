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
  
  
