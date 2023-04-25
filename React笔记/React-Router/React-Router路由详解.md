[toc]

# React-Router路由详解

- 前往官网观看最新版本的React-Router

[Picking a Router v6.9.0 | React Router](https://reactrouter.com/en/main/routers/picking-a-router)

```js
// main.js
import * as React from "react";
import * as ReactDOM from "react-dom";
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

import Root, { rootLoader } from "./routes/root";
import Team, { teamLoader } from "./routes/team";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    loader: rootLoader,
    children: [
      {
        path: "team",
        element: <Team />,
        loader: teamLoader,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider router={router} />
);
```



## 一. 基本介绍

- 前端路由的核心:**改变URL, 但是页面不进行整体的刷新**



- URL的hash,==待学习==
  - URL的hash也就是锚点(#), 本质上是改变了window.loaction的href属性
  - 我们可以直接赋值location.hash来该百年href, 但是页面不发生刷新

```html
<a href="#/home">Home</a>
```



- HTML5的history, 它有六种方式改变URL而不刷新页面
  - replaceState: 替换原来的路径
  - pushState: 使用新的路径
  - popState: 路径的回退
  - go: 向前或向后改变路径
  - forward: 向前改变路径
  - back: 向后改变路径
- 使用完整的URL而不是hash(#)的好处: 
  - 更有益于SEO
  - 更有益于服务端渲染
  - 更适配web剩余的平台




## 二. React-router基本使用

```shell
npm install react-router-dom   # react-router包含react-native的内容, 对于web开发来说是多余的
```

- react-router最主要的API是给我们提供了一些组件
- BrowserRouter和HashRouter
  - Router中包含了对路径改变的监听, 并且会将相应的路径传递给子组件
  - BrowserRouter使用的是history模式
  - HashRouter使用的是hash模式

```jsx
// 主文件: index.js
import { HashRouter } from "react-router-dom"

root.render(
	<HashRouter>
    	<App />
    </HashRouter>
)
```

```jsx
// App.jsx

render() {
    return (
    	<div class="header">
        	<Header />
            <!-- 跳转 -->
            <Link to="/ome" ></Link>
        </div>
        
        <!-- 中间建立映射关系-->
        <Routes>
            <!-- 在之前的版本中为component, element需要直接传入一个实例 -->
        	<Route path="/home" element={ <Home /> } />
        </Routes>
        
        <div class="footer">
        	<footer />
        </div>
    )
}
```



## 三. 路由映射配置和跳转

目前用的是Router6.x的版本

**配置:**

- Routes: 包裹所有的Route, 在其中匹配一个路由
  - Router5.x使用的是switch组件
- Route: Route用于路径的匹配
  - path: 属性用于匹配路径
  - element: 用于匹配到路径后渲染的组件, 直接传入实例
    - Router5.x用的是component属性



**跳转:**

- Link和NavLink

  - 通常路径的跳转是使用link组件, 最终会被渲染成a元素
    - to属性是link最重要的属性, 设置跳转的路径

  ```jsx
  <Link to="/ome" ></Link>
  ```

  - NavLink是在link基础上增加了一些**样式属性**
    - style: 传入函数, 函数接收一个对象, 包含isActive属性
    - className: 传入函数, 函数接收一个对象, 包含isActive属性

```jsx
<NavLink to="/home" style={({isActive}) => ({ color: isActive ? "red" : "" })} />
```



## 四. Navigate

- Navigate用于路由重定向, 当这个组件**出现**时, 就会执行跳转到对应的to路径中

```jsx
{ isLogin ? <Navigate to="/home" /> : <button onClick={修改isLogin} >登录</button> }

<Route path="/" element={<Navigate to="/home" />} ></Route>

<Route path="*" element={<Notfound />} ></Route>
```



## 五. 子路由配置

- 配置子路由是为了让子组件显示在父路由的范围下
  - 父路由的显示范围由定义时的`<Routes></Routes>`决定
  - 或者由`useRoutes()`调用时的位置决定

- 子路由显示时, 父路由一定显示

<img src="C:\Users\zZOMZz\Desktop\Typora笔记\React笔记\图片\Snipaste_2022-12-28_22-29-05.png" alt="Snipaste_2022-12-28_22-29-05" style="zoom:50%;" />

### 1. 配置父路由children

```jsx
// App.js
 <Routes>
    <!-- 在之前的版本中为component, element需要直接传入一个实例 -->
    <Route path="/home" element={ <Home/> }>
    	<Route path="/home/recommed" element={ <Recommed/> }  />
    </Route>
</Routes>
```

### 2. 父路由界面配置Outlet占位符

```jsx
// Home组件
<div>
	<div>HHHH</div>
    <h2>UUUUU</h2>
    
    <!-- 占位符, 子路由的组件会被渲染在这 -->
    <Suspense>
    	<Outlet>
    </Suspense>
</div>
```

- 二级路由最好也添加一个`Suspense`, 避免在懒加载二级路由的时候, 调用的是以及路由的fallback, 使这个fallback的影响范围增大

## 六. 手动路由跳转(useNavigate Hook)

- 不止利用`<a><a/>`来跳转, 也可以使用别的任何标签

```jsx
import { useNavigate } from "react-router-dom"
export function App(props) {
    // hook只能放入顶层
    // useNavigate是一个hook函数, 只能在函数组件中被调用
    const navigate = useNavigate()
    
    navigateTo(path) {
        // 返回一个函数对象
        navigate(path)
    },
    render() {
        <button onClick={() => this.navigateTo("/home")} ></button>
    }
}
```

- 利用高阶组件对类组件进行增强, 在类组件外层嵌套一个函数组件， 将`navigate`这个函数对象通过props传入到类组件中

```jsx
function withRouter(WrapperComponent) {
    return function(props) {
        const navigate = useNavigate()
        const router = { navigate }
        return (
        	<WrapperComponent {...props} router={router} />
        )
    }
}
export default withRouter(Home)
```



## 七. 路由获取params

### 1. 动态路由的参数(useParams)

`/home/:id/:name`这种格式的url

- 需要先定义路径

```jsx
<Route  path="/home/:id" element={<Detail/>} />
```

- 在<Detail />这个页面内拿取到传入的id, 利用useParams这个hook

 `const params = useParams()`通过useparams这个hook拿取



### 2. 字符串的参数(useSearchParams)

`/user?name="zz"&age="18"`这种格式的url

- 利用`useSearchParams()`这个Hook

```jsx
// 这种方式是写死的
const [searchParams] = useSearchParams()
console.log(searchParams.get("name"))
console.log(searchParams.get("age"))
// 1.利用searchParams生成一个新的对象
const query = Object.fromEntries(searchParams)
// 2.再将query传入到类组件中
```





## 八. 分模块定义routes(useRoutes Hook)

- 分开像VUE一样定义好routes数组
- 再将routes数组直接在HTML中使用`useRoutes(routes)`进行调用

```jsx
import ...
import React from "react"
// 懒加载的文件会单独打包出来一个独立js文件
const Login = React.lazy(() => import("../pages/Login"))

const routes = [
    {
        path: "/"
        element: <Navigate to="/home" />
    },
    {
        path: "/home",
		element: <Home />
        children: [
        	{
        		path: "/home/recommend"
   			}
        ]
    }
]
```

```jsx
// App.js
import { useRoutes } from "react-router-dom"
import { routes } from "./routes"

{
    useRoutes(routes)
}
```

```jsx
// index.js为懒加载做一些配置
// fallback中的组件可以在异步组件的单独文件还没有被下载下来时, 进行显示
<HashRouter>
	<Suspense fallback={<H3>Loading...</H3>} >
    	<App />
    </Suspense>
</HashRouter>
```



## 九. 懒加载

```tsx
// router.ts
const Focus = lazy(() => import('@/views/focus'))
const Album = lazy(() => import('@/views/discover/c-views/album'))

// App.tsx
function App() {
  const dispatch = useDispatch<any>()
  useEffect(() => {
    dispatch(fetchCurrentSong(25906124))
  })

  return (
    <div className="App">
      <Header />
          <!-- 需要将需要懒加载的组件用Suspense包裹起来, 提供一个fallback  -->
      <Suspense fallback="loading">
        <div className="main">{useRoutes(Routes)}</div>
      </Suspense>
      <AppPlayerBar />
      <Footer />
    </div>
  )
}

export default App
```



```js
let route = {
  path: "projects",
  async loader({ request, params }) {
    let { loader } = await import("./projects-loader");
    return loader({ request, params });
  },
  lazy: () => import("./projects-component"),
};
```

