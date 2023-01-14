[toc]

# Redux高级

## 一. ReduxToolkit工具包

RTK官方推荐的编写redux逻辑的方法

- 两个包
  - toolkit
  - **react-redux: 提供Provider和connect映射函数, 通过这两个函数在子组件中拿到定义好的store**

```shell
npm install @reduxjs/toolkit react-redux
```



- 核心API
  - **configureStore**: 
    - 包装createStore以提供简化的配置选项和良好的默认值. 
    - 可以自动组合你的slice reducer, 
    - 添加你提供的Redux中间件, redux-thunk(增强dispatch, 开启异步请求), 
    - 并且启用Redux DevTools Extension
  - **createSlice**: 
    - 接受reducer函数的对象, 切片名和初始状态, 并自动生成切片reducer, 并带有相应的actions
  - **createAsyncThunk**: 
    - 接受一个动作类型字符串和一个返回承诺的函数, 并且生成一个pending/ fulfilled/rejected基于该承诺分派动作类型的thunk



## 二. RTK创建Store

### 1. configureStore

- configureStore用于创建store对象, 常见参数:
  - reducer: 将slice代码片段中配置好的reducer传入其中
  - middleware: 可以使用参数, 传入其他中间件
  - devTools: 是否配置devTools工具, 默认为true

```jsx
import { configureStore } from "@reduxjs/toolkit"

const store = configureStore({
    reducer: {
        counter: counterReducer
    }
})
```



### 2. createSlice, createAsyncThunk

[createAsyncThunk | Redux Toolkit (redux-toolkit.js.org)](https://redux-toolkit.js.org/api/createAsyncThunk)

- 用于创建不同模块之间的slice片段
- `createAsyncThunk`接收的第一个参数, 可以用来表示生命周期
- For example, a `type` argument of `'users/requestStatus'` will generate these action types:
  - `pending`: `'users/requestStatus/pending'`
  - `fulfilled`: `'users/requestStatus/fulfilled'`
  - `rejected`: `'users/requestStatus/rejected'`

```jsx
import { createSlice } from "@reduxjs/toolkit"

// 5. 异步处理函数也需要导出
// 子模块也是通过dispatch(fetchHomeData())来调用异步请求
export const fetchHomeData = createAsyncThunk("counter/fetchHomeData", 
	// 在子模块dispatch这个异步函数时, 第一个参数为传入的参数, 第二个参数为默认传入的store
	async (_, { dispatch }) => {
    	const res = await axios.get("url")
    	
        // 也可以通过接收到dispatch来修改state, 从而避免进行状态监听
        dispatch(addNumber(res.data.counter))
        
        
        // 需要状态监听则需要返回参数
    	return res.data
})

const counterSlice = createSlice({
    // name用于生成type: "counter/addNumber"
    name: "counter",
    initialState: {
        counter: 555
    },
    // 1.同步修改的函数, 里面定义的函数会被自动加到action中
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
    },
    // 2.异步状态处理, 用于处理异步的函数
    // 处理异步请求的三个不同的状态: reject, fulfilled, pending
    extraReducers: {
        [fetchHomeData.fulfilled](state,action) {
            state.banners = action.payload.data.banners
        }
    }
})
// 3. 需要将别的组件要使用的action函数导出
// 导出的函数只能利用dispatch来进行调用
// dispatch(addNumber(4))传入的参数4会被传入到
// action: { type:"counter/addNumber" , payload: 4 }中
export const { addNumber, subNumber } = counterSlice.actions

// 4.只需要将slice片段的reducers模块导出就行, 用于store定义
export default counterSlice.reducer
```

- 在slice片段外, 通过`createAsyncThunk`定义好被模块dispatch的函数, 在slice片段内, 定义好async处理函数面对不同状态的处理, 改变state, 不能在外部直接修改内部slice的数据

- 也可以直接在async函数内通过dispatch直接修改state, 避免进行状态监听



## 三. Redux中的数据不变性

使用了`immerjs`第三方库, 早期使用`immutable`这个库



新的算法: Persistent Data Structure(持久化数据结构或一致性数据结构)

- 用一种数据结构来保持数据
- 当数据被修改时, 会返回一个对象, 但是新的对象会尽可能的利用之前的数据结构而不会对内存造成浪费



## 四. Redux手写connect函数

```jsx
// 导入定义好的store
<StoreContext.Provider value={store}>
</StoreContext.Provider>
// 也可以通过context全局注入一个store
import store from "..."

export function connect(mapStateToProps,mapDispatchToProps) {
    return function(OldComponent) {
        class NewComponent extends PureComponent {
            constructor(props,context) {
                super(props)
                
                this.state = mapStateToProps(store.getState)
                this.dispatch = mapDispatchToProps(store.dispatch)
            }
            
            // 监听store的变化
            componentDidMount() {
                this.unSubscribe = store.subscribe(() => {
                    this.setState(mapStateToProps(store.getState()))
                })
            }
            
            render() {
                return (
                 <OldComponent{...this.props}{...this.state,...this.dispatch} />
                )
            }
        }
        // 确定context
        NewComponent.contextType = StoreContext
        // 这样组件内可以通过this.context获取到store
        // 或者使用constructor接收到的第二个参数context来完成
        return NewComponent
    }
}
```

