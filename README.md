# vue-sourcecode-analysis
## 基于Vue2.0的源码解读

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
首先基于shared目录讲解
share 目录的结构如下：
```
  └── shared
     └── util.js
```
util.js 里定义和重写了一些基础方法
```js
 /* @flow */
/**
 * Convert a value to a string that is actually rendered.
 */
export function _toString (val: any): string {
  return val == null
    ? ''
    : typeof val === 'object'
      ? JSON.stringify(val, null, 2)
      : String(val)
}
```
 _toString方法接受一个任意类型的参数并返回一个string类型，如果参数是 null 则返回空字符串, 如果是参数是对象则使用JSON.stringify()方法将其序例化输出，
 其他类型则直接使用String构造函数将其转化成字符串
####  /* @flow */ 该注释是每个vue源文件的都有声明的，vue2.0是使用fackbook开源的一个js类型检查工具flow作为数据类型检查的，所以vue源码里会有非常多类似于
```js
  epxort function _toString (val: any): string {}
  export function toNumber (val: string): number | string {}
```
 _toString函数表示接受一个任意数据类型的参数并返回一个string，  _toNumber函数表示接收一个string类型的参数并返回一个number或者string
 详细文档可以自行查阅 docs https://flowtype.org/
 
