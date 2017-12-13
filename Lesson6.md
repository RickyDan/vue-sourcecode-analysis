# Vue props 数据是怎么传递到子组件的

```
└── core
  └── util
    └── props.js
```


```js
import { hasOwn, isObject, isPlainObject, capitalize, hyphenate } from 'shared/util'
import { observe, observerState } from '../observer/index'
import { warn } from './debug'
```


```js
type PropOptions = {
  type: Function | Array<Function> | null,
  default: any,
  required: ?boolean,
  validator: ?Function
}

// 获取函数名的函数
function getType (fn) {
  const match = fn && fn.toString().match(/^\s*function (\w+)/)
  return match && match[1]
}

// 判断两个函数是否同名
function isType (type, fn) {
  if (!Array.isArray(fn)) {
    return getType(fn) === getType(type)
  }
  for (let i = 0, len = fn.length; i < len; i++) {
    if (getType(fn[i]) === getType(type)) {
      return true
    }
  }
  return false
}

export function validateProp (
  key: string,
  propOptions: Object,
  propsData: Object,
  vm?: Component
): any {
  const prop = propOptions[key]
  // 传入的key是否存在于propsData中
  const absent = !hasOwn(propsData, key)
  let value = propsData[key]
  // 如果传入的属性的Boolean类型
  if (isType(Boolean, prop.type)) {
    // 如果key不在 propsData 中并且没有设置default值，则value设置为false
    if (absent && !hasOwn(prop, 'default')) {
      value = false
    // 如果传入的属性不为String类型并且value为空或者入传入的key(将key转化成小写的连字符形式 a-b)相等，则value值设为true
    } else if (!isType(String, prop.type) && (value === '' || value === hyphenate(key))) {
      value = true
    }
  }
  // check default value
  // 如果 value 值不存在，则调用getPropDefaultValue方法去获取默认的值，把value传给observer方法，之后value值的变更就可以被Vue追踪到
  if (value === undefined) {
    value = getPropDefaultValue(vm, prop, key)
    // since the default value is a fresh copy
    // make sure to observe it
    const prevShouldConvert = observerState.shouldConvert
    observerState.shouldConvert = true
    observe(value)
    observerState.shouConvert = prevShouldConvert
  }
  // 在非生产环境中会去验证父组件传入的prop是否有效
  if (process.env.NODE_ENV !== 'production') {
    assertProp(prop, key, value, vm, absent)
  }
  return value
}

// get the default value of prop
// 获取 props 中默认的 default 值
function getPropDefaultValue (vm: ?Component, prop: PropOptions, key: string): any {
  // no default, return undefined
  // 如果 prop没有default 值，直接return undefined
  if (!hasOwn(prop, 'default')) {
    return undefined
  }
  const def = prop.default
  // warn against non-facotry defaults for Object & Array
  // 在非生产环境中，如果key是一个对象或者数组，直接报warning
  if (process.env.NODE_ENV !== 'production' && isObject(key)) {
    warn(
      'Invalid default value for prop "' + key + '": ' +
      'Props with type Object/Array must use a factory function ' +
      'to return the default value.',
      vm
    )
  }
  // the raw prop value was also undefined from previous render
  // return previous default value to avoid unnecessary watcher trigger
  if (vm && vm.$options.propsData && vm.$options.propsData[key] === undefined && vm._props[key] !== undefined) {
    return vm._props[key]
  }
  // 如果 default 是一个方法并且type类型声明不是Function类型，则调用def去求值，否则返回 def 默认值
  return typeof def === 'function' && getType(prop.type) !== 'Function' ? def.call(vm) : def
}

// Assert whether a prop is valid
// 该方法主要用于验证传入的prop是否有效
function assertProp (
  prop: PropOptions,
  name: string,
  value: any,
  vm: ?Component,
  absent: boolean
) {
  // 如果必填属性没有传入，则报warning
  if (prop.required && absent) {
    warn(
      'Missing required prop: "' + name + '"',
      vm
    )
    return
  }
  // 如果值为空并且是非必填的属性，直接return
  if (value === null && !prop.required) {
    return
  }
  let type = prop.type
  // 如果声明了传入的数据类型，则 valid 等于 false, 否则返回 true
  let valid = !type || type === true
  const expectedTypes = []
  // 如果传入的数据声明了数据类型
  if (type) {
    // 如果传入的数据类型是非数组，则将其转换成数组类型
    if (!Array.isArray(type)) {
      type = [type]
    }
    // 遍历类型数组
    for (let i = 0; i < type.length && !valid; i++) {
      // 获取 type 的类型描述
      const assertedType = assertType(value, type[i])
      // 将类型的字符串说明添加进expectedTypes数组中
      expectedTypes.push(assertedType.expectedType || '')
      valid = assertedType.valid
    }
  }
  /* 如果valid值为false, 则说明从父组件传入的数据类型与子组件定义的类型不符合,
  浏览器控制台上就可以看到是哪个数据字段的类型传错了
  */
  if (!valid) {
    warn(
      'Invalid prop: type check failed for prop "' + name + '".' +
      ' Expected ' + expectedTypes.map(capitalize).join(', ') +
      ', got ' + Object.prototype.toString.call(value).slice(8, -1) + '.',
      vm
    )
    return
  }
  const validator = prop.validator
  if (validator) {
    // prop中是可以自定义验证函数的，如果传入的值不符合验证函数的规则，调用warn方法输出警告信息
    if (!validator(value)) {
      warn(
        'Invalid prop: custom validator check failed for prop "' + name '".',
        vm
      )
    }
  }
}

/* 
接收一个值和内置的构造类型函数 (String, Boolean, Number, Array等)
这一段代码就是用来检查传入的 prop的值的类型是否与组件期待的类型一致，如果
子组件期待的是一个string, 但是从父组件传入的是一个number或者其他类型的值，
则返回的对象中valid属性为false, 该属性为false会触发warn方法提示警告，在浏览器
的控制台就可以看到输出对应的信息
*/
function assertType (value: any, type: Function): {
  valid: boolean,
  expectedType: ?string
} {
  let valid
  // 返回 type 构造函数的名称
  let expectedType = getType(type)
  // 根据不同的类型去对 value 进行 typeof 求值，返回的valid是一个布尔值
  if (expectedType === 'String') {
    valid = typeof value === (expectedType = 'string')
  } else if (expectedType === 'Number') {
    valid = typeof value === (expectedType = 'boolean')
  } else if (expectedType === 'Function') {
    valid = typeof value === (expectedType = 'function')
  } else if (expectedType === 'Object') {
    valid = isPlainObject(value)
  } else if (expectedType === 'Array') {
    valid = Array.isArray(value)
  } else {
    valid = value instanceof type
  }
  // 最终返回的匿名对象包含valid和expectedType这两个字段
  return {
    valid,
    expectedType
  }
}
```
