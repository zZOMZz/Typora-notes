[toc]

# React-Router(v6)最新笔记

## 一. createBowserRouter

- 摒弃传统的使用`<BowserRouter/>`和`HashRouter`这种组件式写法, 采用API的形式来完成简洁明了

```ts
function createBrowserRouter(
  routes: RouteObject[],
  opts?: {
    basename?: string;
    window?: Window;
  }
): RemixRouter;
```

```js
const router = createBrowserRouter([
  {
    // it renders this element
    element: <Team />,

    // when the URL matches this segment
    path: "teams/:teamId",

    // with this data loaded before rendering
    loader: async ({ request, params }) => {
      return fetch(
        ``/fake/api/teams/${params.teamId}.json``,
        { signal: request.signal }
      );
    },

    // performing this mutation when data is submitted to it
    action: async ({ request }) => {
      return updateFakeTeam(await request.formData());
    },

    // and renders this element in case something went wrong
    errorElement: <ErrorBoundary />,
  },
]);
```



## 二. RouterProvider

- 渲染利用API产生的`router`

```jsx
<RouterProvider
  router={router}
  fallbackElement={<SpinnerOfDoom />}
/>
```

```jsx
import {
  createBrowserRouter,
  RouterProvider,
} from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Root />,
    children: [
      {
        path: "dashboard",
        element: <Dashboard />,
      },
      {
        path: "about",
        element: <About />,
      },
    ],
  },
]);

ReactDOM.createRoot(document.getElementById("root")).render(
  <RouterProvider
    router={router}
    fallbackElement={<BigSpinner />}
  />
);
```





## 三. Route

- 可以写成JSX的写法`<Route/>`, 但更推荐写成对象的形式
  - 如果是JSX的形式, 在传入到`createBrowserRouter()`前, 需要先传入` createRoutesFromElements()`,在将结果传入`createBrowseRouter`

- 各个属性的用法自己阅读文档[Route v6.9.0 | React Router](https://reactrouter.com/en/main/route/route#index)

```ts
interface RouteObject {
  path?: string;
  index?: boolean;
  children?: React.ReactNode;
  caseSensitive?: boolean;
  id?: string;
  loader?: LoaderFunction;
  action?: ActionFunction;
  element?: React.ReactNode | null;
  Component?: React.ComponentType | null;
  errorElement?: React.ReactNode | null;
  ErrorBoundary?: React.ComponentType | null;
  handle?: RouteObject["handle"];
  shouldRevalidate?: ShouldRevalidateFunction;
  lazy?: LazyRouteFunction<RouteObject>;
}
```





## 四. deferred Data

- 可以在`route`改变的时候就发出数据请求, 而不是等component运行时, 特别是lazy载入
- 它可以让页面渲染的更快, 但会加大内容布局转移(CLS): 内容发生布局变化, 产生视觉不稳定性, 用户感觉到页面的跳动或闪烁 

```js
async function loader({ request, params }) {
  const packageLocationPromise = getPackageLocation(
    params.packageId
  );
  const shouldDefer = shouldDeferPackageLocation(
    request,
    params.packageId
  );

  return defer({
    packageLocation: shouldDefer
      ? packageLocationPromise
      : await packageLocationPromise,
  });
}
```

```js
export default function PackageRoute() {
  // 获取在loader中定义的异步 缓慢数据
  const data = useLoaderData();

  return (
    <main>
      <h1>Let's locate your package</h1>
      <React.Suspense
        fallback={<p>Loading package location...</p>}
      >
        <Await
          resolve={data.packageLocation}
          errorElement={
            <p>Error loading package location!</p>
          }
        >
          <PackageLocation />
        </Await>
      </React.Suspense>
    </main>
  );
}

function PackageLocation() {
  const packageLocation = useAsyncValue();
  return (
    <p>
      Your package is at {packageLocation.latitude} lat and{" "}
      {packageLocation.longitude} long.
    </p>
  );
}
```

