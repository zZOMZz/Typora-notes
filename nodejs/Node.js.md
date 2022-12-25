[toc]

# Node.js

https://www.simform.com/blog/what-is-node-js/

## 一. 定义

- Node.js是一个基于V8 Javascript运行时环境
  - 无论是浏览器(chrome)还是Node.js事实上都是嵌入了V8引擎来执行JavaScript代码
- 在**浏览器中**, 需要解析,渲染HTML,CSS等相关渲染引擎, 另外还需要提供支持浏览器操作的API, 浏览器自己的事件循环
- 在**Node.js**中, 需要能进行一些额外的操作,如: **文件系统读/写 , 网络IO, 加密, 压缩解压文件**



- libuv是使用C语言编写的库
- libuv提供了事件循环,文件读写, 网络IO, 线程池等等内容



- 可以在Node的process.argv中找到文件路径和当前运行环境
- argc: arguments counter 传递参数的个数
- argv: arguments vector 传入的具体参数

## 二. Node.js的引用场景

- 目前前端开发的库都是以node包的形式进行管理
- npm, yarn, pnpm工具成为前端开发使用最多的工具
- 使用Node.js作为web服务器开发,中间件, 代理服务器
- 大量项目需要借助Node.js完成前后端渲染的同构应用
- 资深的前端工程师需要为项目编写脚本工具, 用node运行脚本
- 在Node环境下使用Electron来开发桌面应用程序



## 三. Node版本工具

- 同一台电脑上存在多个版本的node
- 可以使用n / nvm 来管理版本, 不支持windows(使用nvm-windows)
- nvm
  - nvm install x.x.x
  - nvm use ...
  - `nvm list` 列出所有的版本
- n(不在windows上使用)
  - 可以直接用npm安装 `npm install -g n`
  - `n lts` 安装最新的lts版本
  - `n latest` 安装最新的版本
  - `n`查看所有的版本





## 四. Node的全局对象

- 浏览器和node中都有globalThis指向全局对象

- Node中使用的不是window, 而是global
- 全局对象有许多
  - module
  - exports
  - require()
  - __dirname: 当前文件所在的目录结构
  - __filename: 当前目录+文件名称

在node中使用var定义的变量不会加到window里



## 五. 模块化

### 1. CommonJs

- 导出: exports, exports是一个对象, 可以在这个对象上添加属性, 添加的属性会导出, 在内存中拿到的是同一个对象(引用赋值)

```js
export.name = "zzz"
```

- 导入: require, require是一个函数, 默认导入module.exports对应的对象

```js
const { name } = require(filepath)
console.log(name)
```

- 导出(module.exports) module.exports指向的对象和exports指向的对象是同一个

```js
// 会被默认导入的对象, 这种写法会新建一个对象, 因此导入的也是这个新对象, 利用exports改变的只是老对象
module.exports = {
    
}
```

- 模块被导入时, 模块内的js代码会被执行一次
  - 多次引用只会加载一次 (module.loaded = true)
- Node在引用时采用的是**深度优先算法**

- CommonJS加载模块是同步的

  - 只有等到对应的模块加载完毕, 当前模块中的内容才能被运行
  - 服务器中的js文件大多都是本地文件, 因此加载速度比较快

  - 在浏览器端加载js文件需要向下载, 如果下载没有完成, 则后续的js代码无法正常运行, 所以在浏览器中一般不使用CommonJS(在webPack中除外)



### 2. ES Module

[ES modules: A cartoon deep-dive - Mozilla Hacks - the Web developer blog](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)

- 导入 import
- 导出 export
  - export default 默认将整个文件导出, 且导入时不需要 { } 来解构

ES Module的解析三阶段:

1. 构建: 根据地址查找js文件, 并且下载, 将其解析为模块记录
2. 实例化: 对模块记录进行实例化, 并且分配内存空间, 解析模块的导入和导出语句,将导出的变量放入环境记录里, 并把模块指向对应的内存地址
3. 运行: 运行代码, 计算值, 并且将值填充到内存地址中