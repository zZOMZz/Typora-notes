[toc]

# Next.js 文档笔记

## 一. Server component和client component

### 1. fetch

- 在`Server Component`中fetch操作会自动去重



### 2. 何时使用client Component和server Component

- `server component`由于是在服务器渲染的因此能获得更好的性能, 且能直接访问一些后端资源, 但由于不是在浏览器(客户端)渲染, 没有DOM和BOM, 因此无法交互
  - 在server端渲染使用的依赖, 如果客户端不需要, 则不会打包传输
- `client component`为了性能, 最好尽量少用client component, 只有当需要**交互**或者**副作用**或者**组件需要缓存状态**时才使用. `client component`能在server服务器端预渲染, 然后在客户端通过hydration激活使用

![image-20230411014831763](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230411014831763.png)



### 3. 混合使用server component和client component

- 能在server component中使用client component
  - 在必要的需要交互等的组件使用客户单渲染

![image-20230411020530576](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230411020530576.png)





- 不能在client component中直接导入使用server component. 因为server component中可能会有一些私密代码(访问数据库等)
  - 可以将server component作为props或者children传入到client component使用
  - 也可以在另外一个server component中包裹这个client component和server component以达成嵌套使用



### 4. server-only code

- 有些代码最好只在服务器端运行, 比如数据获取和数据库查询

```js
export async function getData() {
  let res = await fetch("https://external-service.com/data", {
    headers: {
      authorization: process.env.API_KEY,
    },
  });

  return res.json();
}
```

- `process.env.API_KEY`只在server的环境中定义了, 因此在client端会变成空字符串, 导致这个数据获取函数失效, 但用户不一定能察觉
- 为了解决这个问题, 可以使用`server-only`第三方库来检查保证, 只需要在相应文件`import "server-only"`就行, 不需要额外的配置



### 5. 第三方库解决server和client问题

- 有些第三方组件库想要在server端运行, 但是可能在其内部使用了`useState`等不能在server端使用的hook, 可以通过手动导入的方式, 来手动为它们分类

```js
'use client';

import { AcmeCarousel } from 'acme-carousel';

export default AcmeCarousel;
```



### 6. context

- context和state状态有关, 因此只能在client component中使用, context相关的API都能在客户端正常使用



## 二. Static and Dynamic Rendering

### 1. Static Rendering(Default)

- Static Rendering: 在静态路由中, 组件在构建时(build)就已经生成好, 构建(build)之后就不会发生改变, 因此工作的结果也能被缓存, 并在后续的请求中重复使用
- Static Rendering能提升应用的性能, 因为所有的渲染都在构建(build)阶段完成了, 并且能把结果缓存到CDN(Content Delivery Netwoek)中



### 2. Static Data fetching(Default)

- 默认情况下, Next.js会缓存未明确选择退出缓存行为的`fetch()`请求的结果, 即未设置缓存选项的fetch请求将使用force-cache选项
- 如果路由中的任何fetch请求使用revalidate选项, 则在重新验证期间将静态重新呈现该路由



### 3. Dynamic rendering

- Dynamic rendering: 具有dynamic functions和dynamic data fetching, 在动态渲染中, 组件会在客户端请求时再渲染生成组件
- 在static rendering中, 如果next.js发现了dynamic function 或者dynamic `fetch()`请求(没有缓存), next.js就会在请求这个route时切换到dynamic rendering, 被缓存的data request在dynamic rendering中能被重复使用



### 4. Dynamic Functions

- Dynamic Functions依赖只有在用户请求之后才能获取的和渲染相关的数据, 比如用户的cookies, requests headers, URL's search params



- 应用场景
  - 在server component中使用`cookies()`和`headers()`, 会使整个route在请求时变为动态渲染
  - 在client component中使用`useSearchParams()`会跳过静态渲染, 并暂时渲染展示最近的Suspense边界
    - 推荐当使用`useSearchParams()`时用`<Suspense/>`边界包裹住, 则允许任何在其上面的客户端组件能被静态渲染. 因为suspense会返回一个fallback, 而不是将包含整个组件的组件也变为动态渲染

- 使用`searchParams`Page prop也会在请求时转为动态渲染



### 5. Dynamic Data Fetches

- Dynamic data fetch是将缓存设置为`no-store`或将`revalidate`设为0, 以明确选择退出缓存行为的`fetch()`行为
- 在`layout`和`page`页面中的所有`fetch`请求的缓存选项也可以使用segment config对象进行设置



