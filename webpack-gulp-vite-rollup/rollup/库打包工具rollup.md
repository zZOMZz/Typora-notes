[toc]

# 库打包工具rollup

## 一. 认识rollup

- **rollup是一个JavaScript的模块化打包工具, 可以帮助我们编译小的代码到大的, 复杂的代码中, 比如一个库或者应用程序**



- rollup与webpack非常相似
  - rollup也是一个模块化的打包工具, 但是rollup主要是针对**ES Module**进行打包的
  - 另外webpack通常可以通过各种的loader处理各种各样的文件, 以及处理它们的依赖关系
  - **rollup更专注于处理JavaScript代码**(当然也可以处理css, font, vue等), **因此相对webpack更加轻量级**



## 二. 打包结果区别不同的环境

```shell
# -f format
-f cjs # node环境, 支持commonjs
-f iife # browser环境, 有全局对象
-f amd # AMD环境
-f umd --name zzt # UMD环境, 上面的环境全部支持, 需要一个名字
```

```shell
npx rollup ./lib/index.js -f cjs -o dist/bundle.js
```



## 三. rollup.config.js

```js
const commonjs = require('@rollup/plugin-commonjs')
const nodeResolve = require('@rollup/plugin-node-resolve')
const babel = require('@rollup/plugin-babel')
const terser = require('@rollup/plugin-terser')

module.exports = {
    input: './lib/index.js',
    // 可以同时打包多种格式的文件
    output: [
        {
            format: "umd",
            name: "zzt",
            file: "./build/bundle.umd.js",
            globals: {				// 打包引入的第三方库
                loadsh: "_"
            }
        },
        {
            format: "amd",
            file: "./build/bundle.amd.js"
        }
    ],
    plugins: [
        commonjs(),
        nodeResolve(),
        babel({  						// 需要配置babel.config.js
            babelHelper: "bundled",
            exelude: /node_modules/
        }),
        terser()  
    ]
}
```

- **一些第三库不会被打包, 因为其使用的是commonjs, 而rollup默认会对ES module进行打包, 因此需要使用一个plugin**

```shell
# 安装解决commonjs的库, 使其可以打包用commonjs导入导出的库或文件
npm install @rollup/plugin-commonjs -D  
# 安装解决node_modules的库, 使其可以打包node_modules下的库
npm install @rollup/plugin-node-resolve -D  
# 代码转换的插件
npm install @rollup/plugin-babel -D
# 压缩代码
npm install @rollup/plugin-terser -D
```



```js
module.exports = {
    presets: ["@babe/preset-env"]
}
```



## 四. 处理css

- 需要使用插件

- 如果项目中需要处理css文件, 可以使用postcss:

```js
npm install rollup-plugin-postcss postcss -D
```



## 五. 处理vue文件

- 需要使用插件`@vue/compiler-sfc  rollup-plugin-vue`

```js
const vue = require('rollup-plugin-vue')
```

- **需要插入一个全局变量**

```shell
npm install @rollup/plugin-replace -D
```

```js
const replace = require('@rollup/plugin-replace')

module.exports = {
    plugins: [
        replace({
            "process.env.NODE_ENV": JSON.stringify("production")
        })
    ]
}
```



## 六. 搭建本地服务器

1. 使用`rollup-plugin-serve`搭建服务
2. 当文件发生变化时自动刷新浏览器, 使用`rollup-plugin-livereload`插件
3. 启动服务时, 开启文件监听`加上 -w, 现在: npx rollup -c -w     `

```js
const serve = require('rollup-plugin-serve')
const livereload = require('rollup-plugin-livereload')

module.exports = {
    plugins: [
        serve({
            port: 8000,
            open: true,
        }),
        livereload(),
        
    ]
}
```



## 七. 区分不同环境

```shell
# 开发环境
rollup -c --environment NODE_ENV:development -w
# 生产环境
rollup -c --environment NODE_ENV:production
```

