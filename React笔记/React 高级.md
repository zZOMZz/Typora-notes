[toc]

# React 高级

## 一. setState

- 类组件改变state调用的方法

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

- **setState设计为异步, 可以显著的提升性能**
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

**因为`setState`是异步的, 因此不能直接对state和props形成依赖**

```jsx
// 错误写法
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

**修正**: 可以传递一个接收上一个`state`和`props`为参数的函数

```jsx
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```



## 二. React更新流程

==diff算法==

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

- 会自动为我们进行render函数优化, 会自动进行判断, 如果前后传入的`props`相同, 则不会重复更新



### 2. 函数组件: 

- memo包裹函数, memo会帮助我们判断前后传入的`props`是否相同, 如果相同则不会重复更新

```jsx
const Profile = memo(function(props){
    return <h2>Profile: {props.message}</h2>
})
```



- **注: 子组件重新渲染不是指整个子组件重新渲染, 而是根据diff算法, 找到子组件中改变了的地方, 重新进行渲染**

[渲染和提交 • React (jscn.org)](https://beta.react.jscn.org/learn/render-and-commit)



### 3. 函数组件改变比较逻辑

```jsx
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```





## 五. 受控组件(双向绑定)和非受控组件

### 1. 受控组件

- 表单元素如`input`, `textArea`, `select`一旦绑定一个**`value`属性**, 就会变为受控组件, 无法自由输入输出
- 使如`input`, `textArea`, `select`这样的组件中自己的`state`与React组件中的`state`同步起来, **使React的`state`成为唯一数据源**
  - 因此, 这种组件必须由`setState`改变状态, 否则无法改变
  - **监听`onChange`事件**, 包括`input, textarea, select`

- 将`value`设置为`null, undefined`仍可编辑

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
<input type="file"/>  // 它的value是只读的, 因此是非受控组件
```



如果你想寻找包含验证、追踪访问字段以及处理表单提交的完整解决方案，使用 [Formik](https://jaredpalmer.com/formik) 是不错的选择



## 六. 高阶组件

**高阶组件本身是一个函数, 接受一个组件作为参数, 返回值为一个新的组件**

- 可以对传入的组件进行一层封装
- 进行数据劫持设置封装



- `this.forceUpdate()`: 强制执行render函数





## 七. Portals(挂载元素)

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



