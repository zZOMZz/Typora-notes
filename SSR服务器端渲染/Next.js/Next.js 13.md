[toc]

# Next.js 13

- **Next.js是一个React框架, 支持CRS, SSR, SSG,ISR等渲染模式**

## 一. 创建项目

```shell
# 方式一
npx create-next-app@latest --typescript
# 方式二
npm i create-next-app@latest -g && create-next-app 
# 方式三
pnpm create-next-app  -typescript
```



## 二. 环境变量

- 使用`.env`文件中定义环境变量
  - 如果需要客户端也可以访问需要以`NEXT_PUBLIC`开头

```yaml
# 服务器端访问
NAME = localhost
PORT = 8888
# 客户端访问
NEXT_PUBLIC_HOSTNAME = 
```

```js
// 服务器端访问
console.log(process.env.NAME)
```

- `.env.development`: 开发时环境变量, 执行`next dev`时加载并生效
- `.env.production`: 生产时环境变量, 执行`next start`时加载并生效
- `.env.local`: **始终覆盖上面文件定义的默认值**. 所有环境生效, 通常只需一个.env.local文件(**常用于存储敏感信息**)

- **也可以直接在next.config中配置env属性(该方法优先级最高)**



## 三. next.config

常见的属性配置:

- **reactStrictMode**: 是否启用严格模式
- **env**: 配置环境变量, 也是被添加到`process.env`
- **basePath**: 在域名的子目录下部署Next.js应用程序
  - basePath: 允许为应用程序设置URL前缀
  - 如: `basePath = /music`, 即用/music访问网页, 而不是默认的/
- **images**: 可以设置图片URL的白名单等信息
- **swcMinify**: 用Speedy Web Compiler编译和压缩技术, 而不是Babel + Terser技术



## 四. 内置的组件

常见的内置组件:

- **Head**: 用于将新增的标签**添加到页面的head标签**中, 需要从next/head导入
  - 可以进行一些SEO优化
  - 只在当前页面有效, 如果所有页面都要进行SEO优化, 可以单独封装一个组件





## 五. 全局和局部的样式

### 1. 全局的样式:

- 在styles文件夹下编写`globals.css`, 任何在`_app.tsx`中引入



### 2. 局部的样式

- Next.js是默认支持**CSS Module**, 在`styles`文件夹下编写如: `home.module.css`
- 在需要引入的模块中导入
- `scss`文件中可以使用`@use '' as *`导入需要复用的样式

```tsx
import styles from './index.module.scss'

return (
	<div className={styles["local-style"]} ></div>
)
```

```scss
// 必须放到css module文件中(及模块化的index.module.scss), 不能放到普通的scss文件中
:export {
    primaryColor: $primaryColor
}
```



## 六. 静态资源的使用

- **public目录常用于存放静态资源**, public中的资源如: `me.png`, 可以直接通过静态URL**/me.png**访问
- 也可以自己创建`assets文件夹`, 通过背景图片的方式引入. `background: url(~/assets/index.png)`



## 七. Router(约定式路由)

- Next.js项目页面需在**pages目录**下新建(**.js .jsx .ts .tsx**), 该文件需要导出React组件
- **预定义(确定的)路由优先于动态路由, 动态路由优先于捕获所有的路由([...slug] 404页面)**



- Next.js和Nuxt3一样, 也支持嵌套路由(只在app目录下), **但是Next.js嵌套的路由自动生成后还是属于一级路由(即子路由无法称为父路由的一部分, 只能全部替代)**
  - 方案一: 使用Layout布局嵌套来模拟
  - 方案二: 使用Next.js13 新增的app目录

### 1. 目录结构

- **具有约定式路由**
  - `pages/index.jsx ` -> `/`
  - `pages/about.jsx`  -> `/about`
  - `pages/blog/index.jsx`    -> `/blog`
  - `pages/blog/post.jsx`  -> `/blog/post`  (嵌套路由)
  - `/pages/blog/[slug].jsx`  -> `/blog/:slug ` (动态路由)
  - `/pages/blog/[role]/[id].tsx` -> `/blog/:role/:id`

### 2. 填充占位

- **不需要使用占位组件**
  - 在`_app.tsx`中的`<Component {...pageProps} />`就是默认占位符



### 3. 404页面

- **方式一**: 捕获所有不匹配的路由
  - **[...slug].tsx, 如果在子目录下, 仅作用于该目录以及子目录 **, 比如:`/pages/post/[...slug].js`只匹配`/post/a   /post/a/b `,无法匹配`/post`



### 4. 解决二级路由的问题(app目录)

![Snipaste_2023-01-30_23-24-22](C:\Users\zZOMZz\Desktop\Typora笔记\SSR服务器端渲染\图片\Snipaste_2023-01-30_23-24-22.png)

```tsx
// app/profile/layout
export default function ProfileLayout({children}) {
    return (
    	<div>
        	<div>Head</div>
        	{ children }
        </div>
    )
} 
```

- 如果profile的子路由login没有自己的layout, 就会沿用副路由下的layout, 因此就能解决问题, 实现部分显示





## 八. 组件导航(Link和useRouter)

### 1. Link

- 页面之间的跳转需要用到`<Link>`, 需要从next/link包导入
- **Link组件属性**
  - **href**(不支持to)
    - 字符类型: "/", "/home"
    - 对象类型: { pathname: "/", query: {} }
    - URL: 外部网站
  - **as**
    - 在html中隐藏真实的url
    - `as="profile_v2"`



### 2. useRouter

- 通过**编程导航**可以更加轻松的实现动态导航, **缺点是不利于SEO优化**

- router对象常用方法
  - push(url): 页面跳转
  - replace(url, [options]): 页面跳转(会替换当前页面)
  - back(): 页面返回
  - event.on(name,func): 客户端路由的监听(建议在_app.tsx监听, 建议放在useEffect()副作用钩子中)
- **`router.query`: 不但能拿到查询字符串(query)的参数, 还可以拿到动态路由(params)的参数**
  - 如果名字重复, 会优先使用params的参数, 查询字符串的参数就会丢失



```tsx
import { useRouter } from "next/router"


useEffect(() => {
    router.event.on("routerChangeStart",(url) => {
        
    })
})
```

