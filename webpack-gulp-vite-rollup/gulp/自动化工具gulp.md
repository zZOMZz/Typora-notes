# 自动化工具gulp

- **编写一系列的自动化任务, 一个工具包, 可以帮你自动化和增加工作流**



## 一. 与webpack的区别

- gulp的核心理念是**task runner**
  - 可以定义自己的一系列任务, 等待任务被执行
  - 基于文件Stream的构建流
  - 我们可以使用gulp的插件体系来完成某些任务

- webpack的核心理念是**module bundler**
  - webpack是一个模块化的打包工具
  - 可以使用各种各样的loader来加载不同的模块
  - 可以使用各种各样的插件在webpack打包的生命周期完成其他的任务



## 二. 简单使用gulp

```js
// gulpfile.js
const foo = (callback) => {
    console.log("执行了foo")
    callback()
}

module.exports = {
    foo
}
```

```shell
npx gulp foo
```



## 三. 创建gulp任务

- 每个gulp任务都是一个异步的JavaScript函数:

  - 次函数可以接受一个callback作为参数, 调用callback函数可以结束这个任务

  - 或者返回一个stream, promise, event emitter, child process或observable类型的函数

- 任务可以是public或者private类型的:

  - **公开任务(Public tasks)**: 从gulpfile中被导出, 可以直接通过gulp命令调用
  - **私有任务(Private tasks)**: 被设计在内部使用, 通常作为series()或parallel()组合的组成部分

  ```js
  const seriesFoo = series(foo1,foo2,foo3)  // 串行执行
  const parallelFoo = parallel(foo1,foo2,foo3)	// 并行执行
  module.exports = {
      seriesFoo,
      parallelFoo
  }
  ```

  

## 四. 读取和写入文件

- **gulp暴露了src()和dest()方法用于处理计算机上存放的文件**
  - src()接收参数, 并从文件系统中读取文件然后生成一个Node流(stream), 他将所有匹配的文件读取到内存中并通过流(stream)进行处理
  - 由src产生的流应当从任务(task函数)中返回并发出异步完成信号
  - dest()接受一个输出目录作为参数, 并且它还会产生一个Node流, 通过该流将内容输出到文件中

```js
const { src, dest } = require('gulp')

const copyFile = () => {
    // 1. 先读取文件, 2. 再写入文件
    return src('./src/**/*.js').pipe(dest('./dist'))
}

module.exports = {
    copyFile
}
```



## 五. 使用插件

- 在官网上寻找插件

```js
const { src, dest } = require('gulp')
const babel = require('gulp-babel')
const terser = require('gulp-terser')

const jsTask = () => {
    // 1. 先读取文件, 2. 再写入文件
    return src('./src/**/*.js')
        	.pipe(babel())
    		.pipe(terser({ mangle: { toplevel: true } }))
        	.pipe(dest('./dist'))
}

module.exports = {
    jsTask
}
```

