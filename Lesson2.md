## Vue2.0 核心功能的实现
从这里开始研究Vue2.0核心功能是如何实现的，包括最核心的MVVM，组件的状态管理，生命周期钩子函数等核心功能

### core目录的结构如下
```
  └── core
     └──components          components目录主要实现了组件的keep-alive功能
     └──global-api          Vue里一些全局API接口的实现
     └──instance            实现Vue组件化编程的核心关键
     └──observer            MVVM模式实现的关键 主要运用了观察者模式
     └──util                该目录主要存放了一些通用的功能
     └──vdom                虚拟DOM的实现
     └──config.js           基础配置文件
     └──index.js .          将整个core实现的功能模块暴露出去
```
先来看看基础配置文件 config.js
```js
/* @flow */
// 引入 no, noop, identity 这几个方法
import { no, noop, identity } from 'shared/util'
export type Config = {
  // user
  optionMergeStrategies: { [key: string]: Function }; // 
  silent: boolean;
  productionTip: boolean;
  performance: boolean;
  devtools: boolean;
  errorHandler: ?(err: Error, vm: Component, info: string) => void;
  ignoredElements: Array<string>;
  keyCodes: { [key: string]: number | Array<number> };
  // platform
  isReservedTag: (x?: string) => boolean;
  parsePlatformTagName: (x: string) => string;
  isUnknownElement: (x?: string) => boolean;
  getTagNamespace: (x?: string) => string | void;
  mustUseProp: (tag: string, type: ?string, name: string) => boolean;
  // internal
  _assetTypes: Array<string>;
  _lifecycleHooks: Array<string>;
  _maxUpdateCount: number;
};
```

flow 不仅可以检测原生js的数据类型，还支持自定义类型，比如 type Config就是一个自定义类型，下面来看Config类型的实现, 可以看到config必须是Config类型的，这就保证了暴露出去的config配置项不会被随意更改。 在config对应的属性注释里我加上的中文注释，应该不难看懂

```js
const config: Config = {
  /**
   * Option merge strategies (used in core/util/options)
   * optionMergeStrategies 属性是一个对象，其key是一个字符串，value值是一个function
  */
  optionMergeStrategies: Object.create(null),

  /**
   * Whether to suppress warnings. 
   *  silent 属性主要用于控制warnings是否显示
   */
  silent: false,

  /**
   * Show production mode tip message on boot?
   *  在生产环境下启动时是否报出提示信息
   */
  productionTip: process.env.NODE_ENV !== 'production',

  /**
   * Whether to enable devtools
   * 在开发环境下是否使用调试工具
   */
  devtools: process.env.NODE_ENV !== 'production',
  
  /**
   * Whether to record perf
   * 与性能测试有关的一个选项
   */
  performance: false,

  /**
   * Error handler for watcher errors
   * 错误处理捕获
   */
  errorHandler: null,

  /**
   * Ignore certain custom elements
   * 存放忽略掉的元素类型的数组
   * Ignore certain custom elements
   */
  ignoredElements: [],

  /**
   * Custom user key aliases for v-on
   * 存放键盘对应的映射值
   */
  keyCodes: Object.create(null),

  /**
   * Check if a tag is reserved so that it cannot be registered as a
   * component. This is platform-dependent and may be overwritten.
   * isReservedTag主要是查看一个组件的名字是否是保留字，如果是则无法注册一个组件
   */
  isReservedTag: no,

  /**
   * Check if a tag is an unknown element.
   * Platform-dependent.
   * 检查一个标签是否是未知的元素
   */
  isUnknownElement: no,

  /**
   * Get the namespace of an element
   * 元素的命名空间
  */
  getTagNamespace: noop,

  /**
   * Parse the real tag name for the specific platform.
   * 在特定的平台下解析一个标签的名称
   */
  parsePlatformTagName: identity,

  /**
   * Check if an attribute must be bound using property, e.g. value
   * Platform-dependent.
   * 元素的attributes必须以属性的方式调用
   */
  mustUseProp: no,

  /**
   * List of asset types that a component can own.
   * 一个组件可用的属性
   */
  _assetTypes: [
    'component',
    'directive',
    'filter'
  ],

  /**
   * List of lifecycle hooks.
   * 生命周期钩子函数的名称数组
   */
  _lifecycleHooks: [
    'beforeCreate',
    'created',
    'beforeMount',
    'mounted',
    'beforeUpdate',
    'updated',
    'beforeDestroy',
    'destroyed',
    'activated',
    'deactivated'
  ],

  /**
   * Max circular updates allowed in a scheduler flush cycle.
   * 一个生命周期内允许的最大更新数
  */
  _maxUpdateCount: 100
}

export default config
```
