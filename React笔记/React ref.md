[toc]

# React ref

## 一. 类组件

1. `createRef`, 先创建再绑定

```jsx
import { createRef } from "react"

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

- 也可以传入函数

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

## 二.  函数组件

### 1. 继续使用createRef

- **利用forwardRef**加强后的函数组件可以**拿到父组件传入的ref**, 在接收的第二个参数

  1. 我们通过调用 创建了一个 [React ref](https://react.docschina.org/docs/refs-and-the-dom.html) 并将其赋值给 变量。`React.createRef``ref`
  2. 我们通过指定 为 JSX 属性，将其向下传递给 。`ref``<FancyButton ref={ref}>`
  3. React 传递 给 内函数 ，作为其第二个参数。`ref``forwardRef``(props, ref) => ...`
  4. 我们向下转发该 参数到 ，将其指定为 JSX 属性。`ref``<button ref={ref}>`
  5. 当 ref 挂载完成， 将指向 DOM 节点。`ref.current``<button>`

- 本组件中使用ref, 需要`useRef Hook`

```jsx
improt { forwardRef } from "react"

// 传给函数式组件的ref会被传入到函数的接受变量ref中
const HelloWorld = forwardRef(function(props,ref){
    return(
    	<h2 ref={ref}>HHH</h2>
    )
})
const ref = createRef()
// 将ref传递给HelloWorld组件中, 会被forwardRef中的第二个参数接收
<HelloWorld ref={ref} />
```



### 2.**使用`useRef` Hook**

```jsx
import { useRef } from 'react'

fucntion app() {
	const ref = useRef(null)
    
    return (
    	<div ref={ref} >aaa</div>
    )
}
```

