[toc]

# TypeScript泛型编程

- 类型的参数化
- T相当于是一个变量, 记录着本次调用的类型, 在整个函数的执行周期中, 可以一直保留着参数的类型

## 一. 简单使用

- 使用<T>接受参数
- `T`: Type的缩写
- `K, V`: key和value的缩写
- `E`: Element的缩写, 元素
- `O`: Object的缩写, 对象

```tsx
// 1. 定义
function bar<T>(arg: T): T {
    return arg
}

// 2. 完整的调用, 手动传入
const res1 = bar<string>("aa")
const res2 = bar<number>(15)

// 3. 简写, 会自动推导
const res1 = bar("aa")
const res2 = bar(15)

// 4. 箭头函数
const arr = <T>(num1: T) => {
  return  num1
}

function a<T>(num1: T) {
  return num1
}
```



## 二. 泛型接口和泛型类

### 1. 泛型接口

```tsx
interface IKun<T> {
    name: string
    age: number
    slogan: T
}
```

### 2. 泛型类

```tsx
class Ponit<T> {
    constructor(public x: T, public y: string)
}
```



## 三. 泛型约束(extends)

```tsx
interface ILength {
    length: number
}

// 限制传入的泛型必须包含一个length属性
// 也就是说T必须实现ILength
function<T extends ILength>(arg: T) {
    //
}
```



## 四. keyof

```tsx
interface IKUN {
    name: string
    age: number
}

type IKunKeys = keyof IKun  // "name" | "age"
// 也就被赋予IKunKesys类型的变量, 只能接受"names"或者"age"这两个参数
// key只接受obj中的key名: "name"| "age"
function getObjectProperty<O, K extends keyof O>(obj: O, key: K) {}
```



## 五. 映射类型(Mapped Types)

- 有时, 一个类型需要基于另外一个类型, 但是不想用拷贝这种做法, 这个时候可以使用映射类型
  - 大部分的内置的工具都是通过映射类型来实现
  - 大部分的类型体操的题目也是通过映射类型来完成
- 建立在索引签名的基础上
- 使用`in`操作符

```tsx
interface IPerson {
    name?: string
    age?: number
}

// 可以添加修饰符或者可选等操作
// 定义映射类型时, 只能用type
type MapType<T> = {
    // 会自动遍历所有的key, 返回的类型全为readonly和必选的(删除了可选修饰符)
    readonly [key in keyof T]-?: T[key]
}

type NewPerson = MapType<IPerson>
```



###  修饰符的符号

- 默认为`+`
- `-`删除后面的修饰符, 可在遍历时清除所有的`? | readonly`





## 六. 内置工具和类型体操

### 1. 内置工具:

#### `ThisParameterType<T>`

- 用于提取一个函数类型Type的this参数类型
- 如果这个函数类型没有this参数(箭头函数)则返回unknow

```tsx
type FooType = typeof foo
type FooThisType = ThisParameterType<FooType>  // 拿到其中的this参数类型
```

#### `OmitThisParameter<T>`

- 用于移除一个函数类型Type的this参数类型, 返回当前的函数类型

```tsx
function foo(this,args){}
type FooType = typeof foo
OmitThisParameter<FooType> // (args) => void
```

#### `ThisType<T>`

- 给this赋予一个类型, 可以与interface用`&`联系起来
- 用作标记一个上下文的this类型
- 用于绑定一个上下文的this


```tsx
// store中的this绑定IState类型, 使typescript能识别出this的类型并用于检测
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

#### `ReturnType<T>`

- 拿到函数类型的返回类型
- 注意不能将函数传入, 而应该**将函数的类型传入**

```tsx
type Calc = (arg1: number, arg2: number) => number
function foo() {
    return "sss"
}

type returnType = ReturnType<Calc>  // number
type returnType = ReturnType<typeof foo> // string
```

#### `Partial<T>`

- 将一个类型中所有的选项都转换为可选的
- 可以利用映射类型来实现

```tsx
interface IKun {
    name: string
    age: number
}

type IKunOptional = Partial<IKun>  // name和age都变为可选的
```

#### `Required<T>`:

- 将所有的选项都变为必选的
- 利用映射类型加上`-`修饰符, `-?`

```tsx
interface IKun {
    name?: string
    age?: number
}

type IKunOptional = Required<IKun>  // name和age都变为必选的
// { name: string, age: number  }
```

#### `Readonly<T>`

- 与上面相似, 也是与映射类型相关

#### `Record<K extends keyof any, T>`

- 定义一个对象的属性(key)和值(value)的类型, 主要用于定义对象
- **TypeScript中会隐式的将number等类型的key转换为string**

```tsx
type res = keyof any // number | string | symbol 能作为key的类型
type MyRecord<keys extends keyof any, T> = {
    [key in keys]: T
}
```

#### `Exculde<keyof T, K >`

- 排除一些key, 返回的类型还是`keyof ` : ` "name" | "age" `这种的

```ts
type Person = {
    name: string,
    age: number,
    hobby: string
}

type newPersonkeys = Exclude<keyof Person, "hobby">
// "name" | "age"


// 实现
// 当U为"bane" | "zzzz"这种多类型时, "bane"也会单独匹配上
type Exclude<T, U> = T extends U ? never : T;
```



#### `Pick<T, K extends keyof T>`

- 构造一个类型, 从Type类型中挑取一些keys

```tsx
type Pick<T, K extends keyof T> = {
    [P in K]: T[P]
}
```

#### `Omit`

- 构造一个类型, 从Type类型中过滤一些属性keys
- 注意`Omit`返回的也是一个interface对象

```tsx
// Omit可以通过Pick和Exclude实现
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>

// 1. 复杂写法
type MyOmit<T, K> = {
    [P in keyof T as P extends K ? never : P]: T[P]
}
// 2. 简化
type MyOmit<T, K extends keyof any> = {
	[P in Exclude<keyof T, K>]: T[P]            
}

interface IPerson {
    name: string
    height: string
}

type IKun = Omit<IPerson,"height">
// IKUN
// { name: string }
```

#### `InstanceType`

- 用于构造一个由所有Type的构造函数的实例类型组成的类型

```tsx
class Person {}

type MyPerson = InstanceType<typeof Person>
// typeof Person 用于拿到Person的构造函数类型, 如:new (...args: any[]) => any
// InstanceType 用于拿到构造函数创建出来的实例的类型
                             
type MyInstanceType<T extends new (...args: any[]) => any> = T extends new (...args: any[]) => infer R ? R : never
```





### 2. 类型体操

- 类型体操: https://github.com/type-challenges/type-challenges



## 七. 条件类型

- 进行条件判断, 基于输入的类型来决定输出的类型
- `SomeType extends OtherType ? TrueType : FalseType`

```tsx
function sum<T extends number | string>(arg1: T, arg2: T): T extends number? number : string

function sum(arg1: any, arg2: any) {
    return arg1 + arg2
}

sum(10,20)
sun("ww","zz")
```



## 八. 在条件类型中进行推断(infer)

- 一种占位推断符, 在条件类型中推断类型变量
  - 只能用于条件类型, 并且用在`extends`关键字之后的类型中`
  - 它的作用是告诉 TypeScript 编译器，在推断时应该把某个类型的具体部分作为一个**新的类型变量**来处理，而不是将其作为一个具体类型来处理。这样就可以在条件类型中根据不同的条件推断出不同的类型。


```tsx
// 实现ReturnType
// (...args: any[]) => any 函数类型
type MyReturnType<T extends (...args: any[]) => any> = T extends  (...args: any[]) => (infer R) ? R : never
// 返回参数类型
type MyReturnType<T extends (...args: any[]) => any> = T extends  (...args: infer A) => any ? A : never
```

```ts
type ElementType<T> = T extends (infer U)[] ? U : never;

type NumberArray = Array<number>;
type Element = ElementType<NumberArray>;  // number
```



## 九. 分发联合类型

- 在使用泛型`<T>`时, 进行类型推导和计算时, 对于联合类型`number | string`会将其拆分成多个子类型, 并对每个子类型进行操作和计算, 最终形成新的联合类型

- 在泛型中使用条件类型时, 如果传入一个联合类型, 就会变成分发的

```tsx
type toArray<T> = T extends any? T[] : never

// 生成string[] | number[] , 而不是 (string | number)[]
type NumberAndString = toArray<number | string>
```



## 十. 函数签名中的泛型

```ts
// 这是一个使用interface定义的函数类型
// 因为useSelector是两层函数嵌套的, 因此这里也是两层函数
interface TypedUseSelectorHook<TState> {
    // <TSelector>意思是这个函数可以输入确定一个类型, 如果返回的类型不符, 则会报错
    <TSelector>(selector: (state: TState) => TSelector) : TSelector
}

// 相同
type TypedUseSelectorHook<TState> 
	= <TSelector>(selector: (state: Tstate) => TSelector) =>  TSelector

// TSelector的作用
import { TypedUseSelectorHook, useSelector } from "react-redux";

const useAppSelector: TypedUseSelectorHook<AppState> = useSelector;

// 确定返回的值是number, 可以自动推导, 也能使其自己推导
const count = useAppSelector<number>((state) => state.count);
```

- 表示类型为`TypedUseSelectorHook`的函数可以接收一个参数`<TSelector>`, 确定其返回值的类型, `foo<TSelector>()`, 如果没有传入则会自行进行推导
- `TState`使TypeScript能自动推导出`State`的类型, `TSelector`使TypeScript能推导出返回值的类型
