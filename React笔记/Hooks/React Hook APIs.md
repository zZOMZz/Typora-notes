[toc]

# React Hook APIs

- useState和useEffect这两个主要的API在另外的一个文件中

## 一. useContext

之前的开发中(比较繁琐):

- 类组件可以通过[类名].contextType = MyContext的方式, 在类中获取context
- 在函数式组件中或者多个Context中通过MyContext.Consumer的方式给函数传入value来共享context
- 多个context一起使用时会比较繁琐



```jsx
import { memo, useContext } from "react"
import { Usercontext } from "./context"

const App = memo(() => {
    // 通过useContext可以直接拿取到context中的value, 从而避免了多级嵌套
    const user = useContext(UserContext)
    
})
```

- 注意: 当组件最上层包裹的<MyContext.Provider  value={..} >更新时, 该Hook会利用最新的数据进行重新渲染





## 二. useReducer

- useReducer只是useState的一种替代方案
  - 在一些场景下, 如果state的处理逻辑比较复杂, 我们可以通过useReducer对其进行拆分
  - 或者这次修改的state需要依赖之前的state时, 可以使用

```jsx
const reducer = (state,action) => {
    switch(action.type) {
        case "addNumber":
            return { ...state, counter: state.counter + 1 }
        case "subNumber":
            return { ...state, counter: state.counter - 1 }
    }
}

const App = mome(() => {
    const [state,dispatch] = useReducer(reducer, { counter: 0 })
    
    return (
    	<h2>{state.counter}</h2>
        <button onClick={e => dispatch({ type: "addNumber" })} ></button>
    )
})
```



## 三. useCallback(优化)

- useCallback目的是为了进行多次定义的性能优化
  - 它会返回一个函数的memoized(记忆的)值
  - 在依赖不变的情况下, 多次(相同)定义的时候返回的值是相同的, **但是因为还是会定义一次函数, 因此在本身组件内调用, 优化程度较小; 主要是对传入子组件的函数进行优化, 前后定义的两个函数即使完全相同,但也是不同的对象, 因此会触发子组件重新渲染**
  - 如果有依赖但是传入空数组`[]`时, 会造成闭包陷阱, 即数据只会变化一次
    - 因为useCallback没有发现依赖变化, 因此返回的永远是之前的函数, 而之前的函数定义时闭包的是之前的变量, 即使后面发生了改变, 也是在之后的渲染中, 发生改变的数据也是在之后的函数中

- `useCallback(fn, [])`
- **对传入被memo或pureComponent进行引用的子组件的函数进行优化**
- **当传入组件的props(对象,object和fucntion)发生改变时,组件会重新渲染**, 如果是传入相同的数字或字符串,则不会重新渲染

作用

```jsx
function zzCOM(props) {
    
}

const fff = memo(() => {
	const [message,setMessage] = useState()
    const [counter,setCounter] = useState()
    
    const increment = useCallback(() => {
        
    },[counter])
    
    return (
    	<zzCOM increment={increment} />
    )
})
// 当传入组件的props发生改变时,组件会重新渲染
// 作用就是在修改state时, 不重新渲染zzCOM
```



## 四. useRef

1. 通过useRef来定义一个值, 再将这个值赋值给element元素的ref属性, 从而绑定DOM, 根据这个值拿到DOM上的属性

```jsx
const titleRef = useRef()
(<h2 ref={titleRef} ></h2>)
```



2. useRef 在组件多次渲染时, 返回的是同一个值, 改变的是current值, **因此可以保存一个数据, 这个ref对象在整个生命周期中都可以保持不变**

```jsx
const countRef = useRef()
countRef.current = count
const increment = useCallback(() => {
    setCount(countRef.current + 1)
}, [])
```



3. 因此可以通过useRef, 和传入正确的依赖来**避免陷入闭包陷阱**(页面重新渲染但使用的还是旧数据), 和避免多次重复渲染

​	

## 五. useMemo(优化)

- 拿到一个返回值, 而不是像`useCallback`一样拿到一个函数, 避免在依赖不变的情况下进行重复计算

```jsx
function calcNum(num) {
    // 累加: 1+2+...+num
}

const App = memo(() => {
    const [count,setCount] = useState()
    const result = useMemo(() => {
        return calcNum(count)
    },[count])
    
    return (
        <h2>{ count }</h2>
    	<h2>{result}</h2>
    )
})
```

- useMemo返回的是一个传入子组件的**对象**的时候能产生优化



## 六. useImperativeHandle

- 子组件对父组件传入的ref进行处理, 权限控制
  - 函数式组件想要接收ref这个props需要通过`forwardRef`来进行增强
    - 第一个参数接收props
    - 第二个参数接收ref

```jsx
// 子组件
import { forwardRef, memo } from "react"
const HelloWorld = memo(forwardRef((props,ref) => {
    const inputRef = useRef()
    useImperativeHandle(ref,() => {
        // 返回的这个对象会被父组件接收
        return {
            focus()  {
                inputRef.current.focus()
                console.log("focus")
            }
        }
    })
    
    return (
    	
    )
    
}))
```



## 七. useLayoutEffect

- useLayoutEffect和useEffect类似
  - useEffect会在渲染的内容更新到DOM后执行, 不会阻塞DOM 的更新
  - useLayoutEffect会在渲染的内容更新到DOM之前执行, 会阻塞DOM的更新
- 避免闪烁现象