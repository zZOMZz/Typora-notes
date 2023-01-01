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



## 二. 简单使用(useState)

```jsx
import { useState } from "react"
// 在触发setCounter函数时, 该函数式组件会重新渲染
function App(props) {
    const [counter,setCounter] = useState(0)
    
    return (
    	<div>{counter}</div>
        <button onClick={e => setCounter(counter + 100)} >+100</button>
    )
}
```

- useState Hook:
  - 会返回一个数组, 第一个参数为设置的**值**, 第二个参数为修改当前设置值的**函数**
  - **在首次使用Hook的时候会创建一个state, React会保存这个state, 在下次调用的时候, useState会返回这个state**
- 一个小坑:
  - **`usestate(initialState)`中的initialState这个初始值只有在==第一次渲染==时才有效, 之后的渲染中拿到的是之前的state, initialState不再有意义**




- Hook必须使用在函数顶部, 不能在循环,条件判断或者子函数中调用
- 普通的Javascript函数内部不能使用Hook    
- 只有函数式组件内, 或者自定义Hook(以`use`开头的函数)内才能使用Hook



## 三. Effect Hook

- Effect Hook可以让你来完成一些类似于class生命周期的功能, 但它比原来的生命周期更加强大
- 类似网络请求, 手动更新DOM, 事件监听, 都是react更新DOM的一些**副作用**

```jsx
const App = memo(() => {
    const [counter,setCounter] = useState()
    
    // 1.任何上述的副作用,都在useEffect Hook中使用, 这个Hook会在渲染完成后调用
    useEffect(() => {
        document.title = counter
        
        // 2.会在组件被重新渲染或者卸载时执行, 清除函数
        return () => {
            // 进行一些取消工作
        }
    })
    // 3. 同一个函数式组件内可以存在多个useEffect Hook
    useEffect(() => {
        
    })
    
    return (
    	<div>{counter}</div>
        <button onClick={e => setCounter(counter+1)} >+1</button>
    )
})
```



- useEffect实际上有两个参数
  - 参数一: 执行的回调函数
  - 参数二: 该useEffect在哪些state发生变化时, 才重新执行
    - 如果传入的是一个空数组, 即不受任何参数的影响, 只会在初始渲染的时候执行