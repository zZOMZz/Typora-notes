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
  - 依赖DOM的操作可以在这里进行
  - 发送网络请求
  - 可以在此处添加一些订阅
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