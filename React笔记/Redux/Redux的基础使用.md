[toc]

# Redux的基础使用

## 一. Javascript纯函数

- 函数式编程中有一个重要的概念: 纯函数, Javascript符合函数式编程, 所以也有纯函数的概念
  - react开发和纯函数息息相关
  - react的组件就被要求像是一个纯函数, redux中也有一个reducer的概念, 也要求是一个纯函数
  - react规定, 所有的组件都得像纯函数一样保护它们得props不被修改



- 有相同的输入值时, 也会产生相同的输出
- 函数的输出和输入值与其他隐藏的信息和状态无关
- 该函数不能有可观察的函数副作用, 如触发事件等, 不能修改外部状态



## 二. Redux的核心理念

### 1. action

- Redux要求我们通过action来更新数据
  - 所有的数据变化必须通过`dispatch`(派发)action 来进行更新
  - action是一个对象, 用来描述更新的type和content



### 2. reducer

- reducer将state和action联系在一起
  - reducer必须是一个纯函数
  - 将传入的state和action结合起来生成一个新的state, 并将新的state返回



### 3. Store

- 定义一个统一的规范来操作这个数据
- **三大原则: 单一数据源, state是只读的, 使用纯函数来执行修改**
- State只能是只读的, 只能通过action来改变他, 不能直接改变它
  - 可以确保所有的修改被集中化处理, 并且按照严格的顺序来执行
  - 确保View或者网络请求都不能直接修改State, 它们也只能通过action来改变state
- 可以创建多个store, 但是最好只用**单一数据源**
- 使用纯函数来执行修改, 如reducer
  - 可以将将reducer拆分成多个小的reducer, 但必须都得是纯函数



### 4. 文件结构

- `store/index.js`: 初始化store
- `store/reducer.js`: 记录reducer函数
- `store/actionCreators.js`: 记录一些不同action的生成函数
- `store/constans.js`: 记录一些修改type值, 和一些常量



## 三. 简单引用

```jsx
// 1.定义store
import { createStore } from "redux"
// 1.1 定义初始值
const initialState = {
    name: "zzt",
    age: "18"
}
// 1.2 定义reducer, reducer返回的值会修改state, 在与store联系后, state在被调用时会被自动传入
function reducer(state = initialState, action) {
    switch(action.type) {
        case "change_name":
            return { ...state, action.name }
        case "change_age":
            return { ...state, action.age }
        default:
            return state
    }    
}
// 1.3 定义store, 将reducer与store联系起来
const store = createStore(reducer)


// 2.修改store

// 获取到state
console.log(store.getState())

// 2.1 定义生成action函数, 并修改state
const changeNameAction = (name) => {
    return { type: "change_name", name }
}
// 2.2 派发action, 这个action会被传入到reducer的第二个参数中, 并执行reducer,生成新的state
store.dispatch(changeNameAction("zzzzz"))


// 3. 订阅store的数据

// 每次state发生改变时, 就会执行这个函数
const unSubscribe = store.subscribe(() => {
    console.log("数据变化了", store.getState())
})

// 取消订阅
unSubscribe()
```

- 调用store的dispatch函数, 会重新调用store定义时传入的reducer函数, 并将`dispatch(action)`中的action传入到`reducer`的第二个参数
- store.subscribe如果在`componentDidmount`中定义了, 那么也需要在`componentWillUnmount`中取消



## 四. 类组件使用Redux

### 1. Provider

**思路:**

- 在`componentDIdMount`组件中, 定义一个`store.subscribe`, 在每次改变store的state时都通过这个订阅来修改类组件的state
- 类组件的一些方法在修改变量时, 不再是直接修改类组件的state, 而是通过`dispatch`来修改store中的state, 然后再触发订阅的`subscribe`回调函数, 修改类组件中的state

- 可以使用高阶组件封装这个过程

```shell
# 将组件与封装的store联系在一起
npm install react-redux
```

```jsx
// 最上层父组件
import { Provider } from "react-redux"
import { store } from "./store"

// 直接给所有的子组件提供store
root.render(
    <React.StrictMode>
		<Provider store={store} >
    		<App />
    	</Provider>
    </React.StrictMode>       
)
```



### 2. connect

- 利用`connect`降低store和和类组件的耦合度

```jsx
// 子组件接收store中的数据
import { connect } from "react-redux"
// 拿取action制造函数
import { addNumberAction, subNumberAction } from "../store/actionCreators"

export class App extends PureComponent {
    render() {
        const { counter } = this.props
    }
}

// 定义connect接收的第一个函数, 主要作用是为了明确这个组件需要使用store中的哪些参数
// 通过返回值接收参数, 当别的父组件用这个子组件时, 会自动地将store中的映射数据加入到props中
const mapStateToProps = (state) => ({
    return {
        counter: state.counter
    }
})
// 这里返回的对象中的内容, 也会被映射到props中
const mapDispatchTOProps = (dispatch) => ({
    return {
        addCounter(num) {
            dispatch(addNumberAction(num))
        },
        subCounter(num) {
            dispatch(subNumberAction(num))
        }
    }
})

// 需要连续调用两次, 第一次确定state和store的映射关系, 返回一个高级组件, 第二次将对应组件传入
// 返回一个封装好的组件
export default connect(mapStateToProps, mapDispatchTOProps)(App)
```



## 五. Redux中进行异步请求

- 对redux中的dispatch接收进行增强(中间件)
  - store.dispatch(object)
  - store.dispatch(function)
- 当dispatch接收到一个函数时, 这个函数会被立即调用执行, 并传入dispatch和getState函数
  - dispatch: 用于再次派发action
  - getState: 可以获取到一些原来的状态

```shell
npm install redux-thunk
```

```jsx
// index.js
import  thunk from "redux-thunk"
import { createStore, applyMiddleware } from "redux"

// 在创建store的时候注册中间件
const store = createStore(reducer, applyMiddleware(thunk))
```

````jsx
// actionCreators.js
// 返回的函数会被自动调用
export const fetchData = () => {
    // 返回一个函数
    return (dispatch, getState) => {
        axios.get("url").then((res) => {
            const data = res.data.data
            
            // 利用接收到的数据改变store中的state
            dispatch(changeData(data))
        })
    }
}
````



## 六. 调试工具使用

[reduxjs/redux-devtools(github.com)](https://github.com/reduxjs/redux-devtools)

- 使用redux-devtools

```jsx
import { compose } from "redux"
// 开启redux-devtools, 生产环境下关闭
const composeEnhancers = window._REDUX_DEVTOOLS_EXTENSION_COMPOSE_ || compose
// 上面的函数会将几个中间件整合起来
const store = createStore(reducer, composeEnhancers(applyMiddleware(thunk)))
```





## 七. 模块整合

```jsx
import { combineReducers } from "redux"

// 将这个整合后的reducer传入createStore
const reducer = combineReducers({
    counter: counterReducer,
    home: homeReducer
})

// 获取conunter的state数据
console.log(store.getState().counter)
```

```jsx
// combineReducers内部原理
// 类似, 源码会有一些优化
function reducer(state = {}, action) {
    return {
        counter: couterReducer(state.counter, action),
        home: homeReducer(state.home, action)
    }
}
```



