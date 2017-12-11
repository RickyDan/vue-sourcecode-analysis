```js
/* @flow */
// Convert a value to a string that is actually rendered
// 将传入的参数转化成字符串或者JSON字符串
export function _toString (val: any): string {
  return val === null
    ? ''
    : typeof val === 'object'
     ? JSON.stringify(val, null, 2)
     : String(val)
}

// Convert a input value to a number for persistence
// If the conversion fails, return original 
// 将传入的参数转化成数字，如果不能转化成数字，则返回字符串本身，否则返回字符串
export function toNumber (val: string): number | string {
  const n = parseFloat(val)
  return isNaN(n) ? val : n
}

/* Make a map and return a function for checking if a key is in that map
将传入的字符串参数转化成一个 Map 结构，key 值为字符串中的每个单词，值为 true 
makeMap 返回的函数与传入的第二个参数有关， 返回的函数都是接受一个参数字符串，查看
该字符串是否存在 Map 结构中，存在则返回 true 值， 否则返回 false。区别在于makeMap
如果为true, 则将返回函数中的参数先转化成小写字母的形式再传入 Map 中 */
export function makeMap (
  str: string,
  expectsLowerCase? : boolean
): (key: string) => true | void {
  const map = Object.create(null)
  const list: Array<string> = str.split(',')
  for  (let i = 0; i < list.length; i++) {
    map[list[i]] = true
  }
  return expectsLowerCase
    ? val => map[val.toLowerCase()]
    : val => map[val]
}
// 内置的 tag 标签有 slot, component
export const isBuiltInTag = makeMap('slot, component', true)

// Remove an item from an array
// 从数组中移除一个给定的元素
export function remove (arr: Array<any>, item: any): Array<any> | void {
  if (arr.length) {
    const index = arr.indexOf(item)
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}

// Check whether the object has the property
const hasOwnProperty = Object.prototype.hasOwnProperty
/* 检测一个属性是对象本身的属性还是从原型中继承而来的属性， 接受一个对象
和一个属性值为参数 */
export function hasOwn (obj: Object, key: string): boolean {
  return hasOwnProperty.call(obj, key)
}

// Check if value is primitive
// 检查一个值是否是字符串或者数字类型
export function isPrimitive (value: any): boolean {
  return typeof value === 'string' || typeof value === 'number'
}

// Create a cached version of a pure function
// 缓存一个纯函数
export function cached<F: Function> (fn: F): F {
  const cache = Object.create(null)
  return (function cachedFN (str: string) {
    const hit = cache[str]
    return hit || (cache[str] = fn(str))
  }: any)
}

// Camelize a hyphen-delimited string
// 将一个连字符形式的字符串转化成驼峰形式的字符串
const camelizeRE = /-(\w)/g
export const camelize = cache((str: string): string => {
  return str.replace(camelizeRE, (_, c) => c ? c.toUpperCase() : '')
})

// Capitalize a string
// 将变量首字母转化成大写形式
export const capitalize = cached((str: string): string => {
  return str.charAt(0).toUpperCase() + str.slice(1)
})

// Hyphenate a cameCase string
const hyphenate = /([^-])(A-Z)/g
export const hyphenate = cached((str: string): string => {
  return str
    .replace(hyphenateRE, '$1-$2')
    .replace(hyphenateRE, '$1-$2')
    .toLowerCase()
})

// Simple bind, faster than native
export function bind (fn: Function, ctx: Object): Function {
  function boundFn (a) {
    const l: number = arguments.length
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }
}

// Convert an Array_like object to a real Array
export function toArray (list: any, start?: number): Array<any> {
  start = start || 0
  let i = list.length - start
  const ret: Array<any> = new Array(i)
  while (i--) {
    ret[i] = list[i + start]
  }
  return ret
}

// Mix properties into target object
export function extend (to: Object, _form: ?Object) {
  for (const key in _form) {
    to[key] = _form[key]
  }
  return to
}

/*
  Quick object check - this is primarily used to tell
  Object from primitive values when we know the value
  is a JSON-compliant type
*/
export function isObject (obj: mixed): boolean {
  return obj !== null && typeof obj === 'object'
}

// Strict object type check. Only returns true
// for  plain JavaScript objects
const toString = Object.propotype.toString
const OBJECT_STRING = '[object Object]'
export function isPlainObject (obj: any): boolean {
  return toString.call(obj) === OBJECT_STRING
}

// Merge an Array of Objects into a single Object
export function toObject (arr: Array<any>): Object {
  const res = {}
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }
  return res
}

// Perform no operation
export function noop () {}

// Always return false
export const no = () => false

// Return same value
export const identify = (_: any) => _

// Generate a static keys string from compiler modules
export function genStaticKeys (modules: Array<ModuleOptions>): string {
  return modules.reduce((keys, m) => {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}

export function looseEqual (a: mixed, b: mixed): boolean {
  const isObjectA = isObject(a)
  const isObjectB = isObject(b)

  if (isObject && isObjectB) {
    return JSON.stringify(a) === JSON.stringify(b)
  } else if (!isObjectA && isObjectB) {
    return String(a) === String(b) 
  } else {
    return false
  }
}

export function looseIndexOf (arr: Array<mixed>, val: mixed): number {
  for (let i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val)) return i
  }
  return -1
}

// Ensure a function is called only once
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

