[toc]

# React Hook APIs

## 一. useRef

**作用:**

- 可以让组件"记住"某些信息, 但又不想触发新的渲染
- 一般将`ref`视为一种应急方案, 如果你想改变某个值(不是渲染在页面上的), 而不触发渲染, 那么可以考虑`useRef`, 如: 清楚某个定时器
- 获取DOM的操作权



### 1. ref的简单使用

1. 通过useRef来定义一个值, 再将这个值赋值给element元素的ref属性, 从而绑定DOM, 根据这个值拿到DOM上的属性

```jsx
// 定义
const ref = useRef(0)
// 返回 {　current: 0 } 这个对象

// 绑定一个值
const titleRef = useRef()
(<h2 ref={titleRef} ></h2>)
```



### 2. ref = 秘密"口袋"

2. useRef 在组件多次渲染时, 返回的是同一个值,**可以将其返回`ref.current`看成是以个对于React来说无法追踪,用来存储组件信息的秘密"口袋"** , 因此可以保存一个数据, 这个ref对象在整个生命周期中都可以保持不变, 组件也**不会因为ref的改变而重新渲染**

```jsx
// 随着按钮的点击, alert窗口显示的ref.current会递增, 但按钮上显示的值一直是0, 因此可以证明ref.current的值改变了, 但没有触发渲染, ref.current对React来说是一个秘密的口袋, 即使整个组件用某种方式触发了重新渲染, ref.current这个口袋里的值依旧保留着 
export default function Counter() {
  let ref = useRef(0);
  let i= 0;

  function handleClick() {
    ref.current = ref.current + 1;
    alert('你点击了 ' + ref.current + ' 次！');
  }

  return (
    <button onClick={handleClick}>
      {ref.current}
    </button>
  );
}
```



### 3. ref 的一些使用场景

- 需要跳出React, 与外部的API进行通信时使用

- 存储`timeout ID`

- 存储和操作DOM元素

  - `useRef` Hook 返回一个对象，该对象有一个名为 `current` 的属性。最初，`myRef.current` 是 `null`。**当 React 为这个 `<div>` 创建一个 DOM 节点时**，React 会把对该节点的引用放入 `myRef.current`。然后，你可以从 [事件处理器](https://beta.react.jscn.org/learn/responding-to-events) 访问此 DOM 节点，并使用在其上定义的内置[浏览器 API](https://developer.mozilla.org/docs/Web/API/Element)。

  ```jsx
  // 绑定一个值
  const myRef = useRef(null)
  myRef.current.focus()  // 调用API. 获得焦点
  
  return (<div ref={myRef} ></div>)
  ```

  

- 存储不需要被用来计算JSX的其他对象
- 需要存储一些值, 但不影响渲染逻辑

```jsx
const countRef = useRef()
countRef.current = count
const increment = useCallback(() => {
    setCount(countRef.current + 1)
}, [])
```

- 同时可以将ref向下传(props), 让子组件可以调用父组件的DOM 方法; 也可以对父组件暴露(利用`useImperativeHandle`), 让父组件调用其DOM 方法, 比如改变CSS等

```jsx
// 这样父组件就可以通过子组件的ref, 调用这个focus方法, childRef.current.focus
const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus，没有别的
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});
```



​	

### 4. useRef的原理

```jsx
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

- 如果赋给`useState`的是一个普通的数字或者字符串时, 即使ref在一次渲染中改变了值(不使用`set`函数), 但由于**快照**的存在, 它在下一次渲染中还是会将一切变化归0, 按照快照的值来渲染
- **但是对象是不同的**, 因为这里是`{ current: initialValue }`这个对象, 因为我们只改变了其中`current`的值, 没有改变这个对象, 因此从对象的角度来看, 对这个对象的**快照**并没有改变, 改变前后都是同一个对象, **因此对`current`的改变会记录到下一次渲染中**



### 5. 更新refs的时间

React的更新分为两个步骤:

- **渲染:** 调用组件(调用函数), 确定画面上该显示什么内容
- **提交:** 把React的变更应用于真正的DOM

React在提交阶段设置`ref.current`, 在更新DOM之前, React将受到影响的`ref.current`值设为`null`, 更新DOM后, React会**立即将对应的DOM节点的引用值注入到`ref.current`中**, 因此ref的值是在函数调用完成后, 由虚拟DOM生成真实DOM之后才有的

```jsx
// 所以无法直接在函数组件顶层内调用ref.current上的操作
fucntion app() {
    const videoRef = useRef()
    
    // error, 因为在调用ref的时候还是渲染阶段, DOM的引用还没被注入
    ref,current.play()
}
```



### 6. 优化初始值

- 尽管`useRef(null)`的初始值只在第一次渲染时起作用, 在之后的渲染中都会忽略它, 但是如果其中的是一个函数, 那么每次渲染还是会重新调用这个函数

```jsx
// 可以这么优化
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```









## 二. useImperativeHandle

### 1. 定义

- `useImperativeHandle(ref, createHandle, dependencies?)`
  - `ref`: 从`forwardRef`中接收的`ref`
  - `createHandle`: 一个不接受参数的函数, 返回你想要暴露的`ref handle`
  - optional `dependencies` : `createHandle`中所有的`Reactive`的值, 如果`dependencies`前后发生了改变, 或者没有定义`denpendencies`, 呢么每次渲染都会重新执行`createHandle`这个函数, 返回新的handle到ref上
- `Return` 返回`undefined`

### 2. 作用

- 子组件对父组件传入的ref进行处理, **权限控制**, 只允许父组件调用子组件主动暴露出的DOM操纵API
  - 函数式组件想要接收ref这个props需要通过`forwardRef`来进行增强
    - `(props,ref)`
- 你也可以在暴露出去的API中自定义一些操作, 以达到你想要的效果

```jsx
// 子组件
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```



### 3. 建议

- 不要在代码中过度使用`refs`, 能用`props`代替的地方, 尽量用`props`代替. 通过在`useEffect`中利用`props`进行判断, 再进行DOM操作

```jsx
useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
});
```



## 三. useLayoutEffect

### 1. 定义

- `useLayoutEffect(setup, ?dependices)`: 与`useEffect`类似, 但是是在**浏览器**重绘页面之前被调用的



### 2. 作用

- 在浏览器重绘之前测量布局(layout)
  - 有时你需要根据一些实际的参数, 来改变渲染的布局

```jsx
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // You don't know real height yet

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // Re-render now that you know the real height
  }, []);

  // ...use tooltipHeight in the rendering logic below...
  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    // 第一次渲染中, 会默认将其放置在当前元素之上
    tooltipY = targetRect.top - tooltipHeight;
    // 如果上面塞不下, 则将其放到元素的下方
    if (tooltipY < 0) {
      // It doesn't fit above, so place below.
      tooltipY = targetRect.bottom;
    }
  }
}
```

**具体步骤:**

1. `Tooltip`使用`tooltipHeight = 0`来进行渲染
2. React将其渲染出来放入DOM中, 并且运行`useLayoutEffect`中的代码
3. `useLayoutEffect`中测量实际的高度, 并触发重新渲染
4. React使用真实的`heigh`进行渲染, 并将结果更新到DOM中
5. 最终浏览器将组件显示在页面上



### 3. 注意点

- 由于`useLayoutEffect`是在浏览器重绘之前被调用的, 因此, 此时真实DOM已经形成了
- 在`useLayoutEffect`中的代码会**阻塞**浏览器的重绘, 因此当过度(excessive)使用时, 会导致整个应用变得非常缓慢(slow), 因此可能的话, 尽可能使用`useEffect`, `useEffect`不会阻塞浏览器
  - 上面的例子中, 如果你使用的是`useEffect`, 则它会在绘制完成后, 再次触发`useEffect`中的`set`函数, 造成再一次的渲染绘制, 因此, 在较慢的设备上, 你可能会看到这个组件突然**闪烁一下**, 你甚至可以看到两次绘制的效果, 但在`useLayoutEffect`中, 你永远只会看到一次, 因为它会阻塞浏览器绘制



## 四. useImmer

[GitHub - immerjs/use-immer: Use immer to drive state with a React hooks](https://github.com/immerjs/use-immer)

[React & Immer | Immer (immerjs.github.io)](https://immerjs.github.io/immer/example-setstate)

### 1. 作用

- 为了符合不变性(immutable)的要求: 将一个`state`看作**只读**的, 修改state只能通过`set`函数
- 但是为了保持不变性, 每次为了更新需要把之前整个对象不需要的部分都拷贝一份, 如果对象嵌套很多层, 则会非常复杂, 因此可以使用这个hook构造一个不变的对象或者数组, **直接在上面进行更改, 后续会自动将这些更改应用于state上**

```jsx
import { useImmer } from 'use-immer';

const [myList, updateMyList] = useImmer(
    initialList
  );

// 不需要返回, 对draft的所有修改都会应用到state上
updateMyList(draft => {
  const artwork = draft.find(a =>
    a.id === id
  );
  // 因为这里修改draft中某一项执行的对象, 等同于修改了draft
  artwork.seen = nextSeen;
});
```

### 2. 原理

- `useImmer`中的`draft`使用了Proxy技术, 因此你可以随意在上面进行修改, 它会记住你所有的修改, 在函数结束后, 克隆出一个全新的对象, 并将其设置到`state`上



## 五. useId

### 1. 定义

- `const passwordId = useId()`, 根据`parent path`生成唯一的`id(string)`



### 2. 作用

- 可以用于生成横跨服务器端和客户端的唯一ID,可以保证应用程序在客户端和服务器运行时生成唯一的ID,  同时避免hydration不匹配的, 但是如果**服务端和客户端生成的树结构不相同**, 那么生成的id也会不同
  - 如果使用那种递增的id, 无法保证`hydrated`的客户端组件的顺序能正确匹配服务端生成的HTML发出的组件

- **不能用于生成list的keys**, 返回的值与在这个特殊的组件中调用的useId有关

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
```

```jsx
// 因为一个页面中可能在很多地方都使用了这个组件, 因此为了保持id的唯一性, 可以使用useId这个hook
function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}
```





## 六. useTransition

### 1. 定义

- `const [isPending, startTransition] = useTransition()`, 在不阻碍更新UI的情况下, 更新state
  - `isPending` flag : 告诉你当前是否有一个代办的过渡(pending transition)
  - `startTransition` function: 让你能将一个状态(state)更新标记(mark)为过渡(transition)

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```



### 2. 作用

- 将一个`state`状态更新标记为一个非阻塞转换(non-blocking transition)
  - `Transitions` 让你的UI在重新渲染的过程中也**保持响应性**, 如: 如果用户点击了一个tab, 马上又点击了另一个tab, 这种情况下就不需要等待第一次的重渲染结束
  - 缓慢的重新渲染不会阻塞UI(user interface), 即你在渲染一个繁重的任务的时候, 如果点击渲染了另一个任务, 那么界面会及时的响应
  - 在网速和性能较差时, 可以提高用户体验
  - **因此可以理解为, 你将一次触发重新渲染的操作, 包裹在返回的`startTransition`函数中, 会使组件在这次的重新渲染中依旧保持响应性**
  - 原因是因为. React的调度器采用了"时间切片"这种并发模式
- 可以通过`ispending`在渲染过程中, 显示等待中这个状态
- 可以通过`useTransition`阻止不想要的Suspense(挂起), 再通过`ispending`显示自己想要的效果

### 

### 3. 注意点

- `startTransition`函数不能直接与`input`的`value`联系在一起, 因为`input`的`value`需要时`synchronize`同步的, 而`startTransition`转换的值是非阻塞的
- 不能在`startTransition`中调用异步操作
- `startTransition`只是将更新操作标记为`transition`, 并不是想`setTimeout`推迟整个回调函数的调用时间

```jsx
let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... schedule a transition state update ...
  } else {
    // ... schedule an urgent state update ...
  }
}
```





## 七. useDeferredValue

### 1. 定义

- `useDeferredValue(value)`: **推迟**一部分UI的更新

```jsx
const deferredValue = useDeferredValue(value)
```

### 2. 作用

- 当新的内容还在加载时, 先显示旧的内容, 新的内容会在后台渲染, 当后台渲染完成后, 再将其转换回来
  - **后台渲染是可以被打断的**, 因此当你一直改变`deferred`的值时, 渲染会暂停, 然后直接跳到最后一步
- 可以作为一种**性能优化**的方案, 将需要花比较长时间重渲染的组件放在后台渲染, 防止阻碍其他部分的更新
  - 否则一次性更新整个树, 会使某些方面(如: 在input打字输入)体验糟糕, 这种情况下`useDeferredValue`允许React先**优先于更新该更新的地方**, 更新的慢的地方优先级降低, 放到之后再更新
  - 但是子组件**必须使用`memo`包裹起来**, 负责即使前后传入两个相同的值, 还是会触发重新渲染

```jsx
// 原本每次当SearchResults的props query改变时, 组件都会重新渲染, 从而会短暂的显示fallback
// 但这里由于使用了useDeferredValue会在更新的值load之前一直显示旧的值, 因此不会产生Suspensed, 所以不会短暂显示fallback
// 会显示旧值渲染, 然后直接转变为新的值渲染
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

```jsx
// 可以通过css的方式, 让用户直观的感受组件还在渲染中
<div style={{
  opacity: query !== deferredQuery ? 0.5 : 1,
}}>
  <SearchResults query={deferredQuery} />
</div>
```



### 3. 思考: 和节流和防抖的关系

- 相同
  - 都是不直接显示当前的值, 有一定的延迟
- 不同: 
  - `useDeferredValue`相对防抖和节流, 不需要设定一个固定的时间, 并且它与React深度融合, 因此能更直观的显示渲染的速度, 适应用户的设备和网络情况
  - 节流和防抖可以避免多次重复发送网络请求, 但`useDeferredValue`做不到



## 八. useDebugValue

- 让你可以在`React DevTools`中向自定义Hoos添加`label`

- `useDebugValue(value, format?)`
  - `value`: 想要在`React DevTools`中显示的值
  - 可选的`format`: 一个格式化函数, 当组件被检查时, 会将`value`值作为函数输入值, 最后像素返回的值
- `useDebugValue`这个Hook实际上并不做任何事

```jsx
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```



## 九. useInsertionEffect

### 1. 定义

- `useInsertionEffect(setup, dependencies)`: 是一种版本的`useEffect`, 它在DOM变化之前运行
  - `setup`: 和`useEffect`类似, 但是是在组件被添加到DOM之前形成的
  - `dependencies`: 和`useEffect`的类似, `props, state, and all the variables and functions declared directly inside your component body`



### 2. 作用

- 在`css-in-js`库中, 动态注入`style`
  - React官方更推荐: 静态样式的 CSS 文件，动态样式的内联样式, 不推荐在运行时注入`<style>`标签, 但是如果你不得不做, 最好使用`useInsertionEffect`
- 在注入`styles`方面`useInsertionEffect`表现的比`useLayoutEffect`和`useEffect`更好, 因为它保证了当别的`effect`运行的时候, `<style>`标签已经被注入了, 不会出现过时的style这个问题



### 3. 注意点

- `Effects`只在客户端运行, 不会在服务器端渲染的时候被调用
- 不能在`useInsertionEffect`中更改state
- 不能在`useInsertionEffect`中使用`ref`, 因为DOM还没被更新, 引入



## 十. useSyncExternalStore

### 1. 定义

- `const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`: 让你能够订阅(subscribe)到外部的仓库(store)
  - `subscribe` function: 订阅这个仓库, 并返回一个取消订阅的函数
  - `getSnapshot` function: 读取一个关于这个仓库的快照(snapshot)

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

**使用场景:**需要使用在React之外的数据

- 第三方的状态管理库中的数据state
- 浏览器APIs暴露的可变数据和订阅它改变的事件

- **在你想要将React与非React整合在一起的时候, 十分有用**

### 2. 作用

```jsx
export function useOnlineStatus() {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine, // How to get the value on the client
    () => true // How to get the value on the server
  );
}

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

