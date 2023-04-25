[toc]

# Nuxt3

- **Nuxt支持多种渲染模式: SSR, CSR, SSG**

- 服务器引擎:
  - 在开发环境中, 它使用Rollup和Node.js
  - 在生产环境中, 使用Nitro将应用程序和服务器构建到一个通用的.output目录中
    - **Nitro服务引擎提供了跨平台部署的支持**, 包括Node, Deno, Serverless, Workers等平台上部署

## 一. 新建项目和项目结构

### 1. 新建项目

- 使用`nuxi`脚手架

```shell
npx nuxi init [name]
pnpm dlx nuxi init [name]
npm install nuxi -g && nuxi init [name]
```



### 2. 项目脚本介绍

```json
{
    "scripts": {
    	"build": "nuxt build",    // 打包正式版本
    	"dev": "nuxt dev",		// 开发环境下运行
    	"generate": "nuxt generate",	// 打包正式版本, 并且提前预渲染每个路由 nuxt build --prerender
    	"preview": "nuxt preview",		// 打包项目之后的本地预览
    	"postinstall": "nuxt prepare"	// npm的生命周期钩子, 当执行npm install之后会自动执行, 生成.next和ts的类型等
  	},
}
```



### 3. 项目文件介绍

```
hello-nuxt        
├── README.md     
├── app.config.ts 
├── app.vue       
├── assets        
├── components    
├── composables   
├── layout        
├── nuxt.config.ts
├── package.json  
├── pages
│   └── index.vue 
├── plugins       
├── pnpm-lock.yaml
├── public        
└── tsconfig.json 
```



#### 1.app.vue(应用入口文件)

- 默认情况下, Nuxt会将这个文件视为入口点, 并用应用程序的每个路由呈现其内容
  - 定义页面布局Layout或自定义布局, 如: NuxtLayout
  - 定义路由的占位, 如:NuxtPage
  - 编写全局的样式
  - 全局的监听路由(路由守卫)



#### 2. nuxt.config.ts(nuxt配置)

- 可以对Nuxt进行配置, 一般如下

- 默认是对SSR下的应用进行配置

  - 可以配置属性`ssr: false`使其改为配置spa

  

##### 2.1 runtimeConfig

- 运行时配置, 即**定义环境变量**

- 可以通过.env文件中的环境变量来进行覆盖

```js
// 定义
export default defineNuxtConfig({
  runtimeConfig: {
    appKey: "zzt", // server
    public: {
      baseURL: "http://baidu.com"  // server + client
    }
  },
});
```

```js
// 获取
const runtimeConfig = useRuntimeConfig()
// 判断运行环境
if(process.server) {
    console.log(runtimeConfig.appKey)  // "zzt"
}
if(process.client) {
    console.log(runtimeCOnfig.appKey)  // undefined
}
```

##### 2.2 appConfig

- 应用常量配置, 定义在构建时确定的公共变量, 如: theme

- 支持抽取到一个文件中`app.config.ts`

```js
// app.config.ts 单独文件
export default defineAppConfig({
    title: "Hello World",
    theme: {
        primary: "yellow"
    }
})
```

```js
// 获取
const appConfig = useAppConfig()
console.log(appConfig.title)
```

##### 2.3 app

- 可以进行SEO优化

```js
// nuxt.config.js
// SEO优化 
// 方式一, 优先级最低
export default defineNuxtConfig({
    // 按照html模板里的内容编写
    app: {
        head: {
            title: "zzt",
            chartset: "UTF-8",
            viewport: " width=device-width ",
            meta: [
                {
                    name: "keywords",
                    content: "zzzz"
                },
                {
                    name: "description",
                    content: "zzzzzz"
                }
            ],
            link: [
                {
                    rel: "shortcut icon",
                    href: "favicon.icon",
                    type: "image/x-icon"
                }
            ]
        }
    }
})
```

```js
// 方式二, 优先级第二
// 动态的给所有的app页面添加html中head的内容
useHead({
    title: "app useHead",
    meta: [
        {
            name: "zzzz",
            content: "zzzzzzzz"
        }
    ]
})
```

```html
<!-- 方式三, 直接使用内置组件, 优先级最高 -->
<Head>
    
</Head>
<Meta name="keyword" content>
```



## 二. Nuxt3内置组件

- **SEO组件:**Html, Body, Head, Title, Meta, Style, Link, NoScript, Base

- **NuxtWelcome**: 欢迎页面组件, 该组件是@nuxt/ui的一部分

- **NuxtLayout**: 是Nuxt自带的页面布局组件

- **NuxtPage:** 是Nuxt自带的页面占位组件

  - 对路由`<router-view>`的封装
  - 需要显示位于目录中的顶级或嵌套页面pages/, 会根据目录结构和文件的命名来配置路径

- **ClientOnly:** 

  - 该组件中的默认插槽的内容只在客户端渲染
  - fallback插槽的内容只在服务器端渲染

  ```html
  <!-- 服务器端渲染完成就会切换到div -->
  <ClientOnly>
      <div>
          只在客户端渲染
      </div>
  	<template #fallback>
      	<h2>
              服务器端正在渲染
          </h2>
      </template>
  </ClientOnly>
  ```

- **NuxtLink:**是Nuxt自带的页面导航组件
  - 由Vue Router的`<RouterLink>`组件和HTML的<a>标签的封装



## 三. 全局样式

### 1. 直接定义全局样式

```js
// nuxt.config.ts
// 配置全局样式
export default defineNuxtConfig({
    css: ["@/assets/styles/main.css"]
})
```



### 2. 手动导入

```vue
<script>
@use "~/assets/styles/variables.css" as vb  // 定义一个命名空间, 也可以使用*省略命名空间, 这样就能直接使用$fsColor

.local-style {
    color: vb.$fsColor
}
</script>
```



### 3. 自动导入

```js
// nuxt.config.js
export default defineNuxtConfig({
    vite: {
        css: {
            preprocessorOptions: {
                // 自动给scss模块添加额外的数据
                additionals: ['@use "@/assets/styles/variables.scss" as *']
            }
        }
    }
})
```



## 四. alias别名

```json
{
    "~~": "/<rootDir>",
    "@@": "/<rootDir>",
    "~": "/<rootDir>",
    "@": "/<rootDir>",
    "assets": "/<rootDir>/assets",
    "public": "/<rootDir>/public",
}
```



## 五. Nuxt Router

### 1. NuxtPage和NuxtLink

- **NuxtPage:** 是Nuxt自带的页面占位组件
  - 对路由`<router-view>`的封装
  - 需要显示位于目录中的顶级或嵌套页面pages/, 会根据目录结构和文件的命名来配置路径
- **NuxtLink:**是Nuxt自带的页面导航组件
  - 由Vue Router的`<RouterLink>`组件和HTML的<a>标签的封装
  - **a标签导航会触发浏览器的刷新事件, 但是NuxtLink不会, 页面导航会通过前端路由来实现**
  - ==NuxtLink的属性自行去官网查看==
- **也可以直接在命令行生成路由文件**
  - 一类是`.vue`文件
  - 一类是`文件夹 + index.vue`文件

```shell
npx nuxi add page [name]
npx nuxi add page login/index
```



### 2. 编程导航(navigateTo)

- **编程导航: navigateTo**, 通过编程导航可以在应用程序中轻松实现动态导航, **但编程导航不利于SEO**
- `navigateTo(to,options)`

```vue
<NuxtLink @click="gotoCategory">
	<button>
        category
    </button>
</NuxtLink>

<script setup>
function gotoCategory() {
    return navigateTo('/category')
}
</script>
```



### 3. 编程导航(useRouter)

- 跳转页面建议还是使用`navigateTo`
- **beforeEach:** 路由守卫钩子, 每次导航前执行(**用于全局监听**)

```js
const router = useRouter()
router.back()
router.go(1)
router.go(-1)
router.beforeEach((to,from) => {
    
})
```



### 4. 动态路由

- pages/detail/[id].vue    /detail/:id
- pages/detail/user-[id].vue   /detail/user-:id
- pages/detail/[role]/[id].vue   /detail/:role/:id
- pages/detail-[role]/[id].vue   /detail-:role/:id

```js
const route = useRoute()
// 1. 拿取动态路由参数
const { id } = route.params
// 2. 拿取查询路由参数
const { name } = route.query
```



### 5. 404 Page

- **捕获所有不匹配的路由:**
  - 通过在方括号内添加三个`.`, 如: `[...slug].vue`, slug也可以改为其他字符串
  - 除了支持在pages根目录下创建, 也支持在子目录下创建
  - 404页面也可以通过`useRoute().param.xxx`来获取路由参数