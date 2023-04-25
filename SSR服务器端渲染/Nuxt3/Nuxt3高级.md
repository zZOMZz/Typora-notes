[toc]

# Nuxt3高级

## 一. 路由中间件

- Nuxt提供了一个可定制的**路由中间件**, 用来监听路由的导航, 包括: 局部和全局监听(支持在服务器端和客户端执行)



- 1.**匿名(内联)路由中间件**

  - 在页面中使用**`definePageMeta`**函数定义, 可监听局部路由. 当注册多个中间件的时候, 会按照注册顺序来执行

  ```js
  definePageMeta({
      middleware: [
          function(to,from) {
              
              // 如果返回的是navigateTo, 则不会执行下一个中间件
          }
      ]
  })
  ```

- 2.**命名的路由中间件**

  - **提高复用性**
  - 刷新浏览器的时候会在server端执行
  - 在客户端切换路由时, 不会在server端执行

  ```tsx
  // middleware/home.ts
  export default defineNuxtRouteMiddleware((to,from) =>{
      
  })
  ```

  ```js
  // 这样进入这个页面时就会执行这个中间件
  definePageMeta({
      middleware: [
          "home",
      ]
  })
  ```

- 3.**全局路由中间件(优先级更高, 支持两端)**

  - 避免手动在每个页面中添加

  ```js
  // middleware/auth.global.ts
  export default defineNuxtRouteMiddleware((to,from) =>{
      
  })
  ```

  

- 4.**路由验证(validate)**

  - Nuxt支持对页面路由进行验证, 通过`definePageMeta`中的`validate`属性来对路由进行验证

  ```js
  // 需要进行路由验证的页面(验证参数)
  definePageMeta({
      validate: (route) => {
          return /^\d+$/.test(route.params.id)
          // 可以自行配置错误页面
          return {
              statusCode: 401
          }
      }
  })
  ```

  ```vue
  // 顶层文件error.vue
  // 这个页面会自动当做验证错误返回页面
  <script>
  	const props = defineProps({
          error: Object
      })
      
      function goHome() {
          // 先清除掉错误信息, 再返回首页
          clearError({ redirect: "/" })
      }
  </script>
  ```

  

## 二. 布局(layout)

### 1. layouts/default.vue

```vue
<template>
	<div class="header"></div>
	<slot></slot>
	<div class="footer"></div>
</template>
```



### 2. app.vue

- `NuxtPage即路由占位符中的内容会自动填入default.vue中的slot中`
- 通过`name`切换使用的布局

```vue
<NuxtLayout name="default">
	<NuxtPage></NuxtPage>
</NuxtLayout>
```



### 3. 自定义单独页面布局

```vue
<script>
definePageMeta({
    layout: "custom-layout"
})
</script>
```



## 三. 渲染模式

- 浏览器和服务器都可以解释JavaScript代码, 都可以将Vue.js组件呈现为HTML元素, 此过程称为渲染

  - 在客户端将Vue.js组件呈现为HTML元素, 称为: 客户端渲染模式
  - 在服务器将Vue.js组件呈现为HTML元素, 称为: 服务器渲染模式

- Nuxt3支持多种渲染模式

  - 客户端渲染模式(CSR): 只需在配置中将ssr改为false
  - 服务器端渲染模式: ssr设置为true
  - 混合渲染模式: 需要routeRules根据每个路由动态配置渲染模式

  ```js
  export default defineNuxtConfig({
      routeRules: {
          "/": { ssr: true },
          "/cart": { static: true }
      }
  })
  ```





## 四. Nuxt插件

- 创建插件的两种方式:

  - 使用**useNuxtApp()**中的provide(name,value)方法直接创建
    - useNuxtApp提供了访问Nuxt**共享运行时上下文**的方法和属性(两端可用): provide, hooks, callhook, vueApp

  ```js
  // 1. 拿到运行时上下文
  const nuxtApp = useNuxtApp()
  // 2. 定义一个插件
  nuxtApp.provide("formData", () => {
      return ".."
  })
  // 3. 使用插件
  console.log(nuxtApp.$formData())
  ```

  - **在plugins目录中创建插件**

    - 顶级和子目录index文件写的插件会在创建Vue应用程序时自动加载和注册
    - **在创建vue实例时就会自动注册好**

    ```js
    // plugins/[filename].ts
    export default defineNuxtPlugin((nuxtApp) => {
        
        return {
            provide: {
                formPrice: () => {
                    
                },
            }
        }
    })
    ```

    



## 五. 生命周期(Lifecycle)

- **注意分辨客户端生命周期钩子和服务器端生命周期钩子**

### 1. App Lifecycle Hooks

```js
// plugins/[filename].ts
//
export default defineNuxtPlugin((nuxtApp) => {
    nuxtApp.hook("app:rendered", () => {
    
	})
	nuxtApp.hook("vue:setup", () => {
    
	})
})
// 如果利用useNuxtApp()来在别的.vue文件中监听生命周期, 那么它是从setup生命周期开始
```



### 2. 组件生命周期

```vue
<script setup>
onBeforeMount(() => {
    
})
onMounted(() => {
    
})
    
</script>
```



## 六. 发送网络请求

- **`$fetch(url,options)`**来发送请求
  - 客户端和服务器端都会发送请求, 会重复两次
- **useAsyncData**:
  - 会阻止页面导航
  - 发起异步请求需要$fetch
- **useFetch**
  - 一种简写的语法糖
  - 会阻塞页面导航
- **useLazyFetch**
  - 不会阻塞页面导航
  - 但不确定什么时候能拿到数据， 需要用watch监听

```vue
<script>
// 1. 直接使用$fetch, 会发送两次请求
$fetch(BASE_URL, {
    method: "GET"
}).then((res) => {
    console.log(res)
})
// 2. 使用hook, 只在服务器端发送请求
const { data } = await useAsyncData("homeInfo", () => {
    return $fetch(BASE_URL+'/homeInfo', { method: "GET" })
})

// 3. 简写
const { data } = await useFetch(BASE_URL + "homeInfo", { method: "GET" })
</script>
```

```js
// 添加拦截器
const { data } = await useFetch("/homeInfo", {
    method: "GET",
    baseURL: BASE_URL,
    
    // 添加拦截器
    onRequest({request,options}) {
        
    },
    onRequestError({request,options,error}) {
    
	},
    onResponse({request,response,options}) {
        
    },
    onResponseError({}) {
        
    }
})
```

- **useFetch默认会阻塞页面的导航**

  - 只有当usefetch拿到数据后才会才会开始挂载渲染页面

  - 需要将options中`lazy: true`
  - 可以使用watch监听数据变化再赋值利用, **使用`useLazyFetch替代useFetch`**

```js
// 刷新
const { data, refresh, pending } = await useFetch()

function refreshPage() {
    // 1. client刷新请求
    refresh()
    // 2. 请求的body中包含响应式的数据, 修改该响应式的数据会重新发送请求
}
```



## 七. Server API

- **可以自己编写接口, 连接数据库**



- server/api/homeInfo.get.ts , get请求

```js
// server/api/homeInfo.get.ts   文件夹格式不能变server/api
export default defineEventHandler((event) => {
    const { req, res } = event.node
    // ... 判断用户名和密码, ...连接数据库
    
    
    return {
        code: 200,
        data: {
            
        }
    }
})
// 访问接口: host/api/homeInfo
```

- post请求:

```js
export default defineEventHandler(async (event) => {
    const { req, res } = event.node
    // 通过全局的方法拿到post数据
    const query = getQuery()
    const method = await getMethod(event)
    const body = await readBody(event)
    
    
    return {
        code: 200,
        data: {
            
        }
    }
})
```



## 八. 全局状态共享

### 1. useState

- Nuxt跨页面, 跨组件全局状态共享可使用`useState`(支持Server和Client)
  - **使用`useState<T>(key:string, init?: () => T | Ref<T>): Ref<T>`**
  - `init`: 为状态提供**初始值的函数**, 该函数也支持返回一个Ref类型
  - `key`: 唯一key, 确保在跨请求获取该数据时, 保证数据的唯一性, **不输入时, 会根据文件和行号自动生成唯一的key**

```js
// composables/useCounter.ts, 会自动全局导入
// 定义
export default function() {
    return useState("couter", () => 100)
}

export const useCounter = () => {}
```

```js
// 拿取
const counter = useCounter()
```



### 2. Pinia

```shell
npm install @pinia/nuxt pinia
```

- `@pinia/nuxt`会处理state同步问题(client和server), 比如不需要关心序列化或XSS攻击等问题

```js
// nuxt.config.ts
export default defineNuxtConfig({
    // 配置nuxt3的扩展库
    modules: ["@pinia/nuxt"]
})
```

```vue
<script>
const homeStore = useHomeStore()
const { counter } = storeToRefs(homeStore)
function addCounter() {
    homeStore.increment()
}
</script>
```



## 九. 安装Element Plus 2

- **具体的操作在官网查看**



- 第一步: 安装依赖

```shell
npm i element-plus
npm i unplugin-element-plus -D
```

- 第二步: 配置Babel对Eelemt Plus 的转译; 配置自动导入样式

```js
import ElementPlus from "unplugin-element-plus"
export default defineNuxtConfig({
    // babel转译
    build: {
        transpile: ["element-plus/es"]
    },
    // 自动导入样式
    vite: {
        plugins: [ElementPlus()]
    }
})
```

- 第三步: 在组件中导入组件

```js
import { ElButton } from "element-plus"
```

