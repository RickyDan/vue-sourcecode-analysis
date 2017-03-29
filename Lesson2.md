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

import { no, noop, identity } from 'shared/util'
export type Config = {
  // user
  optionMergeStrategies: { [key: string]: Function };
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
