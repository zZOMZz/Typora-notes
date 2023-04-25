[toc]

# Webpack深入学习

## 一. source-map

- 真实跑在浏览器上的代码, 和我们实际编写的代码是不一样的, 因此当打包后的代码(production)需要调试时(debug), 是非常难定位错误的, 因此就可以使用source-map
- **source-map是从已转换的代码, 映射到原始的源文件**, 使浏览器可以**重构源文件**并在调试器中**显示重建的原始源**

```js
const path = require('path')

module.exports = {
    entry: './src/main.js',
    // 添加devtool, 可以选择不同的值, 具体见官方文档
    devtool: 'source-map'
    output: {
        path: path.resolve(__dirname,'./build')
        filename: 'bundle.js'
    }
}
```

- 打包完成后会额外生成一个`bundle.js.map`映射文件, 同时`bundle.js(打包后生成的文件)`会在最后一行添加上一段可以被**浏览器解析**的注释`//# sourceMappingURL=bundle.js.map`



- none: production模式下的默认值, 不生成source-map

- eval: development模式下的默认值值, 不会生成source-map
  - 但是会用eval函数包裹执行代码, 并在eval函数的末尾添加`//# sourceURL=`
  - 它会在浏览器执行代码时被解析, 并且在调试面板中生成对应的一些文件目录, 方便我们调试代码
  - 但是相对于直接生成的source-map缺乏一定的准确性, 但性能更高



## 二. CSS处理

### 1. css loader(处理css文件)

- 帮助webpack处理别的类型的文件(.css, .vue ....)

```js
module.exports = {
    module: {
        rules: [
            {
             	test: /\.css$/,
                use: [
                    // loader的使用顺序是从后往前
                    { loader: "style-loader", options: {} },
                    { loader: "css-loader" },
                    { loader: "postcss-loader",
                      options: {
                          postcssOptions: {
                              plugins: [
                                  "autoprefixer"
                              ]
                          }
                      }
                    }
                ]
                // 简写: use: ["style-loader","css-loader",""] 不需要其他属性时
            }
        ]
    }
}

// npm install css-loader -D
```

- **css-loader**只负责解析css
- **style-loader**负责将css style插入到页面内, 创建一个<style></style>标签
- **postcss-loader**: 是一个通过Javascript来转换样式的工具
  - 进行一些css的转换和适配, 如: 添加浏览器前缀, css样式重叠
  - **可以看成CSS中的babel**



### 2. 将css打包文件单独分离出

- **利用mini-css-extract-plugin插件**

```shell
npm install mini-css-extract-plugin -D
```

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name]_[hash:6].css'
            // 动态导入的css文件分包名
            chunkFilename: '[name]_chunk.css'
        })
    ]
}
```

- **因为是单独打包成一个文件, 所以不需要使用style-loader将css插入到html中, 需要以link的方式将单独文件导入到html中**

```js
module.exports = {
    module: {
        rules: [
            {
                test: /.css$/,
                use: [
                    MiniCssExtractPlugin.loader,  // 生产环境推荐
                    'css-loader'
                ]
            }
        ]
    }
    
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'css/[name]_[hash:6].css'
            // 动态导入的css文件分包名
            chunkFilename: '[name]_chunk.css'
        })
    ]
}
```



## 三. Hash, ContentHash, ChunkHash

- 在给项目打包命名的文件placehold中, 可以使用hash
  - hash本身是通过MD4的散列函数处理后, 生成一个128位的hash值(32个十六进制)

**hash**

- 在**文件内容不发生变化**的情况下, 打包出来的hash值是不变的; 反之任何一个入口文件发生改变, 所有的打包文件hash值都会发生变化, 即使别的打包文件内容没变化(**不利于浏览器的缓存工作**)

**contentHash**:

- 不同文件之间不会产生影响, 生成的文件hash名称只与内容有关

**chunkHash**:

- 和contentHash一样, 能保证不同文件之间不会像影响, 但是当一个文件发生改变时, 可能影响它导入文件的名称



## 四. DLL库

- 动态链接库(Dynamic Link Library), 是为软件在Windows中实现共享函数库的一种实现方式
- webpack中也有内置DLL的功能, **我们能够共享, 并且不经常改变的代码, 单独抽取成一个共享的库, 这个库在之后的编译的过程中, 会被引入到其他项目的代码中**



使用步骤:

- 打包一个DLL库
- 引入一个DLL库



VUE和React的脚手架都移除了DLL库



## 五. 抽取不同的开发配置和生产配置

需要安装`webpack-merge`

```shell
npm install webpack-merge -D
```

### 1. 文档方案

- 分为3个文件
  - webpack.config.js
  - webpack.dev.js
  - webpack.prod.js

```js
// webpack.config.js 一些共同的配置
const path = require('path');
 const HtmlWebpackPlugin = require('html-webpack-plugin');

 module.exports = {
   entry: {
     app: './src/index.js',
   },
   plugins: [
     new HtmlWebpackPlugin({
       title: 'Production',
     }),
   ],
   output: {
     filename: '[name].bundle.js',
     path: path.resolve(__dirname, 'dist'),
     clean: true,
   },
 };
```

```js
// webpack.dev.js
const { merge } = require('webpack-merge');
 const common = require('./webpack.common.js');

 module.exports = merge(common, {
   mode: 'development',
   devtool: 'inline-source-map',
   devServer: {
     static: './dist',
   },
 });
```

```js
// webpack.prod.js
const { merge } = require('webpack-merge');
 const common = require('./webpack.common.js');

 module.exports = merge(common, {
   mode: 'production',
 });
```

- 通过`webpack-merge`中的`merge`函数, 将`dev`环境和`prod`环境进行区分和合并

  

- 这里的方法是使用两个文件, 下面的方法是只使用一个文件`common.config.js`

```json
{
    "scripts": {
        "start": "webpack serve --open -- config webpack.dev.js",
        "build": "webpack --config webpack.prod.js"
    }
}
```





## 2. 视频教程方案

分成三个文件:

- **comm.config.js**: 这个文件保存两种环境**相同**的配置选项, 且把这个环境配置导出为一个**函数**
- **dev.config.js**: 这两个文件保留一些**不同**的配置选项
- **prod.config.js**

```json
"scripts": {
    "build": "webpack --config [path] --env production",
    "serve": "webpack serve --config [path] --env develpoment"
}
```

- 下面的例子使导入和导出只使用`common.config.js`通过传入的环境变量来进行配置的区分

```js
// common.config.js
// webpack接收我们将配置设置为一个函数, 这个函数得返回配置对象
// 同时这个函数会接收webpack被调用时传入的参数
const { merge } = require('webpack-merge')
const devConfig = require('./dev.config')
const prodConfig = require('./prod.config')

const getCommConfig = function(isProduction) {
    return {
        ...
        isProduction ? ... : ...
    }
}

module.exports = function(env) {
    const isProduction = env.production
    const mergeConfig = isProduction ? prodConfig : devConfig
    return merge(getCommConfig(isProduction),mergeConfig)
}
```



## 六. 打包分析

### 1. 分析插件和loader消耗的时间

```shell
npm install speed-measure-webpack-plugin -D
```

```js
const SpeedMeasurePlugin = require('spead-measure-webpack-plugin')

const smp = new SpeedMeasurePlugin()

// 将webpack的配置放入wrap中
module.exports = function(env) {
    
    return smp.wrap(mergeConfig)
}
```



### 2. 对打包后的文件进行分析

**方案一:**

- 将script中的build改为: `webpack --config [filepath] --env production --profile --json=status.json`
  - 这样运行build命令后会自动生成`status.json`文件
- 去文档找官方维护的网站, 如果没有, 可以[webpack/analyse: analyse web app for webpack stats (github.com)](https://github.com/webpack/analyse)下载这个库, 然后自己运行分析

**方案二:**

- 使用`webpack-bundle-analyzer`工具

```shell
npm install webpack-bundle-analyzer -D
```

```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin

module.exports = {
    plugins: [
        new BundleAnalyzerPlugin()
    ]
}
// 配置完后, 打包时就会自动打开一个页面
```



## 七. 查看源码

```js
// build.js
const webpack = require('../lib/webpack')
const config = require('./webpack.config')

const compiler = webpack(config)

compiler.run((err,stats) => {
    if(err) {
        console.error(err)
    } else {
        console.log(stats)
    }
})
```

```shell
# 运行webapck
node build.js
```

![Videoframe_20230123_140714_com.huawei.himovie](.\图片\Videoframe_20230123_140714_com.huawei.himovie.jpg)
