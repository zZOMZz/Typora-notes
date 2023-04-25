[toc]

# useMemo和useCallback

## 一. useMemo(优化)

### 1. 作用

- `useMemo(callback, dependencies)`
  - 拿到一个返回值(**callback函数的返回值**), 避免在依赖(`props`或者`state`)不变的情况下进行重复计算(有些计算比较消耗cpu资源)


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



## 二. useCallback(优化)

### 1. 作用

- `useCallback(fn, dependencies)`主要目的: **性能优化**
  - 它会返回一个函数的memoized(记忆的)值
  - 在依赖不变的情况下, 通过比较前后两次`dependencies`的值, 决定返回前后定义的两个函数中的哪一个
    - 由于**还是会定义一次函数**, 因此在本身组件内调用, 优化程度较小; 主要是**对传入子函数组件(memo)或继承`PureComponent`的类组件进行优化**
- **当传入组件的props(对象,object和fucntion)发生改变时,组件会重新渲染**, 如果是传入相同的数字或字符串,则不会重新渲染(需要使用`memo`或者`PureComponent`)



**使用场景:**

- 包裹那些需要传入子组件的函数
- 包裹那些需要作为如`useEffect`的dependencies的function
- 包裹那些需要在自定义组件内返回(暴露)出去的函数
- 包裹那些要作为`Context.Provider`的value的函数或者对象

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



### 2. 与useMemo的联系

- **useMemo和useCallback的区别**
  - useMemo主要负责记住(memorize)函数的计算结果
  - useCallback主要负责记住函数

- 因此`useCallback`可以看作由`useMemo`实现的Hook

```jsx
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```



### 3. change state

- - 如果有依赖但是传入空数组`[]`时, 会造成**闭包陷阱**, 即数据只会变化一次
    - 因为useCallback没有发现依赖变化, 因此返回的永远是之前的函数, 而之前的函数定义时闭包的是之前的变量, 即使后面发生了改变, 也是在之后的渲染中, 发生改变的数据也是在之后的函数中

```jsx
// 1. 需要正确写明dependencies
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]);
    
// 2.可以改变set函数的写法, 这样即使不写dependencies也不会陷入闭包陷阱
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ No need for the todos dependency
  // ...
```



