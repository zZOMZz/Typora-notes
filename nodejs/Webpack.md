[toc]

# Webpack

[ webpack 中文文档 ](https://www.webpackjs.com/concepts/)

Webpack是跑在node环境里的

```yaml
npm i webpack webpack-cli -D # 一般为局部安装, webpack-cli是为了利用命令行
npx webapck # 使用局部的webpack
```



## 一. 认识Webpack

### 1. node中的内置模块path

[Path ](https://nodejs.org/docs/latest-v17.x/api/path.html)

- path: path模块对路径和文件进行处理
  - dirname: 拿到文件的父文件夹名
  - extname: 拿到文件后缀名
  - basename: 获取文件名
  - path.join([...paths]): 将两个路径拼接在一起
  - **path.resolve([...paths])**: 将多个路径拼接在一起, 最后一定返回一个绝对路径, 从右往左,直到遇到"/", 如果还没得到绝对路径, 则和当前路径(目录不包括当前文件)进行拼接



### 2. webpack loader

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
                // 简写: use: ["","",""] 不需要其他属性时
            }
        ]
    }
}

// npm install css-loader -D
// css-loader只负责解析, style-loader负责插入到页面内
```

- postcss-loader: 是一个通过Javascript来转换样式的工具
  - 进行一些css的转换和适配, 如: 添加浏览器前缀, css样式重叠



### 3. asset module type

- 资源模块类型(asset module type) 是用来替代一些loader的,如: raw-loader, url-loader, file-loader
- 4种新的模块类型
  - **asset/resource**: 发送一个单独的文件并导出URL, 需要额外的网络请求
  - asset/inline 导出一个资源的data URI, 利用了浏览器可识别的**编码base64**, 编码的结果放到一个js文件内
  - asset/source 导出资源的源代码(用的少)
  - asset 在导出一个data URI 和发送一个单独的文件之间自动选择
    - 添加parser属性, 并且制定dataURL的条件, 添加maxSize属性

```js
module.exports = {
    module: {
        rules: [
            {
                test: /\.(png|jpe?g|svg|gif)$/,
                type: "asset"
                parser:{
                	dataUrlCondition: {
                		// 决定什么样的图片需要单独url打包
            	    	maxSize: 100 * 1024
           	 		}
            	},
            	generator: {
            		// 占位符: []; name: 原来的图片的名字; ext: 原来图片的后缀名
            		// hash:X webpack生成的哈希值
            		// img/ 生成一个文件夹
            		filename: "img/[name].[hash:6][ext]"
            	}
            },
    		{
    			 test: /\.js$/,
    			 use: ["babel-load"]
			}
        ]
    }
}
```

- 当使用babel要转换的内容过多(打的补丁过多), 可以使用预设, 避免一个一个的设置, webpack会根据我们的预设来加载对应的插件列表, 并且将其传入babel

```yaml
npm install @babel/preset-env -D
```

常见预设: 

- env
- react
- TypeScript



### 4. resolve模块解析

- resolve用来设置模块如何被解析, 帮助webpack从每个(require/import)语句中, 找到需要引入到合适的模块代码
- webpack使用enhanced-resolve来解析文件路径

导入文件,文件夹; 别名

- 如果引入的是文件(文件具有扩展名),则直接打包文件, 负责使用`resolve.extensions`选项作为文件扩展名解析
- 如果引入的是一个文件夹: 则会根据`resolve.mainFiles`配置选项中指定的文件顺序查找
  - `resolve.mainFiles`的默认值是'['index']'
  - 再根据`resolve.extensions`来解析扩展名

```js
module.exports = {
    resolve: {
        extensions: [".js",".json",".vue",".ts",".tsx"]
        alias: {
        	"@": path.resolve(__dirname,"./src/components")
    	}
    }
}
```





### 5. Plugin插件

- Loader是用于特定的模块类型进行转换, 只是帮助我们转换解析文件
- Plugin作用于**更广泛的任务**, 比如打包优化, 资源管理, 环境注入

三个插件

- CleanWebpackPlugin: 自动删除打包前的旧的文件夹

```js
const { CleanWebpackPlugin } f= require("CleanWebpackPlugin")

module.exports = {
    plugins: [
        new CleanWebpackPlugin()
    ]
}
```



- HtmlWebpackPlugin: 打包文件夹下自动生成html文件, 应用方式于上面相同, 可以在new时进行配置 
  - 在html中可以利用`<%= htmlWebpackPlugin.options.title %>`来拿到定义的变量



- DefinePlugin

  - 拿到自己定义的变量, 允许我们在编译时创建配置的全局常量, 是一个内置插件

  ```js
  import { DefinePlugin } = require("webpack")
  module.exports = {
      plugins: [
          new DefinePlugin({
              "BASE_URL" : " './' "(里面的内容会默认执行, 因此字符串需要包裹)
          })
      ]
  }
  ```

  



### 6. mode

- 告诉webpack目前处于什么环境, 使用相应模式的内置优化
- 默认值时production
- 可选值有: "none" | "development" | "production"



## 二. webpack搭建本地服务器

### 1. 开启本地服务器

自动编译:

- webpack watch mode
- **webpack-dev-server(常用)**
- webpack-dev-middleware

```yaml
npm i webpack-dev-server -D
```

把打包后生成的文件放入到内存里, 而不是磁盘里, 不会输出文件

```js
{
    "scripts": {
        "serve": "webpack serve --config webpack.config.js"
    }
}
```



### 2. 模块热替换(HMR)

- 在应用程序运行过程中, 替换,添加,删除模块,而**无需刷新整个页面**

- 默认是开启的, 但需要指定哪一个模块需要HMR

```js
module.hot.accept("url",callback)
```

- 要防止文件之间的依赖对热更新产生影响, 从而导致刷新页面



### 3. host配置

```js
module.exports = {
    devServer: {
        hot: true,
        port: 8888,
        host: "IP地址" // 默认为127.0.0.1发出去的包,直接在网络层旧获取到了,不向下发送
    }
}
```

其他的三个重要配置:

- historyApiFallback 主要解决SPA页面在路由跳转之后, 进行页面刷新时, 返回404的错误, 默认为false, 如果为true,则在刷新返回404错误时, 自动返回index.html的内容

- Proxy也是开发中常用的配置选项, 目的是设置代理来解决跨域访问的问题
  - 在本地服务器和请求的目标服务器之间设置一个代理的服务器, 代理的服务器和目标服务器之间不会发生跨域问题
- changeOrigin: 修改代理请求中headers中的host属性