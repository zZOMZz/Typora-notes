[toc]

# React Hook

## 一. Hook解决的问题

### 1.原函数式组件的缺陷:

- 修改状态时难以重新渲染, 重新执行会重新初始化
- 不存在生命周期钩子



- 因此Hook可以保存下原来的状态, 并且进行重新刷新



### 2. Class类组件存在的问题

- 随着业务的增多, 类组件内的逻辑会越来越复杂
  - 在`componentDidMount`中包含大量的逻辑代码: 包括网络请求和一些事件的监听, 同时在`componentWillUnmount`中移除
  - 这样发展下去, 类组件会变得越来越难以拆分, 它们的逻辑往往混在一起, 强行拆分会造成过渡设计, 增加代码的复杂度
- 难以理解的class和this指向问题
- 组件复用困难



### 3. Hook的好处

- 它可以让我们在不编写class的情况下使用state以及其他的React特性
- 基本能替代class
- Hook只能在函数式组件内使用, 不能在函数式组件之外的地方使用



## 二. Effect Hook

### 1. 使用场景

- Effect让你指定由渲染本身引起的副作用, 而不是通过一些特定的event. **Effect在渲染完成, 屏幕更新后运行**, 这是将**React组件与外部系统(网络或者第三方库)同步**的好时机
- Effect是用于"跳出"React代码, 与外部系统达成同步的
- 类似网络请求, 手动更新DOM, 事件监听, 都是react更新DOM的一些**副作用**(对本组件之外的组件或state状态产生了影响)
- **所有的Effect只会在客户端被调用, 在服务器端渲染的时候不会触发**



```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  // 不能直接在rending阶段操纵DOM, 这会造成不纯, 比如在严格模式下, 两次调用可能结果不同
  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}
```

- 并且在第一次调用这个函数组件的时候, **这个组件的真实DOM并不存在**, 它会被调用先生成虚拟DOM, 再生成真实DOM, 再将真实DOM下的`video标签的引用`注入`ref.current`
- 被包裹在`useEffect Hook`中的代码会被移出渲染代码之外, 在渲染结束后(返回JSX, 形成真实DOM)再运行



### 2. 接收参数

`useEffect(callback,dependencies)`两个参数

- `callback`: 执行的回调函数
- `dependencies`: 所依赖的`props`和`states`, 在与上一次渲染不同的情况下, 才会重新执行
  - 如果传入的是一个空数组, 即不受任何参数的影响, 只会在初始渲染的时候执行
  - 如果不传, 则会每次渲染都调用, 易造成死循环
  - 如果有依赖, 但不传, 则会报错
  - **使用的dependencies在一次render中也是快照, 无法改变**

```js
// 两次渲染dependencies的比较是通过Object.is()这个静态方法
Object.is([], []); // false
Object.is({},{})  // false

// true 才会不触发更新, 因此[], 在dependencies中是唯一特殊的存在
```

- **因为ref(useRef), 对React来说是一个无法追踪的神秘口袋, 确保了不变性`{ current: null }`, 因此不必添加到依赖中**, 当然从父组件传入的ref除外, 因为在当前这个组件无法"看到", 这个ref是否是不变的



### 3. cleanup useEffect

```jsx
 useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
 }, []);
```

- 连续调用这个函数组件中的`useEffect`前, 会先调用前面的返回的`cleanup function`
- 或者在这个函数组件`unmounted`时

```jsx
// cleanup function 的一些使用场景
// 1. 一些特别的API不允许被连续调用两次(development下的一种React检测机制), 这时也需要cleanup function
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);

// 2. 在useEffect中订阅了某些东西, 那么在cleanup函数中, 需要取消订阅
useEffect(() => {
  function handleScroll(e) {
    console.log(e.clientX, e.clientY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);

// 3. 触发动画
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);

// 4. 异步获取数据时
// 由于请求发出去之后无法直接取消, 因此可以通过下面这种方式, 使其在unmounted的状态下也不会影响后续的渲染
// 在有缓存的情况下, 这种做法可以提高性能
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```



### 4. 在useEffect中直接获取数据的缺点

- 在服务器端不起作用, 在SSR服务器端生成的HTML中无法获得数据
- 容易造成"网络瀑布", 父组件获取数据生成子组件, 然后子组件再接力继续发出请求, 在较慢的网速中, 这种影响是灾难性的
- 在`useEffect`中获取数据, 意味着你没有任何**缓存机制**, 这样每次当依赖发生变化时, 你都不得不重新运行`useEffect`, 进而重新发送网络请求



### 5. 改善

- 使用现代框架, 一些现代的React框架中内置了数据获取机制, 不会产生上面这种影响
- 使用客户端缓存技术, 现在流行的缓存解决方案 [React Query](https://tanstack.com/query/latest), [useSWR](https://swr.vercel.app/), 和 [React Router 6.4+.](https://beta.reactrouter.com/en/main/start/overview) , 你也可以自己想办法解决



### 6. useEffectEvent

- 当你要在`useEffect`中使用某个props或state, 但你又不想因为它们发生改变而重新调用`useEffect`
- 即在`useEffect`中接收一个`event handle`而不用添加额外的依赖, **当你需要在`useEffect`中使用`event handle`时可以用这个Hook**

```jsx
// 这种情况下, 只有在roomId改变时才会重新调用useEffect, 但是仍能通过event的方式, 调用最新的theme, 而不用以它为依赖
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

