[toc]

# Webpack开启本地服务器

## 一. webpack搭建本地服务器(devserver)

- devServer的原理是使用Express启动一个HTTP服务器, 编译代码并提供静态资源, 实现实施重载, 模块热替换和代理等功能. 从而方便开发者进行开发和调试



```json
// 编写脚本
"scrips": {
    "serve": "webpack serve"
}

// npm run serve 开启本地服务
```

- **webpack-dev-serve在编译之后不会写入任何输出文件, 而是将bundle文件(打包生成的文件)保留在内存中(使用了memfs库)**



### 1. 开启本地服务器

自动编译:

- webpack watch mode
- **webpack-dev-server(常用)**
- webpack-dev-middleware

```shell
npm i webpack-dev-server -D
```

把打包后生成的文件放入到内存里, 而不是磁盘里, 不会输出文件

```js
// package.json
{
    "scripts": {
        "serve": "webpack serve --config webpack.config.js"
    }
}
```



### 2. 模块热替换(HMR)

- 在应用程序运行过程中, 替换,添加,删除模块,而**无需刷新整个页面或重新启动应用程序**

- 使用了Webpack的`Hot Module Replacement`插件, 这个插件会在webpack打包时生成额外的运行时代码, 通过这些代码监听模块的变化, 当模块发生变化时, 将更新的模块代码通过webpack的`devServer`发送到浏览器, 然后在浏览器中使用新的模块代码替换掉就的模块代码, 实现热替换

```js
// 1. 配置插件
const webpack = require('webpack');

module.exports = {
  // ...
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
};

```

```js
// 2. 启用热替换
const webpack = require('webpack');

module.exports = {
  // ...
  devServer: {
    hot: true
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
};
```

```js
// 3. 代码中进行模块热替换
// module.hot === true 表示当前模块支持热替换
if (module.hot) {
  // 监听模块的变化, 当模块发生变化时, 执行之后的回调
  module.hot.accept('./module', () => {
    // 模块更新后的处理逻辑
  });
}
```





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



## 二. devServer的作用

### 1. 启动服务器

- 使用Express启动一个HTTP服务器, 监听指定的端口, 等待请求



### 2. 编译代码

- 使用webpack编译项目代码, 生成编译后的资源文件

### 3. 提供静态资源

- 将编译后的资源文件提供给客户端访问, 例如: HTML文件, CSS文件, JavaScript文件, 图片, 以及一些字体等文件

### 4. 实现实时重载

- 在开发模式下, 一旦源代码发生变化, 就会重新编译代码, 并将编译后的结果发送到浏览器中, 浏览器会自动刷新页面

### 5. 实现模块热替换

- 部分替换发生改变的模块代码

### 6. 实现代理

- devServer可以将指定的HTTP请求代理到另外一个服务器, 通过这种方式解决跨域

解决步骤:

1. 

```js
// 1. 设置devServer中的proxy属性
devServer: {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',  // 代理目标地址
      changeOrigin: true,  // 开启跨域
      pathRewrite: {
        '^/api' : ''  // 去掉请求前缀
      }
    }
  }
}

```

2. 

```js
// 2. 发起请求时, 将请求地址设置为代理的地址
fetch("/api/users")
```

3. 之后devServer会将所有以`"/api"`开头的请求转发设置到代理目标地址(target): 这里设置为`http://localhost:3000`, 从而实现跨域请求, 在请求返回时, 浏览器会自动将结果返回给发起请求的页面



## 三. 处理静态资源

- 静态资源: 不需要动态生成的文件, 这些资源在访问时, 返回的内容都是固定的, 不会动态发生改变(100次访问, 返回的结果都是相同的)
- 使用`webpack-dev-server`时, devServer中存储的是loader处理后的静态资源, 当devServer启动后会将处理后的静态资源加载到内存中, 并在内存中生成一个虚拟的文件系统. 由于是已经通过loader处理过的, 因此浏览器可以访问识别

```html
<!-- 当html引入如.js, .css,.img等静态资源时 -->
<script src='./content/test.js' ></script>
```

- webpack默认是不会对这些文件进行打包的(public文件夹下的文件除外), **需要去devServer下的static中配置(默认只包含了public文件夹)**

```js
// webpack.config.js

module.exports = {
    devServer: {
        // contentBase: path.resolve(__dirname, 'dist') 
        // contentBase已经在webpack5中被替换为static
        static: path.resolve(__dirname, 'dist')
    }
}
```



## 三.  host, port, opne, compress

**host**

- 默认值是localhost
- 如果希望其他地方也可以访问, 可以设置为0.0.0.0



- localhost本质上是一个域名, 会被解析成127.0.0.1(回环地址), 自己发出去的包会在网络层直接就被自己获取到了
- 0.0.0.0: 监听IPV4上的所有地址, 再根据端口找到不同的应用程序, 在同一网段下的在同一网段下的主机中, 可以直接通过IP地址访问(可以在同网段下的手机中访问)



**port**

- 配置开启服务的端口, 尽量在1024以上, 避开一些特殊端口



**opne**

- 编译成功后是否自动打开浏览器



**compress**

- 是否为静态文件开启gzip compression(文件的压缩), 默认为false



## 四. Proxy代理

- **target**: 表示代理服务器寻找资源的地址, 如`/api/moment`请求会被代理到`http://localhost:9000/api/moment`
- **pathRewrite**: 上面的例子中如果我们不希望`/api`被写入到寻找资源服务器的地址中, 可以使用pathRewrite
- **changeOrigin**: boolean, 它表示是否更新代理后请求的headers中host地址, 使代理服务器发送的请求和资源服务器更符合资源服务器的规范header, 修改host地址
  - 假设代理服务器的地址为`http://localhost:8888`, 他发送的请求的headers中的host也是这个地址, 如果不修改可能会被`http://localhost:9000`的服务器屏蔽, 修改了就能使两者保持一致

```js
module.exports = {
    devServer: {
        proxy: {
            '/api': {
                target: "http://localhost:9000",
                pathRewrite: {
                    '^/api': ''  // 重写路径
                }
            }
        }
    }
}

axios.get('/api/users/list').then(res => {})

// Proxy会在/api前, 添加上http://loaclhost:9000, 由于这里api可以看成是一个触发代理的关键词, 服务器的接口只是/users/list, 因此还需要重写路径, 将^/api写为空字符串
```

- **通过配置Proxy会构建一台node服务器, 充当本地服务器和资源服务器之间的枢纽, 由于跨域问题只存在于浏览器中, node服务器之间是不会发送跨域问题的**



## 五. historyApiFallback

- **解决SPA页面通过`HTML5 History API`进行路由跳转之后, 进行页面刷新时, 返回404的错误**
  - 比如在`http://localhost:3000/about`这个路由下进行刷新, 它会自动请求服务器中about文件夹下的index.html, 但是这里about只是一个路由, 因此在找不到html文件的情况下, 就会报404的错误.
  - 解决办法是通过配置, 使其返回目标文件, 可以通过`rewrites`属性进行进一步控制
- 因此让它在刷新后重定向到`http://localhost:3000`这个地址, 把后面的`/about`当成是history的一部分

- `historyApiFallback: true`

- 底层是用[connect-history-api-fallback](https://github.com/bripkens/connect-history-api-fallback)这个第三方库实现的

```js
module.exports = {
    devServer: {
        historyApiFallback: {
            rewrites: [
                { from: /^\/$/, to: '/views/landing.html'  },
                { from: /^\/subpage/, to: '/views/subpage.html' },
                { from: /./, to: '/views/404.html' }
            ]
        }
    }
}
```

