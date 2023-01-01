[toc]

# React中编写CSS

## 一. 内联样式

- Style接受一个Javascript对象, 而不是一个字符串
- 可以引用state中的状态来设置相关的样式
- 内联样式不会产生冲突
- 只支持小驼峰样式名称

```jsx
<h2 style={{ color: this.state.titleColor }} ></h2>
```



## 二. 普通的CSS文件

- 直接写CSS文件, 并引入到特定需要的组件文件中
- 缺点: 这样写入的CSS是全局的, 会导致CSS混乱



## 三. CSS Modules

所有使用了类似于webpack配置的环境下都可以使用

- 在其他项目中, 需要我们自己来进行配置, 比如配置`webpack.config.js`中`modules:true`

- React已经内置了css module的配置
  - 将`.css/.less/.scss`等文件全部修改为`.module.css/.module.sass/.module.scss`文件再进行引用
  - 会把类名用动态生成的hash混合class, 形成局部作用域, 避免了冲突

```jsx
import appStyle from "./App.module.css"

render() {
    return (
    	<h2 className={appStyle.title} >HHHHHH</h2>
    )
}
```

缺点: 

- 引用的类名不能包含`-`
- 所有的className都必须使用{style.className}的形式编写
- 不方便动态修改某些样式, 依然需要使用内联样式的方式



## 四. CSS in JS

[styled-components](https://styled-components.com/)

- CSS 由 Javascript生成而不是在外界文件中定义. (All in JS)

- `Material UI`库

- `styled-components`库(最流行)

- `emotions-js`库
- `glamorous`库

### 1. 基本使用

- 可以针对一部分的组件, 再抽成一个组件导出, 避免层级过深

- 因为使用的是`js`文件, 因此可以非常方便的使用组件的状态
- **可以接受外部传入的`props`, 需要在内部定义一个函数, 在组件接收到props后会自动回调**
- 也可以使用`styled.div.attrs({ testColor: props => { return props.color || blue } })`来给props设置初始值(少用, 还有问题)
- 也可以统一定义一些CSS标准, 再通过js引入

```jsx
// 模板字符串的使用, 可以用来调用函数
function foo(...args) {
    console.log(args)
}

foo`my name is ${name}, age is ${age}`
// [['my name is', ',age is',''], name, age]
```

```js
import styled from "styled-component"
// 将AppWrapper引用到最外层, 就可以定义其中组件的CSS
export const AppWrapper = styled.div'
	.section {
        ....
        font-size: ${props => props.size}
        .title {
            ...
        }
    }
'
```

```jsx
render() {
    const { size, color } = this.size
    // 直接用已经定义好样式的div
    
    <AppWrapper size={size}>
 		<section className="section">
            <p>JJJJ</p>
        	<h2 className="title">HHH</h2>
        </section>
    </AppWrapper>
}
```

样式继承:

```jsx
// 会继承XXButton的样式
const ZTButton = styled(XXButton)`
 backGroundColor: "red"
`
```





#### 2. ThemeProvider

- 默认给所有子组件共享一些属性
- 拿取: `${ props => props.theme.color }`

```jsx
import { ThemeProvider } from "styled-components"
render() {
 return (
 	<ThemeProvider theme={{ color: "red", size: "10px" }}>
     	<App/>
     </ThemeProvider>
 )    
}

```





## 五. classnames

- 通过第三方库classnames动态管理是否添加class

```jsx
classNames("foo","bar") //" foo bar "
classNames("foo",{ bar: true }) // " foo bar "
classNames("foo", { bar: false, duck: true }) // "foo duck"
```

