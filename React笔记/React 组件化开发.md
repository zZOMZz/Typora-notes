[toc]

# React 组件化开发

插件Snippets: [vscode-react-javascript-snippets/Snippets.md](https://github.com/ults-io/vscode-react-javascript-snippets/blob/HEAD/docs/Snippets.md)

## 一.  组件基本介绍

- 根据组建的定义方式, 可以分为:
  - 函数组件(Functional Component)
  - 类组件(Class Component)
- 根据组件内部是否有状态需要维护, 分为:
  - 无状态组件(Stateless Component)(一般函数式组件)
  - 有状态组件(Stateful Component)(类组件)
- 根据组件的不同职责, 分为
  - 展示型组件(Presentational Component)
  - 容器型组件(ontainer Component)



- 函数组件, 无状态组件, 展示型组件主要关注UI的展示
- 类组件, 有状态组件, 容器型组件主要关注数据逻辑





## 二. 类组件

- 组件名称必须是大写字符开头
- 类组件必须继承自React.Component
- **类组件必须实现render函数**



- render函数返回类型
  - react元素: 通过jsx编写的代码会被调入React.createElement, 返回一个React元素
  - 组件或者一个fragments
  - 字符串, 数字类型



## 三. 函数式组件

```jsx
function App() {
    // 返回值与类组件中的render函数返回的一样
}
```

- 函数组件(不用Hooks)的缺点
  - 没有生命周期, 会被更新挂载, 但是没有生命周期函数
  - this关键字不能指向组件实例
  - 没有内部状态(state)





## 四. 生命周期

[React lifecycle methods diagram (wojtekmaj.pl)](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

- 装载阶段(Mount), 组件第一次在DOM树中被渲染的过程
- 更新阶段(Update), 组件状态发生变化, 重新更新渲染的过程
- 卸载阶段(Unmount), 组件从DOM树中被移除的过程

React的生命周期主要是针对类组件, 函数组件需要hook来模拟生命周期

React的生命周期函数(告诉我们当前处于哪些阶段, 会对我们组件内部实现回调):

- `componentDidMount`: 组件已经被挂载到DOM上, 就会回调
  - **依赖DOM的操作可以在这里进行**
  - **发送网络请求**
  - 可以在此处添加一些订阅
  - 在render之后执行
- `componentDidUpdate`: 组件已经发生更新时, 就会发生回调
  - 更新后会被立即调用
- `componentWillUnmount`: 组件机键被移除时, 就会发生回调

![Snipaste_2022-12-25_11-06-14](C:\Users\zZOMZz\Desktop\Typora笔记\React笔记\图片\Snipaste_2022-12-25_11-06-14.png)

![Snipaste_2022-12-25_11-05-46](C:\Users\zZOMZz\Desktop\Typora笔记\React笔记\图片\Snipaste_2022-12-25_11-05-46.png)



## 五. 组件间通信

- 父组件通过`[属性] = [值]`的形式来传递给子组件数据

```
<MainBanner banners={abc} />
```

- 子组件通过`props`参数来获取父组件传递过来的数据

```jsx
class MainBanner extends Component {
	constructor(props){
		super(props)	
	}
    
    render(){
        const { banners } = this.props
        return (
        	
        )
    }
}
```

### 1. propTypes 参数

- 如果项目中默认继承了Flow或者TypeScript, 那么可以直接进行类型验证
- 如果没有, 也可以使用prop-types库来进行参数验证

```jsx
// 验证类型
Children.propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number,
    height: PropTypes.number
}
```



### 2. defaultProps 默认值

```jsx
Children.defaultProps = {
    bannners: [],
    title: "Zzz"
}
```



### 3. 在子组件中改变父组件

有时,我们需要在子组件中向父组件传递消息,改变父组件:

- vue中是通过自定义事件来完成的`emit`
- React中是通过`props`传递消息, 父组件通过`props`给子组件一个改变自己的回调函数(需要是箭头函数), 然后在子组件中接收这个函数, 并绑定到特定的地方



### 4. 组件插槽实现

1. 组件的children子元素

```jsx
<NavBar>
	<button>按钮</button>
	<h2>标题</h2>
    <i>斜体文字</i>
</NavBar>
```

- 写进组件NavBar中的元素, 会被放入NavBar实例的`this.props.children`属性里, 形成一个数组, 放入多个时是一个数组, **如果只有一个元素, children就是这个元素**



2. props属性直接传递React元素



### 5.实现作用域插槽:

- 需要依赖子组件中数据进行渲染

- 利用props属性直接传递React元素时, 不传递元素, **而是传递一个接收参数的函数**, 这样不需要再从子组件传出来, 自己调用函数即可
- 依赖函数闭包



## 六. Context的使用(全局变量)

```jsx
// 支持解构
<Home {...info} />
```

- Context提供了一种在组件之间共享此类值的方式, 而不必显式的通过组件树的逐层传递props
- Context设计的目的是为了共享那些对于一个组件树而言是**"全局"**的数据
- 可以用redux取代, **redux中也利用到了Context**

```jsx
// 1.父组件定义
const ThemeContext = React.createContext()

class ... {
    ...
    
    render(){
    	return (
    		// 2.需要将能使用这个context的子组件包裹起来
    		<ThemeContext.Provider value={ color: "red" } >
    			<App />
    		</ThemeContext.Provider>    		
    	)
	}
}
```

```jsx
// 子组件拿取
render() {
    // 4.通过this.context获取
    console.log(this.context)
}

// 3.需要确定要用那个context
App.contextType = ThemeContext
```

```jsx
// 5.函数式子组件的用法
// 利用Consumer

return (
	<ThemeContext.Consumer>
    	{	
            // 将父组件定义的context value传进来
            value => {
                return <h2>{ value.color }</h2>
            }
        }
    </ThemeContext.Consumer>

)
```





## 七. 事件总线

- 利用一些第三方库生成事件总线, 进行触发和监听

```jsx
// 触发
eventBus.emit("bannerPre",...args)

// 监听
componentDidMount(){
    eventBus.on("bannerPre", (...args) => {
        console.log("监听到事件")
    })
}

// 移除监听
componentWillUnmount(){
    event.off("bannerPre", () => {})
}
```

