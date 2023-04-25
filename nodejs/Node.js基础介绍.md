[toc]

# Node.js

https://www.simform.com/blog/what-is-node-js/

## 一. 定义

![Node.js的依赖](.\图片\Node.js的依赖.png)

### 1. V8

- ==Node.js不是一个框架, Node.js是一个基于V8 Javascript运行时环境==
- **它使用事件驱动、非阻塞 I/O 模型，使其轻量级、更高效，非常适合跨共享设备运行的数据密集型实时应用程序, Node.js 是高度可定制和可扩展技术的缩影**
  - 无论是浏览器(chrome)还是Node.js事实上都是嵌入了V8引擎来执行JavaScript代码
- 在**浏览器中**, 需要解析,渲染HTML,CSS等相关渲染引擎, 另外还需要提供支持浏览器操作的API, 浏览器自己的事件循环
- 在**Node.js**中, 需要能进行一些额外的操作,如: **文件系统读/写 , 网络IO, 加密, 压缩解压文件**



### 2. libuv

- 使node可以操纵底层的计算机操纵系统的开源库

- libuv是使用C++语言编写的开源库, 重点关注与异步IO的处理
- libuv提供了事件循环,文件读写, 网络IO, 线程池等等内容
- **最重要的是提供了事件循环(Event Loop)和线程池(thread pool)**



- 可以在Node的process.argv中找到文件路径和当前运行环境
- argc: arguments counter 传递参数的个数
- argv: arguments vector 传入的具体参数

### 3. 其他模块

- 下面的一些别的模块用于http, DNS, 压缩等

## 二. Node.js的特点

### 1. 异步/非阻塞线程执行

- **Node.js库中的每个API(非第三方库)都是非阻塞的, 当在等待执行链外的某个响应时, 下一个任务不会被阻塞, 会继续执行**
- Node.js 中开发 Web 应用程序可以实现稳定和安全的非阻塞 I/O 模型，简化代码 **基于事件的异步方式**, 使用**单线程事件循环**

![node-js architecture](.\图片\node-js architecture.png)

#### 1. 线程池

- 过于繁重的任务会被Event Loop卸载放入线程池进行执行, 避免阻塞单线程, 如
  - 压缩
  - 与密码相关的, 如缓存密码
  - 文件调用的一些API
  - DNS查找
- 线程池中的线程数量是可以配置的, 默认4个, 最多128个

```js
// 修改线程池大小
process.env.UV_THREADPOOL_SIZE = 1
```



#### 2. 事件循环(Event Loop)

- 所有非顶层定义的代码, 即在callback function中定义的代码(除去被卸载到线程池中的代码), 其余都是在Event Loop中执行的

![单线程运行过程](.\图片\单线程运行过程.png)



##### 2.1 收集事件(event)

- Timer expired, New Http request, Finish file reading这些操作完成后都会发出事件(event), 事件循环就会收集这些事件, 将他们传入它们对应的**callback queue**, 待循环进行到合适的阶段(phase), 就会出相应的响应(调用callback function)

![Event Loop](.\图片\Event Loop.png)

##### 2.2 执行事件

![Event Loop阶段](.\图片\Event Loop阶段.png)



###### 1. Expired timer callbacks

- 第一个阶段是执行过期的timer中的callback function



###### 2. I/O polling and callbacks

- 主要指网络和文件访问相关的操作
- 主要是在这个阶段执行
- **当回调序列为空时, Event loop在这个阶段等待执行, 因此这个可能是第一个被执行的阶段**



###### 3. setImmediate callbacks

- setImmediate 是一种特殊的定时器. 如果我们想在I/O执行完后立即处理回调



###### 4. close callbacks

- 所有和close相关的事件回调都会在这个阶段被执行





**nextTick queue 和 othermicroTasks queue中的callback会在当前的阶段执行完成后立即执行, 而不是错过了只能等待循环一轮** 

###### 5. nextTick queue 

```js
process.nextTick(() => { console.log("nextTick") })
```





###### 6. othermicroTasks queue

- 主要用于处理promise



##### 2.3 由单线程引发的编程规范

- 不要使用同步版本(sync)的函数在一些如`fs`,`crypto(密码)`,`zilb(压缩)`等库的API在callback function中, 在顶层代码中使用没关系, 因为它会在Event Loop开启前被执行, **同步的函数将不会被放入到Event Loop中, 它们甚至会阻塞其他的异步函数, 即使那些异步函数是立即执行的**

```js
// 顶层代码
fs.readFile(...,() => {
    setTimer(() => {}, 0)
    setImmediateTimer(() => {}, 0)
    
    // 密码加密, 繁重的任务同步
    crypto.pbkdf2Sync(.....)
    
    // 上面的所有异步代码只有等待同步代码执行完成后才会被执行
    // 但是计时器还是会正常计时, 如果完成的时间超过了计时时间, 会在同步代码执行完后立即执行
})
```



- 不要进行复杂的计算
- 小心对待大型JSON对象
- 不要使用过于复杂的正则表达式



### 2. 事件驱动(Event-driven)

- **使用Node.js构建的服务器, 利用一种称为'Events(事件)'的通知机制, 来接收和跟踪之前API请求的响应, 事件循环允许Node.js去执行所有的非阻塞操作**



![Event driven](C:\Users\zZOMZz\Desktop\Typora笔记\nodejs\图片\Event driven.png)



### 3. 跨平台兼容性(Cross-platform compatibility)

- **Node.js在多平台三如Windows, Linux, Mac OS, Unix, 和手机移动端都是兼容的**



## 三. Node.js的优势

### 1. 可扩展性(Scalability)

- 你可以通过垂直扩展可以让您向当前节点添加更多资源，而水平扩展可以让您更快地添加新节点, Node.js 应用程序在整个开发过程中不需要大块(a large block)，因为它与一组微服务(microservices)和模块(module)一起工作。它简单易行，非常适合寻求发展的初创公司



### 2. 高性能(High performance)

- 内置V8引擎
- 单线程
- 在同时处理多个请求方面非常高效

## 四. Node.js的应用场景

- 目前前端开发的库都是以node包的形式进行管理
- npm, yarn, pnpm工具成为前端开发使用最多的工具
- 使用Node.js作为web服务器开发,中间件, 代理服务器
- 大量项目需要借助Node.js完成前后端渲染的同构应用
- 资深的前端工程师需要为项目编写脚本工具, 用node运行脚本
- 在Node环境下使用Electron来开发桌面应用程序



- 社交媒体网络的后端
- 单页面富应用
- 聊天应用
- 数据流
- 物联网应用





## 五. Node的全局对象

- 浏览器和node中都有globalThis指向全局对象

- Node中使用的不是window, 而是global
- 全局对象有许多
  - module
  - exports
  - require()
  - __dirname: 当前文件所在的目录结构
  - __filename: 当前目录+文件名称

在node中使用var定义的变量不会加到window里



## 六. 模块化

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



## 七. Node.js不该被用于哪里

- 要避免Node.js被用于构建**繁重的CPU密集型计算**



1. 在后端具有关系数据库的服务器端 Web 应用程序
   1. 如果 Web 应用程序有任何**繁重的 CPU 密集型计算**，它将阻止 Node.js 的响应能力