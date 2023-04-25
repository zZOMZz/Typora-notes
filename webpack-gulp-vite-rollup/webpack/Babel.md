[toc]

# Babel

## 一. Babel简单介绍

- 开发中如果想使用ES6+的语法, 使用TypeScript, 开发React项目, 它们都是离不开Babel的
- 主要用于旧浏览器或者旧环境中将ES6+以上的代码向后转换为兼容版本的JS代码, 包括语法转换, 源代码转换, 
- **polyfill**实现目标环境缺少的功能(一种垫片)



## 二. 单独使用Babel

- babel可以不和webpack等构建工具配置来使用, 本身也是能进行单独使用的
- 所需的依赖:
  - **@babel/core**: babel的核心代码
  - **@babel/cli**: 可以让我们在命令行使用babel

```shell
npx babel ./src --out-dir --plugins=XXX,XXX ./build  # 将src中的所有文件转换到build文件夹下

# 下载转换插件, 这里下载的是转换const,let到var的插件
npm install @babel/plugin-transform-block-scoping -D
# 下载转换箭头函数的插件
npm install @babel/plugin-transform-arrow-functions -D
```



## 三. 使用预设

- 预设: 将**一组相关的Babel插件**组合在一起, 预设可以简化Babel的配置和使用. 开发人员可以根据自己的需要选择不同的预设, 以便在编译代码时使用不同的转换规则

- 使用上面的方法一个一个的安装插件太麻烦, 可以使用预设来进行一次性配置, 同时预设会根据我们适配哪些浏览器来进行转换

```shell
# preset-env对一些js代码进行转换
npm install @babel/preset-env -D

npx babel ./src --out-dir ./build --presets=@babel/preset-env
```

- 常见的预设(预设都需要安装)
  - `@babel/preset-env`: 用于转换ES6+语法和新特性, 以便在当前的浏览器或环境中运行
  - `@babel/preset-react`: 用于转换React JSX语法, 以便在当前的浏览器环境中运行
  - `@babel/preset-typescript`: 用于转换typescript代码, 以便在当前的浏览器或环境中运行
  - `@babel/preset-flow`: 用于转换Flow代码, 以便在当前的浏览器或环境中运行
  - `@babel/preset-stage-0`到`@babel/preset-stage-4`: 用于转换正在提案中的ECMAScript特性, 以便在当前的浏览器或环境中运行

- 总之, 预设可以大大简化JavaScript编译器的配置和使用, 并使开发人员更轻松的使用最新的JavaScript语言特性

## 四. browserslist

### 1. .browserslistrc配置文件

- **告诉像postcss和babel需要适配的浏览器, 自动调整转换的幅度大小**

```js
> 0.2%    // 市场占有率
last 2 version
not dead   // 24个月官方还有更新和维护

// 三个条件之间是或的关系
```

[查看浏览器市场占有率(caniuse)](https://caniuse.com/usage-table)

### 2. browerserslist工具

- **在不同前端工具之间, 共享目标浏览器和Node.js版本的配置**, 会根据我们的配置来获取相关的浏览器信息, 以方便决定是否需要进行浏览器兼容的支持
  - Autoprefixer
  - Babel
  - postcss-preset-env
  - eslint-plugin-compat
  - stylelint-no-unsupported-browser-features
  - postcss-normalize
  - obsolete-webpack-plugin
- 条件查询时使用的是caniuse-lite工具, 这个工具会去caniuse网站上查询相关的数据



### 3. browserslist编写规则

- **defaults(默认):** Browserslist的默认浏览器 (＞0.5%, last 2 version, Firefox ESR, not dead)
- **5%(百分数)**: 通过全局使用情况统计信息选择的浏览器版本
  - 5 % in US: 使用美国地区的使用情况统计
  - `>5%` in alt-AS: 使用亚洲地区使用情况统计信息
  - cover 99.5% 提供覆盖率的最受欢迎的浏览器
- **dead**: 24个月没有官方支持或者更新的浏览器
- **last 2 version:** 每个浏览器的最后两个版本
  - last 2 Chrome version: 最近的两个版本的Chrome浏览器
  - last 2 major version或last 2 iOS major versions: 最近的2个主要版本的所有次要/补丁版本

 

### 4. 使用browserslist工具

- 在安装babel时, 由于babel会依赖使用browserlist, 因此browserlist会自动被安装

```shell
npx browserslist ">1%, last 2 version, not dead" # 条件之间是或的关系
```

- babel会自动调用这个工具查询, **只需要编写browserslistrc配置文件**



## 五. Babel的底层原理

[jamiebuilds/the-super-tiny-compiler: Possibly the smallest compiler ever (github.com)](https://github.com/jamiebuilds/the-super-tiny-compiler)

[YongzeYao/the-super-tiny-compiler-CN: 这是GitHub项目the-super-tiny-compiler的中文翻译。](https://github.com/YongzeYao/the-super-tiny-compiler-CN)

-  **可以将Babel看成是一个编译器**,将我们的代码从一种源代码转换到另一种源代码(目标语言)



工作流程:

1. **解析阶段**
   - 词法分析, 生成tokens

2. **转换阶段**
   - 语法分析,生成AST树, 再**调用传入的plugin对AST抽象语法树进行处理**, 然后**生成一个新的AST抽象语法树** 

3. **生成阶段**
   - 在利用新的AST抽象语法树生成代码



## 六. 在webpack中调用babel

- 需要使用`babel-laoder`来解析所有的`.js`模块

```js
// 不使用预设的情况
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        plugins: [
                            '@babel/plugin-transform-arrow-functions'
                        ],
                        // 可以使用预设
                        // 也可以在babel.config.js配置文件中设置
                        presets: []
                    }
                }
            }
        ]
	}
}
```



## 七. Babel的配置文件

- 可以将babel的所有配置放到一个单独的文件中, 两种配置文件编写方式
  - **babel.config.json**(或者.js, .cjs, .mjs)文件, 可以直接作用于Monorepos项目里的子包
  - **.babelrc.json**(或者.babelrc, .js, .cjs, .mjs) 文件(早期的项目)

```js
// js文件
// 这里写的配置会自动导入到webpack中的babel-loader后的配置选项options里
module.exports = {
    presets: [
        ['@babel/preset-env',{
            target: {
                
            }
        }]
    ]
}
```



## 八. polyfill

### 1. 定义

- **一种填充物(垫片), 一个补丁, 在浏览器中动态注入一些JavaScript代码, 通过模拟某些新的ECMAScript API来实现老版本浏览器中的兼容性, 可以帮助我们更好的使用JavaScript**
-  一些高级的API无法转换成ES5中的特殊语法, 因此babel不知道如何去转换, 以至于直接将特殊语法保存了下来, 如源代码`new Promise()`, ES5中无法找到Promise这个类, 因此就无法完成转换,直接运行时会直接报错,  **这个时候就需要给Promise打上补丁(polyfill)填充Promise**
  - 比如`'xxx'.includes('a')`includes这个方法, babel可能只认为其是一个方法调用无法正确转换, 需要打上`string.prototype.includes = ..`这个补丁

- 一些较老的浏览器版本中, 一些语言特性可能并未完全实现或者支持不够完善, 为了让代码能够在所有的浏览器中正常运行, 开发者需要使用polyfill来填充浏览器的语言特性缺失
- 在一些现代浏览器中依旧需要使用, 它也可以修复损坏实现(repair broken implem)



### 2. 使用polyfill

- 安装`core-js`和`regenerator-runtime`

```shell
npm install  core-js regenerator-runtime
```

```js
// babel.package.js
module.exports = {
    presets: [
        ['@babel/preset-env',{
            corejs: 3,
            useBuiltIns: 'usage'
        }]
    ]
}
```

- `useBuiltIns`

  - false: 不使用polyfill进行填充
  - usage: 查看我们引入了哪些特殊的API, 从而自动去core-js和regenerator-runtime中寻找相关API并引入, 确保了最终包里的polyfill数量的最小化, 打包的包也会小一点(推荐)
  - entry: 如果担心第三方的库中会使用一些特殊的API, 也可以为这些API引入垫片避免报错, **也会根据browserslist**

  ```js
  // 自己编写的文件, 入口文件
  import 'core-js/stable'
  import 'regenerator-runtime/runtime'
  // 这样就能将所有标准的API和运行中需要的API的垫片全部引入到打包后的文件中
  ```

  

- `corejs`: 使用corejs的版本

### 3. core-js

- `core-js`是一个JavaScript标准库, 主要是提供许多JavaScript新特性的`polyfill`, 可以在不支持这些特性的浏览器上实现相同的功能
- 可以在`Babel`中使用

```js
// babel.config.js
module.exports = {
    presets: [
        "@babel/preset-env",
        {
            useBuiltIns: "useage",	// 自动检测代码中使用的ES6+语言特性, 	只会导入需要的										polyfill, 而不会导入整个polyfill包, 从而减小最终打包									的文件大小
            corejs: "3.19"   		// 确定所使用的core-js的版本
        }
    ]
}
```

```js
// 入口文件中, 手动引入polyfill
import "core-js/stable";		// 具体引入哪些可以看文档按自己需要
import "regenerator-runtime/runtime";
```

- `core-js/stable`: 导入了所有稳定的polyfill
- `regenerator-runtime/runtime`: 导入了Generator和async/await的polyfill



### 4. regenerator-runtime

- 一个JavaScript运行时, 用于支持JavaScript中的异步编程, 特别是`async/await`语法
  - `async/await`是基于JavaScript的`Generator`和`Iterator`发展而来的, 因此一定程度上也可以说是对这两种语法的`polyfill`
- 当使用`@babel/plugin-transform-runtime`插件时, 会自动导入包括`regenerator-runtime`和其他一些运行时库

```js
// 入口文件, 手动导入进行polyfill
import 'regenerator-runtime/runtime';
```





## 九. 使用React(babel处理jsx)

### 1. 使babel处理jsx

插件处理:

- `@babel/plugin-syntax-jsx`
- `@babel/plugin-transform-react-jsx`
- `@babel/plugin-transform-react-display-name`

### 2. 使用预设

- `@babel/preset-react`

```shell
npm install @babel/preset-react -D
```



## 十. Babel处理TypeScript

- **处理TypeScript必须配置`tsconfig.json`文件**, `tsc --init`
- 可以使用`ts-loader`来处理TypeScript, 也可以使用`babel-loader`

处理方法:

- 使用`ts-loader`: `npm install ts-loader`
  - 缺点: **ts-loader与babel-loader不同, ts-loader无法调用polyfill相关内容, 无法处理特殊的API**

```js
// webpack.config.js
moduel.export = {
    module: {
        rules: [
            {
                test: /\.tsx?$/
                use: {
                	['ts-loader']
            	}
            }
        ]
    }
}
```

- 使用`babel-loader`来处理(推荐), **可以对一些特殊的API打上补丁**, 但是, **缺点是babel-loader在编译时, 不会对类型错误进行检测**

  - 插件处理: `@babel/transform-typescript`
  - **预设处理:**`@babel/preset-typescript`(更推荐)

  ```js
  // babel.config.json
  module.export = {
      presets: [
          ['@babel/preset-typescript',{
              // 配置预设
              corejs: 3,
              useBuiltIns: 'usage'
          }]
      ]
  }
  ```

- ==因此可以使用babel来进行转换Typescript代码, 使用tsc来进行类型检查==

  - 增加两个类型检查的脚本

  ```json
  "scripts": {
      "ts-check": "tsc --noEmit"  // noEmit是不要输出
      "ts-check-watch": "tsc --noEmit --watch"  // 实时进行监听
  }
  ```

  

  - `npm run ts-check`可以对ts代码的类型进行检测
  - `npm run ts-check-watch`可以实时的检测类型错误