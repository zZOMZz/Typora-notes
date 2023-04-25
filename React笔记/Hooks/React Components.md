[toc]

# Components

## 一. Fragment

与vue的template相似, 会把其中的元素包裹起来, 但是在最终**无须向DOM添加额外节点**, 也不会被渲染出来

- 语法糖`<>  </> `省略其中的文字, 需要绑定属性, 如key的情况下, 不能使用这种语法糖



## 二. StriceMode 

- StriceMode 是一个用来突出显示应用程序中潜在问题的工具
  - 它与Fragment一样, 不会渲染任何可见的UI
  - 它为后代元素触发额外的检查和警告
  - 严格模式检查仅在开发模式下运行, 它们不会影响生成和构建
- 可以为应用程序的任何部分启用严格模式
  - 不会对Header和Footer组件运行严格模式检查
  - 但是,ComponentOne和ComponentTwo 以及它们的所有后代元素都将进行检查

```jsx
<StriceMode>
// 给其中元素的所有子元素和后代开启严格模式
</StriceMode>
```

- 识别一些不安全的生命周期
- 过时的ref API 如`this.refs`
- 检查的副作用:
  - 这个组件的constructor会被调用两次, 这是在严格模式下的故意的操作, 用来检测是否有一些逻辑代码被重复调用
  - 在生成环境下不会被调用两次
- 使用一些过期的API
- 检测过时的context的API
- **严格模式会强制把生命周期如`useEffect`执行两次, 检查副作用**



## 三. Suspense

### 1. 简单介绍

- 在渲染期间显示别的组件的一种手段
- **Suspened状态会被最近的Suspense捕获并显示其fallback, 不会再往上传**

```jsx
// 在Chat未渲染完成时, 会显示Loading组件
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>

// 也可以这样嵌套使用
// 这里会先显示BigSpinner, 然后是Biography + AlbumsGlimmer, 最后是BigSpinner + Panel
 <Suspense fallback={<BigSpinner />}>
    <Biography artistId={artist.id} />
    <Suspense fallback={<AlbumsGlimmer />}>
      <Panel>
        <Albums artistId={artist.id} />
      </Panel>
    </Suspense>
</Suspense>
```



### 2. 防止隐藏已经生成的内容

- 如下面的代码中, `Router`组件整个被`Suspense`包裹, 如果仅仅因为`content`变化等造成重新渲染, 那么整个组件都会被`fallback: <BigSpinner />`代替, 即使原组件中已经渲染出的

```jsx
export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  let content = .....
  return (
     <>
       <h2>zzzzzz</h2>
       <Layout>
         {content}
       </Layout>
      </>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}

```



**解决办法:**

- 在改变的`content`外包裹一个`Suspense`捕获这个`Suspensed`状态, 避免触发包裹整个组件的`Suspense`
- 使用`startTransition`
  - 告诉react支持的state变化并不紧急, 最好保持之前的页面, 避免隐藏已经显示的内容, 它不会保持或等待特别长的时间, 仅仅确保已经显示的内容不会被隐藏

```jsx
function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);      
    });
  }
  // ...
```



- 使用`useTransition`