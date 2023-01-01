[toc]

# Redux Hook

- 可以避免使用繁琐的`connect`建立store和组件之间的映射关系这一步骤, 直接在组件内定义拿到想要的对象
- 但是还需要`react-redux`这个第三方库, 将react和redux联系起来

```jsx
import { useSelector, useDispatch, shallowEqual } from "react-redux"
```

## 一. useSelector

### 1. 简单使用

- 相当于`mapStateToProps`, 直接在组件内建立store的映射关系

```jsx
const App = memo(() => {
    const { count } = useSelector((state) => ({
            count: state.counter.count
        })
    )
})
```

### 2. 性能优化

- `useSelector`监听了整个state, 当state发生改变时, 即使变换的不是组件的依赖数据, 组件也会发生渲染
- 需要从`react-redux`中导入`shallowEqual`函数, 并将其传入到`useSelector`的第二个参数

```jsx
const { count } = useSelector(() => {}, shallowEqual)
```





## 二. useDispatch

- 直接拿取到`dispatch`, 在组件内直接利用dispatch派发action, 从而避免了封装函数这一步骤

```jsx
import { addNumberAction } from "./reducers"
const App = memo(() => {
   const dispatch = useDispatch()
   
   function addNumber(num) {
       dispatch(addNumberAction(num))
   }
})
```

