[toc]

# React 高级

## 一. setState

```jsx
// 1. setState是异步的
// 2. setState不仅能接手一个对象, 也能接手一个回调函数
// 3. state是改变之前的数据
this.setState((state, props) => {
    // 可以在这个回调函数内进行一些逻辑操作
    
    return {
        message: "改变"
    }
})

// 4. 在改变数据完成后也可以进行操作
this.setState({ message: "zz"}, () => {
    console.log("完成了")
} )
```

- setState设计为异步, 可以显著的提升性能
  - 如果每次调用setState都进行一次更新, 那么render函数会被频繁调用, 界面重新渲染, 造成效率低下
  - 最好的方法是获取到多个更新后, 将多次更新放入到队列里, 获取时进行批量处理, 这样render函数只用执行一次, 避免大量回流和重绘
- 如果同步更新了state, 但是还没有执行render函数, 那么state和props不能保持同步
  - state和props不保持一致性, 同步改变了state但是不能马上执行render函数, 会在开发中产生很多问题
  - 设计成异步, 可以使state一改变就能马上执行render函数



```jsx
import { flushSync } from "react-dom"

// 将setState改为同步函数
flushSync(() => {
    this.setState({ message: "你好" })
})
console.log(this.message)  // "你好"
```



## 二. React更新流程

==待学习diff算法==

React更新算法的复杂度为$$O(n)$$, 并不是所有节点全部比较

- 同层节点之间相互比较, 不会跨层比较
- 不同类型的节点, 会产生不同的树结构
- 开发中, 可以通过key来指定哪些节点在不同的渲染下保持稳定,在虚拟DOM发生改变时, 尽量复用key相同的节点(位置发生改变..), 减少开销
  - (以ul为例)在最后位置插入元素, 有无key影响不大
  - 在最前面插入, 在无key的情况下, 所有的元素都要更改
  - key需要是唯一的



## 三. render函数的优化

- 在父组件重新调用render函数重新渲染时, 子组件即使未发生改变也会被迫重新渲染
- 即使state的前后状态相同, 但因为调用了setState, 还是会进行重新render



解决办法:

- 使用`shouldComponentUpdate(nextProps, newSate)`生命周期钩子(SCU)
  - nextProps: 新的属性, 子组件可用
  - newState: 新的state参数, 可以用来比较



### 1. 类组件: 使用PureComponent纯组件

- 会自动为我们进行render函数优化, 会自动进行判断



### 2. 函数组件: 

- memo包裹函数, memo会帮助我们判断

```jsx
const Profile = memo(function(props){
    return <h2>Profile: {props.message}</h2>
})
```





## 四.  ref获取DOM

在React中, 不推荐使用原生JS获取DOM元素, 因此需要使用`ref`来获取

- 父组件通过ref拿到子组件的实例时, 可以直接调用子组件的方法

1. `this.refs`获取(不推荐)

```jsx
render() {
    return (
    	<h2 ref="zzt">ZZT</h2>
    )
}

getDOM(){
    console.log(this.refs.zzt)  // 不推荐
}
```



2. `createRef`, 先创建再绑定

```jsx
import { createRRef } from "react"

constructor(){
    this.titleRef = createRef()
}

render() {
    return (
    	<h2 ref={this.titleRef}></h2>
    )
}

// 获取
getDOM() {
    console.log(this.titleRef.current)
}
```



3. 传入函数

```jsx
constructor() {
    this.titleEl = null
}

render() {
    return (
        // 传入的回调函数会自动获取到EL, 将其赋值给其他变量, 可供其他函数使用
    	<h2 ref={(el) => { this.titleEl = el }}></h2>
    )
}

getDOM() {
    console.log(this.titleEl)
}
```



- 上述的方法适用于**类组件**, 因为只有类组件有实例, 函数式组件没有实例

函数组件:

```jsx
improt { forwardRef } from "react"

// 传给函数式组件的ref会被传入到函数的接受变量ref中
const HelloWorld = forwardRef(function(props,ref){
    return(
    	<h2 ref={ref}>HHH</h2>
    )
})
```



## 五. 受控组件(双向绑定)和非受控组件

### 1. 受控组件

- 表单元素如`input`, `textArea`, `select`一旦绑定一个`value`属性, 就会变为受控组件, 无法自由输入输出
- 这种受控组件必须由`setState`改变状态, 否则无法改变

```jsx
<input type="text" value={username}/>  // 未绑定监听事件, 同时也无法输入, 绑定onChange事件
// 需要在onChange事件处理函数中改变username
<input type="text" value={username} onChange={(e) => this.changeInput(e)} />

changeInput(e) {
    this.setState({ username: e.target.value })
}
```



- checkbox多选

```jsx
constructor(){
    this.state = {
        hobbies : [
        { name:"", value:"", isCheck: true }
    	]
    }
}

// 在渲染时, 只需要通过isCheck渲染状态, 在监听函数中改变isCheck就行
```



### 2. 非受控组件

一般是直接操纵DOM, 不通过React来管理数据

更推荐使用受控组件

```jsx
<input type="text" defaultValue={intro} ref={this.introRef} />

conponentDidMount() {
    this.introRef.addEventListener(() => {})
    
    console.log(this.introRef.value)
}
```





## 六. 高阶组件

**高阶组件本身是一个函数, 接受一个组件作为参数, 返回值为一个新的组件**

- 可以对传入的组件进行一层封装
- 进行数据劫持设置封装



- `this.forceUpdate()`: 强制执行render函数





## 七. Portals

将特定的元素挂载到特定的地方

类似 vue的`Teleport`: `<Teleport>` 是一个内置组件，它可以将一个组件内部的一部分模板“传送”到该组件的 DOM 结构外层的位置去。

```jsx
import { createPortal } from "react-dom"

// createPortal([需要挂载的Element元素],[想要挂载到的地方(进入标签里)]): 返回一个ReactElement

class .. {
    
    render(){
        return (
        	<h1>HHHHH</h1>
            {
            	createPortal(<h2>YYYY</h2>, document.querySelector("#zzt"))
            }
        )
    }
}
```



## 八. Fragment

与vue的template相似, 会把其中的元素包裹起来, 但是在最终**无须向DOM添加额外节点**, 也不会被渲染出来

- 语法糖`<>  </> `省略其中的文字, 需要绑定属性, 如key的情况下, 不能使用这种语法糖



## 九. StriceMode 

- StriceMode 是一个用来突出显示应用程序中潜在问题的工具
  - 它与Fragment一样, 不会渲染任何可见的UI
  - 它为后代元素触发额外的检查和警告
  - 严格模式检查仅在开发模式下运行, 它们不会影响生成和构建
- 可以为应用程序的任何部分启用严格模式
  - 不会对Header和Footer组件运行严格模式检查
  - 但是,ComponentOne和ComponentTwo 以及它们的所有后代元素都将进行检查

```jsx
<StriceMode>
// 给其中元素的所有子元素和后代开启严格模式
</StriceMode>
```

- 识别一些不安全的生命周期
- 过时的ref API 如`this.refs`
- 检查的副作用:
  - 这个组件的constructor会被调用两次, 这是在严格模式下的故意的操作, 用来检测是否有一些逻辑代码被重复调用
  - 在生成环境下不会被调用两次
- 使用一些过期的API
- 检测过时的context的API
- **严格模式会强制把生命周期如`useEffect`执行两次, 检查副作用**