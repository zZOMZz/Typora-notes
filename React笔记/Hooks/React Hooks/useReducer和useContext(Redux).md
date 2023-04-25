[toc]

# reducer和context

## 一. 利用reducer管理state

- 当随着代码逻辑的越来越复杂, 对某个state的处理可能会越来越多, 越来越复杂, 为了降低复杂度, 并将所有的逻辑存放在一个易于理解的地方, 可以将这些**状态逻辑移到组件之外的一个称为reducer的函数中**



### 1. 介绍`useReducer`

- `const [state, dispatch] = useReducer(reducer,initialState, ?init)`
  - `dispatch(action)`: 更新state, 逻辑在`reducer`函数中定义



- `?init`的作用(性能提升)
  - 当你通过一个复杂函数生成初始值得时候, 即使它只在第一次渲染时起作用, 但后续的渲染, 这个函数都会被调用, 浪费性能, 因此可以通过添加第三个参数的方式避免

```jsx
// 尽管React会记住state值, 并在后续的渲染中忽略掉初始化的值, 但这个函数还是会在每次渲染中被调用
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
    
// 解决办法, 这样createInitialState只会在初次渲染中被调用
function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```



### 2. 注意点:

- **dispatch后, state会在下次渲染中改变, 在本次的渲染调用中不会发生改变, 和`useState`相同, 具有"快照"性质**
- 如果前后更改的`state`相同, 则由于内部优化, 不会触发重新更新渲染, 但函数组件可能还会在放弃前被调用

### 3. 设计思想

reducer实际上是以数组上的`reducer()`方法命名的

```js
const arr = [1,2,3,4]
const sum = arr.reduce((result,number) => result + number, 0)  // 10


// reducer函数等效为
function tasksReducer(state,action){
    switch() {}
}
let initialState = [];
let actions = [
  {type: 'added', id: 1, text: '参观卡夫卡博物馆'},
  {type: 'added', id: 2, text: '看木偶戏'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: '打卡列侬墙'},
];
let finalState = actions.reduce(tasksReducer, initialState);
```

你传递给`reduce`的函数被称为`reducer`, 可以将`reducer`接收的参数`(state,action)`也看为`目前的结果和当前的值`, 然后返回`下一个结果(状态)`



### 4. 三个步骤将`useState`迁移到`useReducer`

1. 将设置状态的逻辑修改成dispatch的一个action

```jsx
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

// 转换为
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}
// dispatch的对象称为action
```



2. 编写一个reducer函数, 该函数返回的是更新后的state

```jsx
// state是当前的状态, action是被dispatch派发的action
function yourReducer(state, action) {
  // 给 React 返回更新后的状态
}

// 也可以使用switch语句, tasks是这里之前的state
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
}
```



3. 在组件中使用reducer

```jsx
import useReducer from 'react'

function test() {
    // tasksReducer是之前定义的reducer函数, initialTasks是初始化值
    // 返回一个有状态的值, 和设置这个状态的函数
    const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
    
    // 定义一个触发函数
    function handleAddTask(text) {
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        });
      }
}
```



- **这三步完成后, useReducer就可以替代useState, 接着可以不断的补充action, 和reducer函数面对不同type的action时的处理手段, 这样就能同时管理不止一种的state**



### 5. useState和useReducer的对比

- **代码体积：** 通常，在使用 `useState` 时，一开始只需要编写少量代码。而 `useReducer` 必须提前编写 reducer 函数和需要调度的 actions。但是，当多个事件处理程序以相似的方式修改 state 时，`useReducer` 可以减少代码量。
- **可读性：** 当状态更新逻辑足够简单时，`useState` 的可读性还行。但是，一旦逻辑变得复杂起来，它们会使组件变得臃肿且难以阅读。在这种情况下，`useReducer` 允许你将状态更新逻辑与事件处理程序分离开来。
- **可调试性：** 当使用 `useState` 出现问题时, 你很难发现具体原因以及为什么。 而使用 `useReducer` 时， 你可以在 reducer 函数中通过打印日志的方式来观察每个状态的更新，以及为什么要更新（来自哪个 `action`）。 如果所有 `action` 都没问题，你就知道问题出在了 reducer 本身的逻辑中。 然而，与使用 `useState` 相比，你必须单步执行更多的代码。
- **可测试性：** reducer 是一个不依赖于组件的纯函数。这就意味着你可以单独对它进行测试。一般来说，我们最好是在真实环境中测试组件，但对于复杂的状态更新逻辑，针对特定的初始状态和 `action`，断言 reducer 返回的特定状态会很有帮助。
- **个人偏好：** 并不是所有人都喜欢用 reducer，没关系，这是个人偏好问题。你可以随时在 `useState` 和 `useReducer` 之间切换，它们能做的事情是一样的！

- 如果在组件状态变化经常出错, 或者想给组件状态变化的时候添加更多的逻辑代码, 推荐使用reducer



### 6. 使用Immer简化useReducer

- `reducers`必须是**纯粹的**, `reducers`是在渲染时运行的(actions会排队直到下一次渲染), 因此当输入相同时, 输出也是相同的, 不应该包含异步请求, 定时器或者任何副作用(对组件外部产生影响), **同时应该以不可变值得方式去更新对象和数组**
- 可以使用`immer`这个库来简化`redcuer`, 在这里, `useImmerReducer`可以让我们通过`push`或`arr[i] = 2`来修改state

```jsx
const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}

```



## 二. Context的使用(全局变量)

### 1. 创造一个context

- 无论函数式组件还是类组件, **创建一个context的方法都是相同的**

```jsx
// 父组件定义
const ThemeContext = React.createContext("默认值")
// 只有当组件所处的树中没有匹配到 Provider 时，其 defaultValue 参数才会生效

```

```jsx
// 类组件, 函数组件也类似 
render(){
    return (
        // 2.需要将能使用这个context的子组件包裹起来
        <ThemeContext.Provider value={ color: "red" } >
            <App />
        </ThemeContext.Provider>    		
    )
}
```

- 如果父组件中传入的value改变了, 那么所有用到这个`context value`的子组件(不是所有)都会`re-render`**重新渲染**,  因此应该避免将没经过处理(`useCallback`和`useMemo`)的的函数或者对象当做`value`



### 2. 类组件使用

- Context提供了一种在组件之间共享此类值的方式, 而不必显式的通过组件树的逐层传递props
- Context设计的目的是为了共享那些对于一个组件树而言是**"全局"**的数据

```jsx
// 子组件拿取
static contextType = ThemeContext  // 静态方法, 也可以这样指定context
render() {
    // 4.通过this.context获取
    console.log(this.context)
}

// 3.需要确定要用那个context
App.contextType = ThemeContext
```



### 3. 函数式组件中的使用

#### 1. 利用Consumer

```jsx
// 利用Consumer
import ThemeContext from ''  // 拿到context

// 子组件返回
return (
	<ThemeContext.Consumer>
    	{	
            // 拿到父组件定义的context中的value
            value => {
                return <h2>{ value.color }</h2>
            }
        }
    </ThemeContext.Consumer>

)
```

- 但需要传递一个对象或者函数时, 为了避免Provider组件在重新render渲染时, 重新传递一个新的值(**对象或者数组或函数**), 从而导致子组件进行不必要的更新, **可以将要传递的值放入到组件的`state`中**

#### 2. 利用Hook

1. 利用`createContext`函数创建context

```jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

2. 为父组件提供context, 利用`Context.Provider`

```jsx
import { LevelContext } from './LevelContext.js';

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

3. 子组件拿到context, 并使用context

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

function text() {
    // 利用useContext Hook 拿到定义的LevelContext中的value
    const level = useContext(LevelContext)
}
```

- **context可以多层嵌套, useContext会拿到距离该子组件最近的context(相同名称的)**
- context的常用用法可以用来直接切换主题(`light or dark`)



## 三. 结合reducer和context

- 当要在多个组件中使用同一个state, 并且操作相对繁多, 或者想要在改变状态时进行别的操作, 可以选择结合使用`reducer`和`context`

```jsx
// 将useReducer生成的state和dispatch函数传递给子组件, 使其可以读取到state状态, 并拥有修改权
import { useReducer } from 'react'
import { createContext } from 'react';

// 创建context
export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

function App() {
    const [tasks,dipatch] = useReducer(reducerFunc, initialState)
    
    return (
    	<TaskContext.Provider value={tasks}>
        	<TasksDispatchContext.Provider value={dispatch}>
                
            </TasksDispatchContext.Provider>
        </TaskContext.Provider>
    )
}
```

```jsx
// 也可以将context和reducer的逻辑都抽象到一个组件中
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// 需要使用时, 只需要
return (
	<TasksProvider>
    	{ 包裹其他的组件 }
    </TasksProvider>
)
```

```jsx
// 也可以对useContext进行一层打包, 方便添加一些额外的逻辑
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```
