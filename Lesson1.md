# vue-sourcecode-analysis
## 基于Vue2.2.5的源码解读

### 目录架构
```
├── src                      // 源码目录
│   ├── compiler             // 存放指令和解析html的模版引擎
│   ├── core                 // vue核心代码实现
│   ├── entries              // 根据不同平台提供对应的入口文件
│   ├── platforms            // web平台和weex平台对应的vue实现 
│   ├── server               // 存放服务端渲染相关文件的目录
│   ├── sfc                  // 存放解析组件中的内容的文件
│   └── shared               // 存放一些公共方法的文件
```

vue2.0的源码目录组织结构如上所示
首先分析shared目录下的模块：
```
  └── shared
     └── util.js
     └── constants.js
```
util.js 里定义和重写了一些基础方法
```js
 /* @flow */
export function _toString (val: any): string {
  return val === null
    ? ''
    : typeof val === 'object'
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```
 _toString方法接受一个任意类型的参数并返回一个string类型，如果参数是 null 则返回空字符串, 如果是参数是对象则使用JSON.stringify()方法将其序例化输出，
 其他类型则直接使用String构造函数将其转化成字符串

####  /* @flow */ 该注释是每个vue源文件的都有声明的，vue2.0是使用fackbook开源的一个js类型检查工具flow作为数据类型检查的，所以vue源码里会有非常多类似于下面这种代码
```js
  epxort function _toString (val: any): string {}
  export function toNumber (val: string): number | string {}
```
 _toString函数表示接受一个任意数据类型的参数并返回一个string，  _toNumber函数表示接收一个string类型的参数并返回一个number或者string，
 详细文档可以自行查阅 https://flowtype.org/

 ```js
export function toNumber (val: string): number | string {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}
```
toNumber方法接收一个字符串并将其转化成number并返回转化后的数字，如果不能转换，则返回字符串本身


```js
export function makeMap (
  str: string,
  expectsLowerCase?: boolean
): (key: string) => true | void {
  const map = Object.create(null)
  const list: Array<string> = str.split(',')
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}
```
makeMap方法接收两个参数，第一个参数是字符串, 第二个参数是布尔值(用于控制key值是大写还是小写的形式传入)，返回值是一个函数，该函数的作用是用来检测传入的key值是否存在于map对象中

```js
export const isBuiltInTag = makeMap('slot,component', true)
```
makeMap返回一个匿名函数，所以isBuiltInTag是一个检测某个标签名是否是内置标签

```js
/**
 * Remove an item from an array
 */
export function remove (arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```
remove方法很简单，接收两个参数，第一个参数是数组，第二个是任意值，返回值可能是数组或者是undefined, 如果传入的item是数组中的元素，则将item从该数组中删除

```js
const hasOwnProperty = Object.prototype.hasOwnProperty
export function hasOwn (obj: Object, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}
```
 这个hasOwn方法用于检查对象上的属性是对象本身的还是继承自原型的
 
 ```js
 export function isPrimitive (value: any): boolean {
  return typeof value === 'string' || typeof value === 'number'
}
```
isPrimitive方法是检查一个值是否是数字还是字符串

```js
export function cached<F: Function> (fn: F): F {
  const cache = Object.create(null)
  return (function cachedFn (str: string) {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }: any)
}
```
cache方法作用是缓存一个纯函数的执行结果，该方法接收一个函数(纯函数)作为参数，并返回一个匿名函数，该匿名函数缓存了函数参数的执行结果。纯函数是指只要输入相同输出就一定相同，没有任何副作用（改变外界状态）的函数。纯函数是函数式编程的一个重要概念。有关纯函数和函数式编程的相关概念可以自行google。Redux里的reducer就是函数式编程的极致体现。

```js
/**
 * Camelize a hyphen-delimited string.
 */
const camelizeRE = /-(\w)/g
export const camelize = cached((str: string): string => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})
```
camelize方法是将"learn-vue"这种类型的字符串转化成驼峰类型的字符串"learnVue"

```js
/**
 * Capitalize a string.
 */
export const capitalize = cached((str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})
```
capitalize方法是将字符串的首字母转化成大写的形式

```js
/**
 * Hyphenate a camelCase string.
 */
const hyphenateRE = /([^-])([A-Z])/g
export const hyphenate = cached((str: string): string => {
  return str
    .replace(hyphenateRE, '$1-$2')
    .replace(hyphenateRE, '$1-$2')
    .toLowerCase()
})
```
hyphenate方法用于将驼峰类型的字符串转化成连字符类型的字符串并将大写形式转化成小写

```js
/**
 * Simple bind, faster than native
 */
export function bind (fn: Function, ctx: Object): Function {
  function boundFn (a) {
    const l: number = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }
  // record original fn length
  boundFn._length = fn.length
  return boundFn
}
```
bind方法重写了原生js上的bind方法，用法与原生bind方法一样

```js
/**
 * Convert an Array-like object to a real Array.
 */
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}
```
toArray方法是将类数组对象转化成真正的数组

```js
/**
 * Mix properties into target object.
 */
export function extend (to: Object, _from: ?Object): Object {
  for (const key in _from) {
    to[key] = _from[key]
  }
  return to
}
```
extend方法是扩展对象属性的一个方法

```js
export function isObject (obj: mixed): boolean {
  return obj !== null && typeof obj === 'object'
}
```
isObject用于判断一个复合数据类型是对象、数组还是其他，如果是不为null并且是Object、Array类型，则返回true, 否则返回false

```js
const toString = Object.prototype.toString
const OBJECT_STRING = '[object Object]'
export function isPlainObject (obj: any): boolean {
  return toString.call(obj) === OBJECT_STRING
}
```
isPlainObject方法用于判断一个任意类型的数值是否是Object对象
```js
/**
 * Merge an Array of Objects into a single Object.
 */
export function toObject (arr: Array<any>): Object {
  const res = {}
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}
```
toObject方法会遍历对象数组里的每一个对象，将对象里的属性添加进一个新对象，并返回该对象

```js
export function noop() {}
export const no = () => false
export const identity = (_ : any) => _
``` 
noop暴露了一个没有任何操作的空方法, no方法永远返回false, identity仅仅将第一个参数返回

```js
/**
 * Generate a static keys string from compiler modules.
 */
export function genStaticKeys (modules: Array<ModuleOptions>): string {
  return modules.reduce((keys, m) => {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```
genStaticKeys方法是返回一个static keys 字符串组成的key值

```js
/**
 * Check if two values are loosely equal - that is,
 * if they are plain objects, do they have the same shape?
 */
export function looseEqual (a: mixed, b: mixed): boolean {
  const isObjectA = isObject(a)
  const isObjectB = isObject(b)
  if (isObjectA && isObjectB) {
    try {
      // 不能直接比较两个对象 a === b, 因为两个对象具有独立的内存地址，所以即使两个对象属性完全一致，也不会想等，
      // 这里是将两个对象JSON序列化之后再比较两个字符串是否相等
      return JSON.stringify(a) === JSON.stringify(b)
    } catch (e) {
      // possible circular reference
      return a === b
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b)
  } else {
    return false
  }
}
```
looseEqual方法用于比较两个值是否相等，如果相等返回true，否则返回false

```js
export function looseIndexOf (arr: Array<mixed>, val: mixed): number {
  for (let i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) return i
  }
  return -1
}
```
looseIndexOf方法的作用是给定一个数组和一个特定的值，如果数组中元素与该值相等，则返回该元素对应的下标，若数组中不存在给定的值，则返回-1

```js
/**
 * Ensure a function is called only once.
 */
export function once (fn: Function): Function {
  let called = false
  return () => {
    if (!called) {
      called = true
      fn()
    }
  }
}
```
once方法确保一个方法只被调用一次，不能多次重复调用

**如果您觉得还不错，点个star, 激励我写出更高质量的教程。也欢迎给我提PR**
