[toc]

# React-dom Component

## 一. createPortal

### 1. 定义

- `createPortal(children, domNode)`
  - `children`: jsx
  - `domNode`: `document.getElementById()`
  - 返回一个React Node, React在渲染中需要这个Node时, 会将`children`放到`domNode`中





### 2. 作用

- 将子组件渲染到DOM树上别的地方

```jsx
<div>
  <SomeComponent />
  {createPortal(children, domNode)}
</div>
```



## 二. flushSync

### 1. 定义

- `flushSync(callback)`
  - `callback`: 触发重新渲染的callback回调函数

### 2. 作用

- `flushSync(callback)`让你强制React同步刷新提供的callback回调函数中的所有更新, 确保DOM的更新是立即的
  - 因此只要其中的一个更新被`suspense`, 整个更新的性能就会受到很大的影响

```jsx
import { flushSync } from 'react-dom';

flushSync(() => {
  setSomething(123);
});
```



### 3. 注意点

- `flushSync`会很严重的破坏性能, 因此尽量别用
- 会触发`suspense`





## 三. createRoot

```html
<div id="root"></div>
```



```jsx
import { createRoot } from 'react-dom/client';
// spa
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

- 如果应用整体不是使用React开发的, 可以多次使用`createRoot`, 在想要使用的节点上调用

```jsx
root.unmount()  // 从DOM树上移除React树, 并清楚它所使用的所有资源
```





## 四. hydrateRoot

### 1. 定义

- `const root = hydrateRoot(domNode, reactNode, options?)`

### 2. 作用

- `hydrateRoot`让你在包含提前由`server`端渲染完成HTML的浏览器DOM节点中显示React组件,



- To hydrate your app, React will “attach” your components’ logic to the initial generated HTML from the server. Hydration turns the initial HTML snapshot from the server into a fully interactive app that runs in the browser.

```jsx
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);

```





- hydrate整个html

```jsx
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

