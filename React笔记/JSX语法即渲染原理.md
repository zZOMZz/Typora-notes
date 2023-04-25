[toc]

# JSX语法

## 一. 基础用法

```jsx
const element = <div>Hello</div>

const root = React.creatRoot(document.querySelector("#root"))
root.render(element)
```

- JSX是一种Javascript的语法扩展
  - html in js
  - css in js
  - **all in js**

- React认为渲染逻辑本质上与其他UI逻辑存在内在耦合,息息相关
  - UI可能需要绑定事件
  - UI中需要展示数据状态
  - 在状态改变时, 又需要改变UI



### 1. JSX的书写规范

- JSX顶层只能有一个根元素, 一般最外层包裹一个div元素
- 在多行的html代码外包裹一个小括号()
- jsx的标签可以是单标签, 也可以是双标签



### 2. 嵌入变量

- {/* */}作为注释
- 嵌入变量为子元素时
  - Number, String, Array类型的变量, 可以直接显示
  - null, undefined, Boolean类型的变量, 内容为空
    - 可以将其转换为字符串显示
  - Object类型的对象不能作为子元素显示
- 嵌入数组

```jsx
// 伪代码
const message = ["aaa","Sss","fgf","sas"]

const messageEl = message.map((item) => <li>{item}</li>)

render(){
    // 将一个由Element组成的数组直接传入ul中
    return (
    	<ul>
        	{messageEl}
        </ul>
    )
}
```





### 3. JSX绑定属性和类

- 属性: JSX中绑定属性一般都用`{ }`, 只用{}就行
- 类: js中`class`是一个关键字, 因此最后用className来绑定类`<h2 className={``模板字符串``}>hhh</h2>`
  - 多个类时
    - 利用模板字符串
    - 利用数组, 插入时`classList.join(" ")`
    - 利用第三方库, classNames



### 4. 动态绑定样式Style

```jsx
// 需要在{}中绑定一个对象
<h2 style={{ color: "red", fontSize: "30px" }} >Hello</h2>
```



### 5. JSX可以防止注入

- 可以在JSX中安全的插入`{}`用户输入的内容,因为React DOM在渲染所有的输入内容之前,默认会进行**转义**, 所有的内容在被渲染之前都被转换为字符串, 可以有效的防止XSS(跨站脚本的攻击)



### 6. React事件绑定

- React事件命名采用小驼峰(如: onClick)
- 需要通过`{}`来传入一个事件处理函数, 这个函数在事件发生时被执行

#### 1.this绑定(重要)

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

##### 解决方法:

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





### 7. 条件渲染

```jsx
const { isReady } = this.state

let showElement = null
if(isReady){
    showElement = <h2>HHHHH</h2>
} else {
    showElement = <h2>AAAAAA</h2>
}


return (
	<div>
    	{showElement}
    </div>
)

// 也可以使用三元运算符
```

 `&&`: 

```jsx
// 在friend有值的情况下再渲染后面的, friend无值的时候直接跳过后面的
<div>{ friend && <div>{ friend.name + "" + friend.test }</div>}</div>
```





## 二. JSX渲染流程(React.createElement)

1. 遇到任何带标签的`<APP/>`, jsx是通过Babel转换成**React.createElement()**函数调用,创建元素组成**虚拟DOM**
   1. 是利用Babel调用`React.createElement()`这个函数


**两种写法效果相同:**

```react
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```jsx
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

**生成的虚拟DOM:**

```jsx
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```



2. 利用Babel转换其中的html代码, 每遇到一个标签,就会调用`React.createElement("div",{属性},...子元素)`创建一个一个元素,标签内的子元素也会被调用生成children, 从而一级一级的调用下去, 直到全部遍历完成, 生成一个**树结构(虚拟DOM树)**
1. React再根据虚拟DOM渲染成真实的DOM
   1. 虚拟DOM可以做跨平台应用程序, 解析方式不同
   2. 可以在更新时利用diff算法决定更新那里
2. 通过`ReactDOM.render` 让虚拟DOM和真实DOM同步起来, 这个过程叫做协调



- JSX可以看成是`React.createElement(component,props,...children)`的**语法糖**, 所有的jsx最终都会被转换为这个函数调用