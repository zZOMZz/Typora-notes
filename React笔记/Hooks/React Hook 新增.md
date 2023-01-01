[toc]

# React Hook 新增

## 一. useId

- 用于生成横跨服务器端和客户端的唯一ID, 同时避免hydration不匹配的
- useId是用于react的同构应用开发, 前端的SPA(单页面富应用)页面不需要他
- 可以保证应用程序在客户端和服务器运行时生成唯一的ID



## 二. useTransition

- 返回一个状态值表示过渡状态的等待状态, 以及一个启动该过度任务的函数
- 告诉React某部分任务的更新优先级较低, 可以稍后进行更新, 可以在硬件条件差或者网速差时使用, 提高用户交互体验



- pending: boolean 表示startTransition函数执行的状态, 当还未执行时为`true`
- startTransition: 降低代码执行渲染优先级的函数

```jsx
const App = memo(() => {
    const [pending,startTransition] = useTransition()
    
    function valueChangeHandle(event) {
        // 将大量数据的渲染优先级滞后
        startTransition(() => {
            const keyword = event.target.value
            ... // 过滤
            // 大数据渲染
            setShowNames(...)
        })
    }
	return (
		<input type="text" onInput={e => valueChangeHandle(e)}/>
    )            
                       
})
```



## 三. useDeferredValue

- useDeferredValue 接受一个值, 并返回该值的新副本, 该副本将被推迟到更紧急的更新之后



```jsx
const App = memo(() => {
    const [showNames,setNames] = useState()
    // 生成副本
    const deferedShowNames = useDeferredValue(showNames)
    // 之后的渲染遍历都使用上面这个副本, 即可进行延迟
})
```

