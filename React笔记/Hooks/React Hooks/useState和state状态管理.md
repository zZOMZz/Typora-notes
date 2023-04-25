[toc]

# useState和state状态管理

## 1. 基本介绍

```jsx
import { useState } from "react"
// 在触发setCounter函数时, 该函数式组件会重新渲染
function App(props) {
    const [counter,setCounter] = useState(0)
    
    return (
    	<div>{counter}</div>
        <button onClick={e => {
                // state只会在下次渲染时,才会改变, 这里可以理解为通知react改变, 因此这里实际还是
                +100, 而不是+300
                setCounter(counter + 100)
                setCounter(counter + 100)
                setCounter(counter + 100)
            }} >+100</button>
        <button onClick={e => {
                // 传入一个处理函数, 告诉react用state做某事, 而不是直接给state传入一个新的值, 					这样就是+300
                // React会把n=> n+100加入到更新队列中, 在函数调用结束后再更新state
                // 因为React是按照队列依次调用函数, 不同于上面的直接给state做快照, 因此这里可以				实现+300
                setCounter(n => n + 100)
                setCounter(n => n + 100)
                setCounter(n => n + 100)
            }} >+100</button>
    )
}
```

- useState Hook:
  - 会返回一个数组, 第一个参数为设置的**值**, 第二个参数为修改当前设置值的**函数**
  - **在首次使用Hook的时候会创建一个state, React会保存这个state, 在下次调用的时候, useState会返回这个state**
  - 如果不是使用`setState`函数改变`state`, 那么state也会发生改变, 但是不会触发渲染, **但是这个变化是不持久的**, 一旦发生变化, React会清空这些变化, 只按照**快照**行事
- 一个小坑:
  - **`usestate(initialState)`中的initialState这个初始值只有在==第一次渲染==时才有效, 之后的渲染中拿到的是之前的state, initialState不再有意义**



## 2. 使用useState hook的场景


- Hook必须使用在函数顶部, 不能在循环,条件判断或者子函数中调用
- 普通的Javascript函数内部不能使用Hook    
- 只有函数式组件内, 或者自定义Hook(以`use`开头的函数)内才能使用Hook
  - **会通过`eslint-plugin-react-hook`这个包检查常见的错误**



## 3. 内部原理

- 在 React 内部，为每个组件保存了一个数组，其中每一项都是一个 state 对。它维护当前 state 对的索引值，在渲染之前将其设置为 “0”。每次调用 useState 时，React 都会为你提供一个 state 对并增加索引值。你可以在文章 [React Hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)中阅读有关此机制的更多信息



## 4. 更新对象(object)

- 不建议直接修改object, 而是重新创造一个新的对象, 或者复制一个对象, 然后将这个新的值赋予state



- 为了避免多层"嵌套"的对象, 为了更新底层的一处变化需要将整个对象都用解构(...)语法来创建一个新的对象, 为了避免这种麻烦, 可以使用`immer`这个库
  - 其原理是先利用Proxy记录下对原对象的所有改变, 然后再创造一个新的对象
- React中使用`use-immer`

```jsx
import { useImmer } from 'use-immer';

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }
```





## 5. 快照和批处理

### 1. state快照特性

- **一个 state 变量的值永远不会在一次渲染的内部发生变化, 即使其事件处理函数的代码是异步的**, 相关的state值, 会在函数被调用时就被"**快照**"所确定下来了, 在这次函数调用期间不会发生变化, 只会随着渲染完成而改变
  - 在一次渲染中, 即一次函数调用中, state是**固定的**, 是不会改变的(**被set函数改变**)



```jsx
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

#### 1.1 number, string的快照

- 如果赋给`useState`的是一个普通的数字或者字符串时, 即使ref在一次渲染中改变了值(不使用`set`函数), 但由于**快照**的存在, 它在下一次渲染中还是会将一切变化归0, 按照快照的值来渲染



#### 1.2 object的快照

- **但是对象是不同的**, 因为这里是`{ current: initialValue }`这个对象, 因为我们只改变了其中`current`的值, 没有改变这个对象, 因此从对象的角度来看, 对这个对象的**快照**并没有改变, 改变前后都是同一个对象, **因此对`current`的改变会记录到下一次渲染中**



### 2.  批处理

- 所有的`set`操作都会等待函数执行完成后才会一起执行, 就像在餐厅点餐时不会点一道餐,服务员就往后厨跑一次(甚至是不同组件中的set操作), 这样可以避免触发多次的重新渲染, 将渲染尽量集中到一次中



## 6. 不变性(immutable)

- **不可变指的是不可直接改变state**

- 在JavaScript中数字, 字符串等本身就是不可变的, 因此使用`setState`修改时不需要注意, **但是**对象和数组在修改时, 不能直接在原对象或原数组中修改, 因为**在一次渲染中state是固定不变的**, 使用如`position.x = 22`这种修改的方式是无法使React重渲染的, 因此需要重新定义一个数组或者对象, 然后利用`setState`将这个新的对象或数组赋给state
  - 因此涉及直接改变原数组或者对象的方法都禁止使用, 如果想要使用可以使用`immer`这个库
- 如果设计多层嵌套的对象或者数组. 可以使用`immer`这个库, 能更加方便的构造全新的对象或者数组
  - 使用`useImmer`这个Hook

- **直接修改可能造成的影响:** 因为像如`...`这种我们平时常用的拷贝都是浅拷贝, 一个state可能在多处使用了, 如果在一个地方**直接修改**了其中的一个嵌套对象, 由于是浅拷贝, 所以所有用到这个state的地方都会发生改变, 从而造成意想不到的BUG



## 7. state保留

- 在DOM树中相同的组件的state会被保留下来, 如果组件的在DOM树中消失了, 即使马上恢复, state也不会保留
- 因此即使是两个分别渲染的组件(用if...else..进行了条件渲染), 只要这两个组件被渲染在了相同的地方(**在DOM树中的位置没有改变**), React是不会管你是否进行了条件判断, 这两个分别定义的组件会继承同一个state, 因此即使变换逻辑, 分别显示了两个组件, 其state状态是保持一致的
  - 如果给两个组件赋予**不同的key**那么state也不会继承
- **当你在相同位置(如, div元素的第一个子元素)渲染了不同的内容, state就会被重置**



- 所以当你在函数式组件中定义另外一个函数式组件并应用它时, 由于函数每次在重新渲染时都会重新生成, 所以导致在相同位置渲染的每次都是不同的组件, 所以state都会被重置, 无法保存下来



## 8. 利用reducer管理state

- 当随着代码逻辑的越来越复杂, 对某个state的处理可能会越来越多, 越来越复杂, 为了降低复杂度, 并将所有的逻辑存放在一个易于理解的地方, 可以将这些**状态逻辑移到组件之外的一个称为reducer的函数中**



### 1. 设计思想

reducer实际上是以数组上的`reducer()`方法命名的

```js
const arr = [1,2,3,4]
const sum = arr.reduce((result,number) => result + number, 0)  // 10


// reducer函数等效为
function tasksReducer(state,action){}
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



### 2. 三个步骤将`useState`迁移到`useReducer`

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



### 3. useState和useReducer的对比

- **代码体积：** 通常，在使用 `useState` 时，一开始只需要编写少量代码。而 `useReducer` 必须提前编写 reducer 函数和需要调度的 actions。但是，当多个事件处理程序以相似的方式修改 state 时，`useReducer` 可以减少代码量。
- **可读性：** 当状态更新逻辑足够简单时，`useState` 的可读性还行。但是，一旦逻辑变得复杂起来，它们会使组件变得臃肿且难以阅读。在这种情况下，`useReducer` 允许你将状态更新逻辑与事件处理程序分离开来。
- **可调试性：** 当使用 `useState` 出现问题时, 你很难发现具体原因以及为什么。 而使用 `useReducer` 时， 你可以在 reducer 函数中通过打印日志的方式来观察每个状态的更新，以及为什么要更新（来自哪个 `action`）。 如果所有 `action` 都没问题，你就知道问题出在了 reducer 本身的逻辑中。 然而，与使用 `useState` 相比，你必须单步执行更多的代码。
- **可测试性：** reducer 是一个不依赖于组件的纯函数。这就意味着你可以单独对它进行测试。一般来说，我们最好是在真实环境中测试组件，但对于复杂的状态更新逻辑，针对特定的初始状态和 `action`，断言 reducer 返回的特定状态会很有帮助。
- **个人偏好：** 并不是所有人都喜欢用 reducer，没关系，这是个人偏好问题。你可以随时在 `useState` 和 `useReducer` 之间切换，它们能做的事情是一样的！

- 如果在组件状态变化经常出错, 或者想给组件状态变化的时候添加更多的逻辑代码, 推荐使用reducer



### 4. 使用Immer简化useReducer

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

