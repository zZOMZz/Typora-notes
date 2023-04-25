[toc]

# webpack性能优化方案

- 结果优化
  - 分包处理
  - 对代码进行丑化(压缩)
  - 删除无用的代码(tree shaking)
  - CDN服务器
- 打包速度的优化
  - exclude
  - cache

## 一. 代码分离(分包处理)

- 将代码分离到不同的bundle中, 之后我们可以**按需加载**, 或者**并行加载这些文件**
- 在默认情况下, 所有的JavaScript代码(自己编写的, 第三方的)在首页全部加载, 会影响到首页的加载速度, 代码分离可以分出更小的bundle, 以及控制资源加载优先级, 提高代码的加载性能



三种代码分离的方式:

- **入口起点**: 通过entry配置手动分离代码
- **防止重复**: 通过Entry Dependencies或者SplitChunksPlugin去重和分离代码
- **动态导入**: 通过模块的内联函数调用来分离代码



### 1. 多入口起点

#### 1.1 基本设置

- **配置多个入口(entry)**, 每个入口的文件都会单独打包成一个文件

```js
module.exports = {
    entry: {
        index: './src/index.js',
        main: './src/main.js',
    },
    output: {
        path: path.resolve(__dirname,'./build'),
        // 使用[]占位符
        filename: '[name]-bundle.js',  // 在本例中会生成index-bundle.js和main-bundle.js
        clean: true
    }
}
```

- **缺点**: 会包含重复代码, 当两个入口都包含一份第三方库时, 两个文件会重复对这个第三方包进行打包, 造成性能浪费

#### 1.2 改进: 防止重复(dependOn)

- 利用`dependOn`属性人为的标注出chunk所依赖的库或代码

```js
// 改进
module.exports = {
    entry: {
        index: {
        	import:　'./src/index.js',
            dependOn: 'shared1'
        },
        main: {
            import : './src/main.js',
            dependOn: 'shared2'
        }
        
        shared1: ['axios','dayjs'],
        shared2: ['redux']
    },
}
```

#### 1.3 多入口文件需要额外的配置

- 当你在多个chunk中包含同一段的module. 每一个chunk都可能会有自身的state, 这样容易造成不同步, 因此需要额外的配置避免这种情况
- 解决方法就是设置`optimization.runtimeChunk`

```js
module.exports = {
    optimization: {
        runtimeChunk: 'single'
    }
}
```

- 这样除了会生成`shared.bundle.js`和`index.bundle.js`和`main.bundle.js`外, 还会额外生成一个`runtime.bundle.js`文件



### 2. 动态导入(dynamic import)

- **动态导入的文件会进行单独分包**, 这种情况下不需要添加额外的`entry`或者使用`SplitChunkPlugin`插件

- 使用ECMAScript中的`import()`语法来完成
  - `import()`调用内部用到`promise`



- 动态导入的包会被自动分包, 但有时会在页面被请求时立刻被调用, 可能对于性能的优化并不大, 可以在一个`event`触发后再导入


```js
// 我们编写的代码
btn.onclick = function() {
    // 可以通过res拿到about这个模块导出部分
    // import()返回的是一个promise
    import('./router/about').then(res => {
        const print = res.default
        print()
    })
}
// 在点击按钮时, 会自动打包文件
```

```js
async function getComponent() {
    const { default: _ } = await import('loadsh')
}
```

- 如果我们有一个模块, 希望是在**代码运行过程中**加载他, 也并不一定会用到这一部分代码, 因此最好**拆分成一个独立的JS文件**, 这样在不用到这部分内容的时候, 浏览器不需要加载和处理改文件的js代码, 使用动态导入来处理这种情况
- **浏览器会在用到这个包里的代码时, 再将这个文件下载下来**



```js
// webpack.config.js
module.exports = {
    // 可以修改动态导入生成的包的名称
    // 默认为 文件夹名_文件夹名_文件名_打包文件名(下面是bundle.js)
    output: {
        filename: 'bundle.js',
        chunkFilename: '[name]_chunk.js'   // chunk.js还替换掉原来的打包文件名bundle.js
    }
}
```

也可以使用魔法注释改变名称

```js
btn.onclick = function() {
    // 这里写入魔法注释里的值会被当成占位符里的name, 生成的文件名就是 about_chunk.js
    import(/* webpackChunkName: "about" */'./router/about').then(res => {
    })
}
```



### 3. SplitChunksPlugin(打包第三方库)

作用: 

- `SplitChunksPlugin`插件可以将**公共的**依赖的模块提取到已有的入口chunk中, 或者提取到一个新生成的chunk中, 原先的文件中重复的代码会被移除, 减轻了文件的大小
- **可以用这工具将代码中引入的一些第三方库中的代码单独打包到一个包中**, 如: (react, loadsh), 这些第三方库一般不会被频繁修改, 因此将它们打包到别的包中, 可以使浏览器命中缓存的几率增加, 增快加载速度



- 以下是由社区提供，一些对于代码分离很有帮助的 plugin 和 loader：
  - [`mini-css-extract-plugin`](https://www.webpackjs.com/guides/code-splitting/plugins/mini-css-extract-plugin): 用于将 CSS 从主应用程序中分离。


```js
// webpack.config.js
module.exports = {
    // 配置optimization属性
    optimization: {
        // 在不同的环境下生成的id是不同的, 也可以自己设置生成的chunkId的算法
        chunkIds: 'named',
        
        splitChunks: {
            chunks: 'all' ,  // 默认值是async, 只对异步动态导入的import进行分包
            maxSize: 20*1024 ,  // 一个包的最大大小, 大于这个大小的再进行拆包
            minSize: 10*1024,	// 一个包的最小大小, 有默认值
            
            // 自己对需要进行拆包的内容进行分组
            cacheGroups: {
                // name:    生成vendors.[contenthash].js
            	vendors: {
            		test: /node_modules/,
            		filename: "[id]_vendors.js"
        		},
        		utils: {
                    test: /utils/,
                    filename: "[id]_utils.js"
                }
        	},
        }
    }
}
```

- `chunkId`: 占位符id中的值

  - `named`: 在development下的默认值, 会将文件夹和文件名都连起来, 比较长
  - `natural`: 按照数字的顺序使用id
  - `deterministic`: 确定性的, 在**不同的编译中不变的短数字id**, 如`497_utils.js`, 不变的情况下不会再次进行打包, 打包上的性能优化, 有利于浏览器缓存

  - 开发时使用named, 生产时使用deterministic



### 4. externals

- webpack中可以通过配置`externals`来将一些`dependencies`从bundle中去除, 降低bundle的大小, 将这些被排除的`dependencies`当为`peerDependecy`, 需要外部额外引入(CDN), 或者使用本地文件

```js
const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: 'webpack-numbers.js',
      library: {
        name: "webpackNumbers",
        type: "umd"
      },
    },
   externals: {
     lodash: {
       commonjs: 'lodash',
       commonjs2: 'lodash',
       amd: 'lodash',
       root: '_',
     },
   },
  };
```

```html
<!-- CDN -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

<!-- 引入本地文件 -->
<script src="./jquery.min.js"></script>
```





## 二. Prefetch和Preload(预获取和预加载)

在声明`import`时, 可以使用内置指令, 让webpack输出"resource hint(资源提示)", 来告知浏览器

- **prefetch**: 预获取, **将来某些导航**下可能需要的资源, 提前下载
- **preload**: 预加载, **当前导航**下可能需要的资源, 使一些还没被引用的包和主包一起进行下载



- 预获取和预加载的不同之处:
  
  1. **加载时间不同**: preload chunk 会在父chunk加载时, 以并行的方式开始加载; prefetch chunk会在父chunk加载结束后开始加载
  
  2. **加载优先级不同**: preload chunk具有中等优先级, 会立即执行下载; prefetch chunk在浏览器闲置时进行下载
  
  3. **作用时间不同**: preload chunk会在chunk中立即请求, 用于当下时刻; prefetch chunk 会用于未来的某个时刻

```jsx
// 预获取prefetch
btn.onclick = function() {
   
    import(
        /* webpackChunkName: "about" */     // name的魔法注释
        /* webpackPrefetch: true */  	    // 开启预获取
        './router/about').then(res => {
    })
}

// 生成
<link rel="prefetch" href="about.chunk.js">, // 并将其追加到页面头部, 在浏览器空闲时在获取
    
// 预加载
import(/* webpackPreload: true */ 'ChartingLibrary');
```



## 三. CDN

- **CDN称之为内容分发网络(Content Delivery Network 或 Content Distribution Network)**
  - 它是值通过相互连接的网络系统, 利用**最靠近用户的服务器**, 更快, 更可靠地将音乐, 图片, 视频, 应用程序以及其他文件发送给用户, 来提供高性能, 可扩展性及低成本的网络内容传递给用户
  - 需要自己公司购买

- 使用CDN的两种方式:
  - 打包所有的静态资源, 放到CDN服务器中, 用户所有的资源都是通过CDN服务器加载的
  - 一些第三方资源放到CDN服务器上



### 1.使用自己的CDN:

```js
module.exports = {
    output: {
        publicPath: "https://[url]/"
    }
}
```

```html
<!-- html中脚本的src也不应该指向本地服务器, 而是CDN服务器的地址  -->
<!-- 在配置了publicPath后, 加载的所有资源前都会自动拼接上"https://[url]/", 不用手动配置 -->
```



### 2. 使用第三方库的CDN服务器

- 一些比较出名的开源框架都会将打包后的源码放到一些比较出名的, 免费的CDN服务器上
  - 国内: bootcdn
  - 国际: unpkg, JSDelivr, cdnjs

- **打包的时候就不需要对这些第三方库进行打包了(loadash,dayjs), 在HTML模块中, 我们需要自己加入相应的CDN服务器地址**

```js
// webpack.config.js
module.exports = {
    // 排除那些不需要打包的包
    externals: {
        // key属性名表示要排除的框架的名称
        // value值与cdn源代码导出的变量名称有关
        react: "React",
        axios: 'axios'
    }
}
```

```html
<!-- html模板, 不是打包后的文件 -->
<script src='https://[package address in cdn]'>
<!-- 这样引入之后就能正常运行 -->
```



## 四. Terser

[terser/terser: 🗜 JavaScript parser, mangler and compressor toolkit for ES6+ (github.com)](https://github.com/terser/terser)

- **对当前已打包后的代码, 进行进一步的压缩(丑化), 使我们的bundle变得更下, 更有利于传输**
- 生产模式下(production mode)默认是自动开启的, 不需要手动进行配置

### 1. 下载terser依赖

```shell
npm install terser -g # 全局安装
npm install terser -D # 局部安装
```



### 2. 命令行使用terser

[terser文档](https://terser.org/docs/cli-usage)

```shell
terser [input files] [options]
# 如
terser js/file1.js -o foo.min.js -c -m   # 引用全局的terser
npx terser js/file1.js -o foo.min.js -c -m   # 引用局部的terser
# -c compress 压缩   -m mangle 绞肉
```

- `-c`的一些配置
  - `arrows`: class或者object中的函数, 转换为箭头函数
  - `arguments`: 将函数中使用的arguments[index]转换为对应的形参名称
  - `dead_code`: 移除不可达的代码
- `-m`的一些配置
  - `toplevel=true`: 把顶层的一些变量名和函数名全部搅碎, 变成简单的名字
  - `keep_classnames`: 默认值是false, 是否保持原来的依赖的类名称
  - `keep_fnames`: 默认值是false, 是否保持原来的函数名称



### 3. 在webpack中配置terser

- webpack中有一个minimizer属性, **在production模式下, 默认就是使用`TerserPlugin`来处理我们的代码**
- 如果对默认的配置不满意, 也可以自己创建TerserPlugin实例, 并覆盖相关的配置(`optimization.minimizer`)



```js
// webpack.config.js
const TerserPlugin = require('terser-webpack-plugin')

module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            // 1.JS代码简化
            new TerserPlugin({
                // 抽取注释
                extractComments: false,
                terserOptions: {
                    compress: {
                        arguments: true
                    },
                    mangle: true,
                    toplevel: true,
                    keep_fnames: true
                }
            })
        ]
    }
}
```

- **extractComments**: 默认值为true, 表示会将注释抽取到单独的文件中
  - 开发中, 不希望保留这个注释的话, 可以设置为false
- **parallel**: 使用多进行并发运行提高构建的速度, 默认值为false
  - 默认值为: os.cpus().length - 1
- **terserOptions**: 设置我们的terser相关的配置
  - compress: 设置压缩相关的配置
  - mangle: 设置丑化的相关选项, 可直接设置为true
  - toplevel: 设置顶层变量是否发生转换
  - keep_classnames:保留类的名称
  - keep_fnames: 保留函数的名称



### 4. 对css进行压缩

- **主要是压缩去除css文件中的一些空格**

```shell
npm install css-minimizer-webpack-plugin -D
```

```js
// webpack.config.js
const CSSMinimizerPlugin = require('css-minimizer-webpack-plugin')

module.exports = {
    optimization: {
        minimize: true,
        minimizer: [
            // 1.JS代码简化
            new TerserPlugin({}),
            // 2. css压缩
            new CSSMinimizerPlugin({
                parallel: true
            })
        ]
    }
}
```



## 五. Tree Shaking(JS)

### 1. 基本介绍

- 目的作用是为了**消除死代码(dead_code)**
- Tree Shaking依赖于**ES Module**的静态语法分析(不执行任何代码, 可以明确的知道模块之间的依赖关系)
  - Tree shaking是通过模块系统实现的, 这意味着模块的导入和导出必须在代码的顶部, 这使得编译器可以在编译时静态分析模块, 从而决定哪些导入和导出是live code 哪些是dead code
  - webpack会将源码转换为抽象语法树(AST), 分析AST,找到被引入的模块和模块中的导出, 最后检查哪些导出被使用了, 哪些导出没有被使用, 没有使用的导入也会被标记为死代码(dead code)

- commonjs或AMD是动态加载模块并执行模块中的代码, 无法想ES module在编译时就能确定模块的依赖关系和输出内容, 因此webpack在处理这种模块时会有所限制





### 2. usedExports

- **分析导入模块时, 该模块的哪些函数有被使用; 哪些函数没有被使用**, 并不会真的删除代码, 删除代码时terser的任务
- 在production模式下会自动开启

```js
module,exports = {
    optimization: {
        usedExports: true
    }
}
```

- 接着会对没有被使用的函数前 添加上魔法注释`/*unused harmony export [name] */`
- terser压缩时就会根据这个注释, 放心大胆的删除这个函数的定义

```js
// 这个例子中, 没有使用printMe2, 因此加上了这个注释

/* unused harmony export printMe2 */
function printMe() {
    // console.log(sadasd)
  console.log('I get called from print.js!');
}

function printMe2() {
  // console.log(sadasd)  
  console.log('I get called from print.js!');
}
```



### 3. sideEffects

- 指定具有副作用的文件, 将整个模块打包, 而不是打包部分, 相当于一种白名单

- 有时,整个文件的函数都没被别的地方调用, 在usedExports的作用下, 那些函数会被删除, **但是由于不清楚是否存在副作用, 这个模块(文件)还是会被依旧保留, 即使它没有作用**
  - 副作用: `window.name = "zzt"`, 给全局设置变量, **一些可以影响别的模块的代码**

- 可以直接配置package.json中的sideEffect属性

```json
{
    "sideEffects": false  // 表示所有导入的文件都不存在副作用, 可以安心删除一些文件
    
    "sideEffects": [	// 配置一些避免被tree shaking 的文件
    	"*.css"			// 因为css中的内容一般不会在模块中使用, 因此需要避免被优化
    ]
}
```



### 4. 魔法注释

- `/*#__PURE__*/`与`sideEffect`, 但`sideEffect`作用于整个模块层面, 而`/*#__PURE__#*/`用于标记纯函数的特殊注释, 可以告知webpack等构建工具, 这是一个无副作用的纯函数, 可以被tree shaking处理掉

```js
/*#__PURE__*/
function add(a, b) {
  return a + b;
}
```



## 六. Tree Shaking(CSS)

- **去除未被使用的CSS**

- webpack默认会进行js的Tree Shaking, 但是默认不会对CSS进行Tree Shaking, 因此需要借助插件
  - **PurgeCSS**

```shell
npm install purgecss-webpack-plugin -D
```

```js
// 配置在生产环境中

// 使用glob模块匹配src文件夹下的所有文件
const glob = require('glob')
const { PurgeCSSPlugin } = require('purgecss-webpack-plugin')

module.exports = {
    plugins: [
        new PurgeCSSPlugin({
            // src文件夹下的所有文件(包括子文件夹下的文件), nodir排除文件夹
          path: glob.sync(`${path.resolve(__dirname,'../src')}/**/*`,{ nodir: true })
        })
    ]
}
```



## 七. Scope Hoisting

- **功能是对作用域进行提升, 并且让webpack打包后的代码更小, 运行更快**
- 不使用Scope Hoisting时, 会单独的把一个模块打包成一个函数
- 默认情况下webpack打包会有很多的函数作用域, 包括一些(比如最外层的IIFE)
  - 无论是从最开始的代码运行, 还是加载一个模块, 都需要执行一系列的函数, 在一个模块中调用另一个模块的函数, 一般情况下, 需要跨越那个模块的作用域才能拿到函数
  - **Scope Hoisting可以将函数合并到一个模块中来运行, 避免了跨越作用域**

```js
const webpack = require('webpack')

module.exports = {
    plugins: [
        // 作用域提升
        new webpack.optimize.ModuleConcatenationPlugin()
    ]
}
```



## 八. HTTP压缩

- HTTP压缩是一种**内置**在服务器和客户端之间的, 以改进**传输速度**和**带宽利用率**的方式

- 可以使用gzip算法生成压缩文件, **浏览器在接收到压缩文件(.gz)后, 会自动对其进行解压, 无需额外配置**



常见的压缩格式:

- compress(不推荐)
- deflate: 基于deflate算法的压缩, 使用zilb数据格式封装
- **gzip**: GNU zip, 目前使用广泛
- br: 一种新的开源压缩算法, 专为HTTP内容的编码而设计

### 1. 压缩js, css

**webpack中进行压缩配置**

- 使用`CompressionPlugin`

```shell
npm install compression-webpack-plugin
```

```js
// 开发环境
module.exports = {
    plugins: [
        new CompressionPlugin({
            test: /\.(css|js)$/,	// 哪些文件需要压缩
            threshold: 500			// 设置文件多大才需要压缩
            minRatio: 0.7,			// 至少采用的压缩比
            algorithm: "gzip"		// 采用的压缩算法
        })
    ]
}
```



### 2. 压缩html

- 设置模板的插件`HTMLWebpackPlugin`会默认对HTML进行压缩

```js
// 开发环境
module.exports = {
    plugins: [
        new HTMLWebpackPlugin({
            tempalte: './index.html',
            // 默认为true, 可以直接设置为false, 也可以自己进行配置
            minify: {
                // 移除注释
            	removeComments: true,
            	// 移除空的属性
            	removeEmptyAttributes: true
                // 折叠空白字符
                collapseWhitespace: true,
                // 移除默认属性
                removeRedundantAttributes: true
                // 压缩内联的css
                minifyCSS: true
            },
        })
    ]
}
```



## 九. bundle分析

[官方分析工具)](https://github.com/webpack/analyse)

社区支持的可选项:

- [webpack-chart](https://alexkuz.github.io/webpack-chart/): webpack stats 可交互饼图。
- [webpack-visualizer](https://chrisbateman.github.io/webpack-visualizer/): 可视化并分析你的 bundle，检查哪些模块占用空间，哪些可能是重复使用的。
- [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)：一个 plugin 和 CLI 工具，它将 bundle 内容展示为一个便捷的、交互式、可缩放的树状图形式。
- [webpack bundle optimize helper](https://webpack.jakoblind.no/optimize)：这个工具会分析你的 bundle，并提供可操作的改进措施，以减少 bundle 的大小。
- [bundle-stats](https://github.com/bundle-stats/bundle-stats)：生成一个 bundle 报告（bundle 大小、资源、模块），并比较不同构建之间的结果。