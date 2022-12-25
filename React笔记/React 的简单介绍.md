[toc]

# React 的简单介绍

## 一 React的开发依赖

- **React**: 包含React所必须的核心代码
  - React包中包含react web和react native**所共同拥有**的核心代码
- **React-dom**: react渲染在不同平台所需要的核心代码
  - react-dom针对web和native所不能完成的功能
    - web端: react-dom 会把jsx渲染成真实的DOM, 显示在浏览器中
    - native端(移动端): 将jsx( JavaScript XML)渲染成原生的控件
- **babel**: 将jsx转换为可以被浏览器识别的JavaScript的工具



## 二. 简单应用

- 通过CDN引入React时, 需要在script标签上, 添加`type = "text/babel"`, 让babel解析jsx的语法

```jsx
// React 18 之后的语法
let message = "Hello World"
function btnClick() {
    console.log("Clicked")
}
const root = ReactDOM.creatRoot(document.querySelector("#root"))
root.render(
    <h1>{ message }</h1>
    <button onClick={btnClick}></button>
)
```



## 三. 类组件

```jsx
class App extends React.Component {
    // 组件数据
    construcor(){
        super()
        this.state = {
            message: "Hello World"
            name: "zzt"
        }
 		// 也可以在构造函数中绑定	       
        // this.btnClick = this.btnClick.bind(this)
    }
    
    // 组件方法
    btnClick() {
        console.log(this) // 在类中为实例对象, 但是被元素调用时, this为undefine
        // setState来自继承方法
        // 1. 将state中的值修改掉
        // 2. 自动重新执行render函数
        this.setState({
            message: "tzz"
        })
    }
    
    // 渲染内容
    render(){
        return (
            <h2>{this.state.message}</h2>
            // 需要显示绑定this
            <button onClick={this.btnClick.bind(this)}></button>
        )
    }
}

const root = ReactDOM.creatRoot(document.querySelector("#root"))
root.render(<App/>)
```

### this绑定(重要)

- 注意类方法中的this问题

```jsx
// 绑定元素时, 类似
const foo = app.btnClick()
foo()  // 在严格模式下, this指向undfined, 宽松下指向window
// 因此在React中使用类组件要注意this, 
```

- 在正常的DOM操作时, 如原生js开发, 监听元素, 监听函数里的this指向的是节点对象
- 这里是因为**React并不是直接渲染真实的DOM**, 我们编写的button是一个语法糖, 本质是React的Element对象
- 因此在这里发生监听时, react执行函数并不会绑定this, 默认情况下就是undefined

#### 解决方法:

- 在绑定监听函数时利用bind绑定this, 或者在constructor底部绑定
- 将函数定义为箭头函数:
  - 可以直接将方法定义为箭头函数然后传入其中
  - 也可以传入一个箭头函数调用方法, 因此在调用方法的时候会通过==隐式绑定==将this传入到方法函数中

```jsx
class obj {
  constructor() {
    this.name = "zzt";
  }

  foo = () => {
    console.log("this", this);
    console.log(this.name);
  };
    
  btnClick(){
      console.log("btnClick",this)
  }
    
  render(){
      return (
      	<button onClick={ () => this.btnClick() } >Click</button>
      )
  }
}

const a = new obj();
const fo = a.foo;
fo();
// 打印出
// obj { name: "zzt", foo: () => {...}}
// zzt
```

原因分析: 

```js
const obj2 = {
  name: "obj2",
  // 由于箭头函数没有this, 且普通对象没有作用域, 所以这里的this指向window
  ab: () => {
    console.log("obj2", this);
  },
};

const obj1 = {
  fa: obj2.ab,
  ab() {
    // 1.这里的this还是指向window
    this.fa();
    
    // 2. 这里的this指向这个对象obj1, 隐式绑定
    console.log("obj1", this);
    const aaaa = () => {
      console.log("aaaaa", this);
    };
    // 3. 这里的this指向了这个对象obj1
    aaaa();
  },
  name: "zzz",
};

obj1.ab();
```

**由此分析, 在箭头函数定义时, 它所处的位置十分重要, ==定义时的this将被放入到作用域链中==, 后续无论在什么环境下使用, 都使用的是定义时的this,==而利用function定义的函数中的this, 与它们被调用时的位置有关==**
