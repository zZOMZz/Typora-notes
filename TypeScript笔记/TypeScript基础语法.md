[toc]

# TypeScript基础语法

## 一. 基本介绍

作用:

- 最主要的作用是进行类型校验, 使代码更加**安全和健壮**
- 有些隐藏的问题只有在运行的时候才能发现, 而现在能通过TypeScript在编写时就自动检测出来
- 在语言层面上, 不仅仅增加了类型约束, 而且包括了一些语法的扩展, 比如枚举Enum 和元组Tuple等
- 并且TypeScript最终还是会被编译成Javascript代码, 因此没有兼容性问题, **而且在编译时不可以不借助Babel这样的工具(TSC)**, 但是也能使用babel来转



## 二. TypeScript的编译环境

- TypeScript最终会被编译成JavaScript来运行, 所以我们需要搭建对应的环境



- 编译的两种方式

  - 利用tsc编译, 再执行编译出的js文件
  
  ```shell
  # 下载
  npm install tsc -g
  # 检查版本
  tsc --version
  # 运行
  tsc [filename]
  ```
  
  
  
  - 利用ts-node
  
  ```shell
  # 安装ts-node
  npm install ts-node -g
  
  # ts-node需要依赖tslib和 @types/node两个包
  npm i tslib @types/node -g
  
  # 再进行编译
  ts-node mathj.ts
  ```
  
  

## 三. 基本语法

### 1. 变量的声明

- 声明后的变量就会进行**类型检测**, 声明的类型可以称之为**类型注解(Type Annotation)**

> var/let/const 标识符: 数据类型 = 赋值

```tsx
let names: string[] = ["Aaa","ss","aa"]
let namess: Array<string> = ["aaa","sss"]

let s1: symbol = Symbol("title")

function sum(num1: number, num2: number): number {
    return num1 + num2
}
```







### 2. 类型推导

- 一些简单的赋值语法中, 即使不添加类型注解, TS也会自动进行**类型推导**判断变量的类型, 后续不符合规范的赋值会报错
- let进行类型推导时, 推导出来的是通用类型
- const进行类型推导时, 会推导出**字面量类型**



### 3. 匿名函数的参数

```tsx
const names = ["cba","ss","Aa"]
// forEach中的匿名函数会自动指定类型
names.forEach((item,index,arr) => {})
```

- 一般匿名函数中的参数会自动推导类型
- 因为TypeScript会根据forEach函数的类型以及数组的类型推断出item的类型
- 这个过程称之为**上下文类型(contextual typing)**, 因为函数执行的上下文可以帮助确定参数和返回值的类型



### 4. any类型

- 当我们无法确定一个变量的类型时, 或者它的类型会发生变化, 这个时候我们就能使用`any`类型
- 我们可以对`any`类型进行任何操作, 包括获取不存在的属性, 方法
- 也可以给any类型的变量赋值任何的值, 比如数字和字符串



### 5. unknown类型

- 用于描述类型不确定的变量
- 和`any`类型相似, 但与any类型相反, 在`unknown`类型的变量上进行的任何操作都是不合法的

```tsx
const flag = true
let result: unknown

if(flag) {
    result = foo()
} else {
    result = bar()
}

// 需要手动进行判断
if(typeof result === 'string') {
    console.log(result.length)
}
```



### 6. void类型

- 表示类型为空, 一般在定义函数类型中使用

```tsx
// 表示是一个函数, 且返回值为空
type Foo: (...args: any[]) => void
const foo: Foo = () => {
    ...
}
```

- 但如果函数返回值类型定义为了void, 并不强制不能返回内容



### 7. never类型

- 永远不会被赋予其他类型的变量
- 永远不会返回任何类型(函数)

```tsx
function foo(): never {
    throw new Error("Ssss")
}
// 如果给message添加别的类型时, 会直接报错, 因为switch语句只处理了两种情况, 别的情况下会赋值给never类型
function handle(message: string | number) {
    switch(message) {
        case "string":
            break
        case "number":
            break
        default:
            const check: never = message
    }
}
```



### 8. tuple类型

- 在数组中由于存在类型推导, 因此最好一个数组内存放的都是同一种类型的数据
- 元组中能存放不同的数据类型, 介于数组和对象中

```tsx
const info: [number,string,number] = [15,"sss",555]
```

- 元组中每个返回的元素都有自己的特定的类型, 因此可以根据索引值确定对应的类型



### 9. 交叉类型和联合类型

- 联合类型:

```tsx
type ABC = string | number  // 只需要满足其中的一个条件即可
```

- 交叉类型
  - 交叉类型需要满足多个类型条件, 都要满足

```tsx
type bas = ABC & CBA  // 既要满足ABC类型, 也要满足CBA类型
```



### 10. 类型断言

**as:**

- 在明确知道类型时, 可以手动进行断言, 确定下变量的类型
- 断言只能断言成更加具体的类型: 不能将number断言成string
- 也可以断言成不太具体的情况: number断言成any

```tsx
// 不使用断言的情况下, 拿到的变量类型为: Element | null, 无法正确的调用其中的方法和属性
const imgEl = document.querySelector(".img") as HTMLImageElement
imgEl.src = ".."
```



**非空类型断言:**

- 强制告诉TS, 前面的属性不为空(undefined), **跳过ts在编译阶段对它的检测**, 最好在确定是再使用

```tsx
interface IPerson {
    name: string
    friend?: {
        name: string
    }
}

const info: IPerson = {
    name: "Zzt"
}
// 在访问属性的时候, 可以使用可选链
console.log(info.friend?.name)

// 赋值时不能使用可选链, 需要使用非空类型断言
info.friend!.name = "zzzz"
```



### 11. 类型缩小

- 可以使用`if`判断
  - **typeof**:`typeof ... === ""` 
  - **in操作符**:`if("name" in Point)`
  - **instanceof**`date instanceof Date`

### 12. 函数类型

```tsx
// 需要准确定义好传入参数的名字和类型, 以及返回的参数类型
type add = (num1: number, num2: number) => number
```

- TypeScript不会对传入函数的参数个数进行验证, 多传入的参数会被忽略





## 四. 类型别名

### 1. type

- 自定义类型, 提高复用性

```tsx
type MyNumber = number | string
// 对象类型
type Ponit = {
    x: number
    y: number
}
```



### 2. interface

- **被赋予interface | type类型的对象不能随意添加属性**

```tsx
// 1. 声明对象
interface Point {
    x: number
    y: number
}

// 2. 声明函数
interface f {
  (a:number, b: number): number
}

const add: f = (a, b) => a + b;
```



### 3. 两者的区别

- type不仅能对象类型, 还能定义别的简单类型, 使用范围更广; interface只能用来声明对象(包括函数)
- 声明对象时, interface能重复声明一个对象, 两个对象会合并; 而type不能允许相同名称的别名同时存在
- interface支持**继承:** `interface Point1 exxtends Point {}`
- interface可以被类实现

```tsx
class Person implements Point {
    
}
```

- **如果是定义对象类型, 推荐使用interface; 别的类型type可以解决**



## 五. 函数签名

### 1. 调用签名

- 可以按上面的定义方法赋予函数类型,  但如果需要给函数添加**属性**, 则需要用到函数标签的做法

```tsx
// 可以同时给函数签名赋予属性
interface IFunction {
    name: string
    (num1: number, num2: number): number
}

const bar: IFunction = (num1: number,num2: number) => {
    return num1 + num2
}
console.log(bar.name)
```



### 2. 构造签名

- 类(class), 也可以看作是一个有构造签名的函数

```tsx
class Person {
    constructor(age: number, height: number) {
        this.age = age;
        this.height = height
    }
    
    getName() {
        return this.name
    }
}
interface IPerson {
    // 表示这个函数可以被new调用, 返回一个Person实例
    // InstanceType<typeof Person>是这个实例的类型
    new (): Person
}

function factory(fn: IPerson) {
    const f = new fn()
    return f
}

// 返回一个实例
factory(Person)
```

#### 2.1 构造签名与`InstanceType<typeof Person>`的区别

- **构造签名**是对类(Person)的构造函数(constructor)接收的参数和返回的类型的描述, 对类中定义的属性和方法没有规定, 其返回一个实例, 类型为`InstanceType<typeof Person>`
- `InstanceType<typeof Person>`是对类的实例的属性和方法进行描述, 包括类中定义的属性和方法, 但对构造函数没有规定, 是一个类型

#### 2.2 `InstanceType<typeof Person>`与`Person`类型的区别

- `Person`: 以类型名直接为类型, 表示的是`Person`类本身的类型, 包括静态属性和方法以及构造函数
- `IntanceType<typeof Person>`: 则是更准确的对创造出来的实例的



### 3. 重载签名(重要)

- 在函数通用逻辑的基础上, 通过对函数输入和输出的类型进行重定义, 使函数具有一定处理能力
- 框架里有用到

```tsx
// 1. 重载签名定义, 不需要函数的实现体{}
function add(arg1: number,arg2: number): number
function add(arg1: string,arg2: string): string

// 2. 编写通用的函数实现
function add(arg1: any, arg2: any): any {
    return arg1 + arg2
}

// 3. 在存在重载签名的情况下, 通用函数无法被调用, 只能通过重载签名调用
add(15,15)
add("Ss","EE")
add({Ss: ASS},54) // error, 找不到能被调用的重载
```



### 4. 索引签名

- 该类型的变量能使用定义的索引类型访问
- index: 只能是string类型或者number类型, 别的不支持
- 数字类型索引的返回值, 只能是字符串索引类型返回值的子类

```tsx
interface ICollection {
    [index: number]: string
    [key: string]: any
}
```





## 六. this类型

###  1. 默认类型 

- 在没有对this进行特殊的类型限制时, 默认是**any类型**



### 2. tsconfig.json

```shell
tsc --init  # 生成tsconfig.json文件
```

```json
{
    "noImplicitThis": true  // 不需要模糊的this, 取消隐式赋予的any
}
```

- 在关闭了隐式的this后, TypeScript会根据上下文推导this, 但在不能正确推导时, 就会报错
-  手动绑定this:

```tsx
function foo(this: { name: string }, info: { age: 14 }) {
    console.log(this, info)
}

// 调用, this参数会在编译后消除
foo.call({ name: "Zzt" }, { age: 14 })
```



### 3. this相关内置工具

TypeScript中提供了一些工具类型来辅助进行常见的类型转换, 这些类型全局可用

- ThisParameterType
  - 用于提取一个函数类型Type的this参数类型
  - 如果这个函数类型没有this参数(箭头函数)则返回unknow

```tsx
type FooType = typeof foo
type FooThisType = ThisParameterType<FooType>  // 拿到其中的this参数类型
```

- OmitThisParameter
  - 用于移除一个函数类型Type的this参数类型, 返回当前的函数类型

```tsx
function foo(this,args){}
type FooType = typeof foo
OmitThisParameter<FooType> // (args) => void
```



- ThisType:
  - 给this赋予一个类型, 可以与interface用`&`联系起来
  - 用作标记一个上下文的this类型
  - 用于绑定一个上下文的this


```tsx
// store中的this绑定IState类型
const store: IStore & ThisType<IState> = {
	state: {
        name: "zz",
        age: 18
    },
    eating: function() {
        console.log(this.name)
    }
}
// 实际调用还是得传入this, 因为上面只是给this赋予了类型
store.eating.apply(store.state)
```





## 七. 字面量赋值检测(一种检测机制)(新鲜度)

- 对于第一次创建的对象字面量, 是新鲜的(fresh), **会进行严格的字面量检测, 必须满足类型的检测(不能有多余的属性)**
- 当进行**类型断言**或者**对象字面量类型扩大**时, 新鲜度就会消失              

```tsx
interface IPerson {
    name: string
    age: number
}

function printPerson(p: IPerson) {
    ...
}
    
// 禁止, 不允许随便添加接口定义的属性之外的属性
printPerson({ name: "zz", age: 18, height: 15 })
    
const obj = {
    name: "zz",
    age: 12,
    height: 12
}
// 但这种方式不会报错
// 但如果obj中缺少IPerson中的类型会报错
printPerson(obj)
```



## 八. 枚举类型

```tsx
enum Direction {
    UP = "UP",
    LEFT = "LEFT",
    DOWM = "DOWN",
    RIGHT = "RIGHT"
}
// 不赋值的会默认赋予"0 , 1, 2 ..."从前面一个属性开始递增

console.log(Direaction.UP)  // "UP"
```

