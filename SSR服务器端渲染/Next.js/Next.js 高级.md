[toc]

# Next.js 高级

## 一. 中间件(middleware)

- Next.js的中间件允许我们去**拦截**客户端发起的请求, 例如: API请求, router切换, 资源加载, 站点图标
  - 可以对请求和响应做一定的修改和验证



### 1. 创建中间件

1. 在**根目录下**创建**middleware.ts**文件
   1. 里面导出的函数会被自动添加
2. 从middleware.ts文件中导出一个中间件**middleware**函数(支持async, 并且只允许在服务器端), 接收的两个参数
   1. **req**: 类型为**NextRequest**
   2. **event**: 类型为**NextFetchEvent**
3. 通过返回NextResponse对象来实现重定向等功能
   1. next(): 继续执行中间件
   2. redirect(): 将重定向
   3. rewrite(): 将重写URL, 如配置反向代理
4. 没有返回值: 页面将按预期加载和返回next()一样



### 2. 匹配器(Matcher)

- 匹配器允许我们过滤中间件在特定路径上运行
  - matcher: '/about/:path*', 意思是匹配以`/about/*`开头的路径. 其中路径开头的`:`是修饰符, 而`*`代表0个或n个
  - `matcher: ['/about/:path*', '/dashbord/:path*']`, 意思是匹配以`/about/*`和`/dashbord/*`开头的路径



```ts
// middleware.ts
export const config = {
    // 匹配不包含_next路径
    matcher: ["/((?!_next).*)"]
}
```



### 3. 返回

- next(): **返回next和不返回的效果是一样的, 就是放行**
- redirect(): 进行重定向

```ts
export function middleware(req: NextRequest) {
    
    // 1. 返回next
    return NextResponse.next()
    
    // 2. 返回重定向
    const token = req.cookies.get("token")?.value
    if(!token && req.nextUrl.pathname !== "login") {
        return NextResponse.redirect(new URL("/login", req.nextUrl.origin))
    }
    
    // 3. 进行重写, 拦截对外部的请求
    if(req.nextUrl.pathname.startsWith("/juanpi/api")) {
        return NextResponse.rewrite(
        	new URL(req.nextUrl.pathname,"http://codercba.com:9060")
        )
    }
}
```



### 4. 使用Cookie

```shell
npm i cookies-next
```

```tsx
import { setCookie } from "cookies-next"

function login() {
    setCookie('token',"sdasd", {
    	maxAge: 60          
    })
}
```



## 二. 布局(Layout)

- Layout布局是页面的包装器, 可以将**多个页面的共性东西**写到Layout布局中, 使用**props.children**属性来显示页面的内容
  - 例如可以将每个页面的头部和尾部写入到Layout布局中



### 1. 简单使用

- 使用步骤:
  - 在`components`目录下新建`layout.tsx`布局组件
  - 在`_app.tsx`中通过`<Layout>`组件包裹`<Component>`组件

```tsx
// components/layout/index.tsx

const Layout = function(props) {
    // 1. 拿到children
    const { children } = props
    
    return (
    	<div>
        	<div class="header">Header</div>
            
            <!-- 2. 填入页面的内容 -->
            { children }
            <div class="footer">footer</div>
        </div>
    )
}
```

```tsx
// 使用布局
import Layout from '@/components/layout'

export default function App({Component, ...rest}: AppProps) {
    
    // 1. 判断不同的页面是否需要这个布局
    if(Component.displayName === "cart") {
        return <Component />
    }
    
    
    return (
        <Layout>
            <!-- Component会作为一个children传入到layout中 -->
            <Component />
        </Layout>
	)   
}
```

### 2. 嵌套布局

```tsx
// components/nest-layout/index.tsx
const NestLayout = function(props) {
    const { children } = props
    
    return (
    	<div>
        	<div class="header">Header</div>
            
            <!-- 2. 填入页面的内容 -->
            { children }
            <div class="footer">footer</div>
        </div>
    )
}
```

```tsx
return (
        <Layout>
        	<NestLayout>
            	<!-- Component会作为一个children传入到layout中 -->
            	<Component />
            </NestLayout>
        </Layout>
)   
```



## 三. API接口

- 可以自己编写可调用的接口

```ts
// pages/api/user.ts
// 这个接口既能接收get方法, 也能接收post方法
import type { NextApiRequest, NextApiResponse } from 'next'

export default function handler(req: NextApiRequest, res: NextApiResponse) {
    const userInfo = {
        
    }
    
    res.status(200).send(userInfo)
}
```



## 四. 预渲染

- 默认情况下, Next.js会预渲染每个页面, 即预先为每个页面生成HTML文件, 而不是由客户端JavaScript来完成
- 预渲染可以**带来更好的性能和SEO优化**
- 当浏览器加载一个页面时, 页面依赖的JS代码就会被执行, 执行JS代码会激活页面(Hydration), 使页面具有交互性



- 两种形式的预渲染:
  - **静态生成**(更推荐): HTML在构建时生成, 并在每次页面请求(request)时重用
  - **服务器端渲染**: 在每次页面请求时, 重新生成HTML页面



### 1. SSG(静态生成)

- 如果一个页面使用了**静态生成**, 在**构建时**将生成此页面对应的HTML文件, 这意味着在生产环境中, 运行next build 时将生成该页面对应的HTML文件, 然后这个HTML文件将在每个页面请求时被重用, 还可以被CDN缓存



- 可以将静态页面分为**带有或者不带有数据的页面**
  - 不需要外部数据的页面, 直接正常构建就行, 构建完成后的文件会在服务器端存储着
  - **需要获取外部数据进行预渲染**
    - 情况一: 页面的**内容**取决于外部数据, 使用`getStaticProps`函数
    - 情况二: 页面的**路径**取决于外部数据(**动态路由**), 使用`getStaticPaths`函数(通常还需要`getStaticProps`), (**生成页面的数量**)
  - **在`npm run build`时, 会自动执行`getStaticProps`拿到数据进行预渲染**



```tsx
// /pages/books-ssg/index.tsx
export default function BookSSG(props) {
    const children = { props }
    
    return (
    	<div>
        	
        </div>
    )
}

// 名字必须为getStaticProps, 返回的props会传入到上方的函数组件的props中
// 会在build阶段就执行, 拿到数据, 进行预渲染
export async function getStaticProps(context: any) {
    // 进行一些网络请求拿到数据
    
    return {
        props: {
            
        }
    }
}
```

```tsx
//  /pages/books-ssg/[id].tsx
export default function BookSSG(props) {
    const children = { props }
    
    return (
    	<div>
        	
        </div>
    )
}

export const getStaticPath: GetStaticPath = async (context) => {
    
    const ids = res.data.books.map(item => {
        return {
            params: {
                id: item.id + ""
            }
        }
    })
    
    return {
        paths: ids || [],
        fallback: false   // 如果动态路由的路径没有匹配上, 则返回404页面
    }
}

export const getStaticProps: GetStaticProps = async (context) => {
    // 会依次拿到paths的全部对象
    console.log(context.params?.id)  // 拿到getStaticPath传来的ids其中的一个id
    // 发送网络请求
    
    return {
        props: {
            book: res.data.book
        }
    }
}
```



#### 1.1 使用场景:

- 用户在请求之前就可以预渲染页面, 那么就可以选择静态生成, 反之就不合适

- 如果页面要显示经常更新的数据, 并且**页面的内容会在每次请求时发生变化**, 这时就可以选择:
  - 静态生成(SSG)和客户端数据获取渲染(CSR)结合使用
    - **可以跳过预呈现页面的某些部分, 然后使用客户端JS来填充它们, 但是客户端渲染(在useEffect中获取数据, 在客户端动态渲染页面)是不适合SEO的优化的**

- 服务器端呈现
  - Next.js会根据每个请求**预呈现**一个页面, 缺点是稍微慢一点, 因为页面无法被CDN缓存, 但是预渲染的页面始终都是最新的



### 2. 服务器端预渲染(SSR)

- 如果页面使用的是服务器端渲染, 则会在每次页面请求时重新生成页面的HTML
- 要对页面使用服务器端渲染, 需要export一个`getServerSideProps`的async函数
  - **服务器将在每次页面请求时调用此函数, 而不是只在打包时才调用**
  - **只在服务器端运行, 不在浏览器端运行**



- **当某个页面需要==预渲染==频繁更新的数据(从外部API获取)**, 你就可以编写**getServerSideProps**获取该数据并将其传递给Page的props

```tsx
export const getServerSideProps: GetServerSideProps = async (context) => {
    console.log(context.query)
}
```



#### 2.1 使用场景

- 当页面显示的数据**必须在请求时获取的**, 才会使用`getServerSideProps`
  - 如: 页面需要显示**经常更新的数据**, 并且页面内容会在每次请求时发生变化
- 如果页面使用了`getServerSideProps`函数, 那么该页面将会在客户端请求时, 会在服务器端进行渲染, 页面默认不会缓存
- 如果不需要在客户端每次请求时获取页面数据, 那么应该考虑在**客户端动态渲染(CSR, 在useEffect中请求渲染)或`getStaticProps`**



### 3. 增量静态再成(ISR)

- Next.js除了支持静态生成和服务器端渲染, Next.js还允许在构建网站后创建或**更新静态页面**

- 比如: 在`getStaticProps`的例子中, 使用ISR渲染模式, **让服务器每隔5s重新生成静态书籍列表页面**

```tsx
export const getStaticProps: GetStaticProps = async (context) => {
    // 会依次拿到paths的全部对象
    console.log(context.params?.id)  // 拿到getStaticPath传来的ids其中的一个id
    // 发送网络请求
    
    return {
        props: {
            book: res.data.book
        },
        // 设置间隔时间
        revalidate: 5  // in seconds
    }
}
```



### 4. 客户端渲染(CSR)

- **在客户端获取数据**, 需要在页面组件或普通组件的**useEffect**函数中获取
- **不利于SEO**



## 五. 集成Redux

### 1. 安装依赖

- 安装`next-redux-wrapper`
  - 可以避免在访问服务器端渲染页面时store的重置
  - 可以将服务器端redux存的数据, **同步一份到客户端上**
    - 当用户访问动态路由或后端渲染的页面时, 会执行Hydration来保持两端数据状态一致
  - 该库提供了**HYDRATE调度操作**
    - 比如: 当用户每次打开了使用`getStaticProps`或`getServerSideProps`函数生成的页面时, HYDRATE将执行调度操作

- 安装`@reduxjs/toolkit react-redux`

```shell
npm i next-redux-wrapper
npm i @reduxjs/toolkit react-redux
```



### 2. 使用

```ts
// 定义slice
import { HYDRATE } from "next-redux-wrapper"

const homeSlice = createSlice({
    name: "home",
    initialState: {
        counter: 10
    },
    reducers: {
        
    },
    extraReducers: (builder) => {
        // Hydrate的操作, 保证服务器端和客户端的数据的一致性
        builder.addCase(HYDRATE, (state,{ payload }) => {
            return {
                ...state,
                ...payload.home
            }
        })
    }
})
```

```ts
// 定义store
import { createWrapper } from "next-redux-wrapper"

const store = configureStore({
    reducer: {
        home: homeReducer
    }
})

const wrapper = createWrapper(() => store)
export default wrapper
```

```tsx
// 使用store
export default fucntion App({ Component, ...rest }: AppProps) {
    const { store, props } = wrapper.useWrappedStore(rest)
    return (
        <Provider store={store} >
            <Layout>
                <Component { ...props.pageProps } />
            </Layout>
        </Provider>
    )
}
```

