[toc]

# Vue3 + SSR

## 一. 简单介绍

- 在Vue中创建SSR应用, 需要调用`createSSRApp`函数, 而不是`createApp`
  - `createSSRApp`: 创建应用, 在激活模式(SSR)下挂载应用
- 服务器端用`@vue/server-renderer`包中的**renderToString**来进行渲染

![Snipaste_2023-01-29_20-55-17](.\图片\Snipaste_2023-01-29_20-55-17.png)

## 二. Node Server搭建

```shell
# 需要安装的依赖
npm i express
npm i nodemon -D
npm i webpack webpack-cli webpack-node-externals 
```

```js
// webpack-node-externals的作用
// server.config.js
const nodeExternals = require("webpack-node-externals")
module.exports = {
    target: "node"  // 针对node环境
    externals: [nodeExternals()]  // 排除node_modules中的包
}
```

```js
// 开启node服务器, 可以使用webpack将其打包(--watch开启监听模式), 再用nodemon运行打包后的文件
const express = require("express")
import createApp from '../app'
import createRouter from '../router'
import { renderToString } from '@vue/server-renderer'

const app = express()

app.use(express.static("build"))  // 对打包好的文件进行静态资源部署

app.get("/*",async (req,res) => {
    // 1. 通过函数生成新的app, 避免跨请求状态污染
    const app = createApp()
    // 2. 通过函数生成新的router
    const router = createRouter(createMemoryHistory())
    app.use(router)
    await app.push(req.url || '/')
    await router.isReady()  // 等待异步路由加载完毕
    
    // 3. 安装pinia
    const pinia = createPinia()
    app.use(pinia)
    
    // 返回静态html页面, 无法进行交互
    const appStringHTML = await renderToString(app)
    res.send(
    	`
    		<div id="app">${appStringHTML}</div>
    		<script src="/client/client_bundle.js"></script>   // 会去部署的静态资源中寻找
    	`
    )
})

app.listen(3000,() => {
    console.log("服务器开启监听")
})
```



## 三. Vue代码

```shell
npm i vue # 下载vue
npm i vue-loader babel-loader @babel/preset-env # 为webpack下载loader
```

```js
import { createSSRApp } from 'vue'

export default function createApp(App) {
    const app = createSSRApp(App)
    
    return app
}
```

### 1. hydration

```js
// 客户端代码
import { createApp } from 'vue'
import App from '../App.vue'

const app = createApp(App)
app.mount('#app')
```

```js
// client.config.js
module.exports = {
    target: "web"
}
```

```js
// 再将打包后的客户端文件通过script的脚本, 添加到HTML静态文件中
```



### 2. 跨请求状态污染

- 在SPA中, 整个生命周期中只有一个App实例对象或一个Router实例对象或一个Store实例对象都是可以的, **因为每个用户在浏览器访问SPA应用时, 应用模块都会被重新初始化, 这是一种单例模式**
- 然而, 在SSR环境下, App应用模块通常只会在服务器启动时初始化一次. 同一个应用模块会在多个服务器请求之间被复用, 而我们的单例状态对象也一样, 也会在多个请求之间被复用
  - 因此当别的用户对共享的单例状态进行修改的时候, 这个状态可能会意外的泄露给另一个在请求的用户
  - 这种情况称为: **跨请求状态污染**
- **解决办法**
  - 在每个请求中为整个应用单独创建一个实例, 包括Router和Store实例
  - 我们创建App或路由或Store对象都使用一个函数来创建, 确保每个请求都会创建一个全新的实例
- **缺点:**
  - 更消耗服务器资源



### 3. Router

- **需要为创建router封装一个函数, 避免跨请求污染**

- 服务器端需要配置
- 客户端也需要配置

```js
// 客户端代码
import { createApp } from 'vue'
import App from '../App.vue'

const app = createApp(App)
const router = createRouter(createWebHistory())
app.use(router)
router.isReady().then(() => {
    app.mount('#app')
})
```



### 4. Pinia

- **为了避免跨请求污染, 也需要单独封装一个函数**
- 在hydration的时候, 会将服务器端Pinia同步到客户端Pinia中