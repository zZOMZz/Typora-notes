[toc]

# React学习中待解决的问题

## 一. Redux中的异步请求位置

疑问: 在不做中间件增强的情况下, 为什么不能直接在action生成函数中直接async await来发出异步请求, 非得使用`react-thunk`库对dispatch进行增强, 使其可以直接返回一个函数

- `actionCreators.js`

```jsx
export fetchData(name) {
    
}
```



## 二. Reduxjs/ToolKit

1. 不使用这个工具时

```jsx
function addNumber(num){
    return { type: "Add" , payload: num }
}

dispatch(addNumber(3))
```

- 这里dispatch接收的是一个标准action对象
- 我的理解是它会把这个对象重新传入到reducer(state,action)的action参数中, 重新调用reducer修改state



2. 使用工具时:

在reducerSlice中:

```jsx
import { createSlice } from "@reduxjs/toolkit"

const counterSlice = createSlice({
    // name用于生成type: "counter/addNumber"
    name: "counter",
    initialState: {
        counter: 555
    },
    reducers: {
        // 里面的一个个函数相当于原来一个switch中的case语句
        // 这里面定义的函数会被放到counterSlice.actions里
        addNumber(state,action){
            const payload = action.payload
            
            // 可以直接修改state, 内部会自动创建一个新对象并返回
            state.counter = state.counter + payload
        },
        subNumber(state,action){
            
        }
    }
})
// 导出的函数只能利用dispatch来进行调用
// dispatch(addNumber(4)) 传入的参数会被传入到action: { type: , payload: }的payload中
export const { addNumber, subNumber } = counterSlice.actions

// 只需要将slice片段的reducers模块导出就行
export default counterSlice.reducers
```

它直接定义好了addnumber函数, 并在这里面直接完成了修改

子模块在调用时

```jsx
import { addNumber } from "..."

mapDispatchToProps = (dispatch) => {
    add(num) {
        dispatch(addNumber(num))
    }
}
```

它这里给dispatch传入的不在是一个标准的action对象, 反而传入的函数addNumber会接收到这个标准对象

`{ type: "counter/addNumber", payload: num }`, 并将其传入到addNumber的action中

**疑问:**那这里的dispatch究竟有什么作用, 它调用了什么函数