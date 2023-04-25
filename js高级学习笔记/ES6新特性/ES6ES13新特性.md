[toc]

# ES6~ES13新特性

## 一. 词法环境

词法环境:

- LexicalEnvironment: 用于处理let/const 声明的标识符, let/const 不能提升作用域,会生成暂时性死区,**虽然这个变量被定义了,但在暂时性死区(TDZ)中无法访问**.
- VariableEnvironment: 用于处理var和function声明的标识符

暂时性死区(TDZ): 

- 从块作用域的顶部一直到变量被声明完成前, 这个变量一直在暂时性死区中,无法被访问

```js
let message = "11"
function foo(){
    // 会报错
    console.log(message)
    let message = "sss"
}
```



## 二. 添加属性到window

```js
// 1.在全局作用域内, 用var定义的变量会添加到window上
var message = "zz"
// 2. let/const 定义的变量不会添加到window上
// 因此可以看出更新后, 环境记录并不全是window
// let/const等定义的在另一个环境上, 声明环境
console.log(window.message)
```



## 三. let/const 块级作用域限制

```js
{
    let age = 18
    const height = 1.88
    // class也会在作用域
    class Person{}
    // 函数能被外界访问,浏览器js引擎做的优化, 但是只能在后面访问,不能在之前访问
    function foo(){}
}
// 上面这种形式会形成自己的作用域,外界无法访问
```

```js
// 在if中使用var定义变量会将变量定义到全局对象上, 外界可以访问到
// 使用const/let会形成块级作用域, 外界无法找到这个变量, 只能在作用域内使用
if (true) {
  var age = "zzt";
  const height = "1.8";
  let name = "zz"
}
console.log(age);

// 报错
console.log(height);
console.log(name)
```



![image-20221213184047902](.\图片\image-20221213184047902.png)

- 一段”有问题“的代码

```js
// 下面这段代码有问题!!!
// 1.所有的按钮点击时打印的数字永远是最后一个数
const btnEls = document.querySelectorAll("button")
for(var i = 0; i < btnEls.length; i++ ){
    const btnEl = btnEls[i]
    btnEl.onClick = function(){
        console.log(`点击了第${i}个按钮`)
    }
}
// 2.原因:
// 这里定义i时使用var, 导致每次都将i添加到window上,没有单独创造环境, 因此函数闭包时只保存全局环境, 在循环结束后, 全局环境上的i自然为最后一个
// 3.改进方法
const btnEls = document.querySelectorAll("button")
for(let i = 0; i < btnEls.length; i++ ){
    const btnEl = btnEls[i]
    btnEl.onClick = function(){
        console.log(`点击了第${i}个按钮`)
    }
}
// 4.原因
// let会创建一个单独的词法环境, 因此函数闭包时保存的环境就是特定的词法环境, 里面的i也是对应的i
```



## 四. 箭头函数

- 箭头函数没有显示原型prototype, 所以不能作为构造函数来使用, 也就不能用new来创建对象
- 箭头函数也不绑定this, arguments, super参数





## 五. 数值的表示

```js
// 二进制
const num1 = 0b100

// 八进制
const num2 = 0o100

// 十六进制
const num3 = 0x100
```

数字过长时,可以用`_`进行分割



## 六. Symbol的基本使用

### 1. 简单介绍

**Symbol时ES6中新增的一个基本数据类型, 作用是可以生成一个独一无二的值, 可以用来生成属性名**

```js
const s = Symbol()
const obj = {
    [s]: "zz"
}
```



### 2. 从属性中拿到Symbol

- 通过`Object.keys()`获取不到Symbol的key
- 可以通过`Object.getOwnPropertySymbols()`来获取所有的Symbol



### 3. Symbol描述符

```js
const s = Symbol("aaa")
console.log(s.description) // "aaa"
```



### 4. 生成相同的Symbol

通过Symbol()函数生成的Symbol都是独一无二的

```js
const s1 = Symbol("aa")
const s2 = Symbol.for(s1.description)
const s3 = Symbol.for(s1.description)

console.log(s2 === s3)  // true
```



### 5. 获取传入的描述符

```js
const s = Symbol("aa")
console.log(Symbol.keyFor(s))  // "aa"
```



### 6. Symbol.for 和 Symbol的区别

- `Symbol.for()`所定义的Symbol是全局作用域内都相同的, 即使是在别的模块中定义, 只要`description`相同, 那么拿到的Symbol就相同
- `Symbol()`定义的Symbol在本地中是唯一的, 即使是同一`description`返回的Symbol是不相同的

```js
import { ss } from "./script_03.js";
const s1 = Symbol("s1");
const s2 = Symbol("s1");
console.log(s1 === s2);		// false

const s3 = Symbol.for("s2");
const s4 = Symbol.for("s2");
console.log(s3 === s4);		// true
console.log(s3 === ss);		// true
```





## 七. Set的基本使用

- 和数组类似， 但是元素不能重复
- 用于数组的去重
- 传入的是一个可迭代对象
- set支持for of 遍历

```js
const set = new Set()

set.add(10)

const set1 = new Set(arr1)
const arr2 = Array.from(set1)
```



set常见属性和方法:

- size: 获取set的长度
- add: t添加元素
- has(): 判断是否存在某个元素
- clear(): 清空set中的元素
- forEach(): 遍历set所有元素
- delete: 删除某个元素



## 八. weakSet的使用

```js
let obj1 = { name: "Zz"}
let obj2 = { name: "z"}
let obj3 = { name: "Zzz"}

const arr = [obj1,obj2,obj3]
obj1 = null
obj2 = null
obj3 = null
```

内存图:

<img src=".\图片\image-20221214192052518.png" alt="image-20221214192052518" style="zoom:50%;" />

在对象被赋予null后, 三个对象仍会被数组引用, 不会被消除

weakSet和set的区别:

- 只能放入对象
- 对所有对象的引用都是弱引用(即会被GC忽略)
  - 在上面的情况中, arr的三个引用会被当做不存在, 三个对象就会被回收掉

使用案例: 

```js
const pWeakSet = new weakSet()
class Person {
    constructor(){
        pWeakSet.push(this)
    }
    
    running(){
        if!(pWeakSet.has(this)){
            // 异常处理
        } else {
            // 正常处理
        }
            
    }
}

// 通过这样可以使running方法只能被实例调用, 其他方法调用时会报错
// 使用weakSet的用途
let p = new Person()
// 当实例被销毁时, pWeakSet中的对象也会被销毁
p = null

```



## 九. Map的基本使用

- map用来存储映射关系
  - 使用对象存储映射关系KEY只能为string或者Symbol, 但是使用map可以使用其他类型来作为key

```js
const map = new Map()
map.set(info,"ASa")
```

### 1. map常见的属性和方法:

- **支持以 `new Map([[key1,value1], [key2,value2], [key3,value3]])`**的形式定义map

- size获取map的尺寸
- get(key): 根据key获取内容
- delete(key): 删除内容
- has(key): 检测有无内容
- clear(): 清空
- forEach(): 遍历, 直接获取到value
- 支持for of 的遍历和迭代, for of 拿到的是[key, value]形式的内容



### 2. map与object的对比

多数开发任务中, map和object的区别不大, 但当涉及到**内存和性能时**, 两者还是有差别的

- 内存占用
  - map的键值对所占用的内存更小
- 插入性能
  - 两者插入键值对的消耗大致相当, 但map在浏览器中稍快一点, 如果代码涉及到大量的键值对插入, map性能更加
- 查找速度
  - 差异极小, 在吧object当数组用(key为连续number), 浏览器会有优化, 如果设计大量查找, 则object更佳
- 删除性能
  - 如果代码涉及大量删除属性操作, 选map, 使用delete删除object属性的性能饱受诟病



## 十. 字符串填充

1. padStart和padEnd , 在头或者尾部进行位数填充

```js
const minute = "15".padStart(2, "0")  // "15"
const second = "6".padStart(2,"0")  // "06"

const cardNumber = "156415646515631561"
const lastFourNumber = cardNumber.slice(-4)  // "1561"
const finalCardNumber = lastFourNUmber.padStart(cardNumber.length, "*")
```



## 十一. 其他属性和方法

- Trailing Commas : 一种语法, 允许在定义和调用函数时,在参数的尾部加一个逗号
- Object.getOwnPropertyDescriptors(): 获取对象的描述符
- async iterators: 迭代器
- Promise.finally
- **flat(number: 遍历层数): 会按照一个指定的深度递归遍历数组, 并将所有元素与遍历到的子数组中的元素合并作为一个新数组返回**
- flatMap() : 对数组中的每一个元素应用一次传入的map对应的函数 (先map,再flat)

```
const finalMessages = message.flatMap(item => item.split(" "))
```

- Object.fromEntries() 将[[name, "Zzz"], [age , 15]] 这种数组转换为对象
- Object.entries(obj) 将对象转换为 `[[key,value],[key,value]]`这种形式

```js
for ( const [key, value] of objs ){
    ...
}
// 等价
   
for ( const [key, value] of objs.entries()){
    
}
```

- trimStart, trimEnd 去除首尾的字符串
- 可选链 ? 

```js
obj?.friend?.fuction?.()  // 当obj有frined再执行这个代码, 没有的话就不执行
// 函数不存在时不能调用, 所以最后一个括号前加上?.可选链判断前面的属性方法是否存在
```



- for in 遍历对象拿到的是key
- for of 拿到的是迭代属性内返回的value



- FinalizationRegistery类: 对象可以让你在对象被垃圾回收时请求一个回调
  - 通过调用register方法，注册任何想要清理回收的对象， 传入对象和值

```js
let obj = { name: "Zz" }
const finalizationRegistery = new FinalizationRegistery((value) => {
    console.log("有个对象被回收了",value)
})
// 给obj注册, 可以给回调函数传入参数
finalizationRegistery.register(obj, value)
// 使obj被回收,触发函数
obj = null
```



- weakRefs: 弱引用

```js
// 强引用
const obj = { name: "zz" }
const info = obj
obj = null  // 这种情况下对象不会被回收

// 弱引用
const info1 = new WeakRef(info)
// 当obj = null后,对象就会被清除
// 使用时需要解析出来
const info2 = info1.deref()
// info2.name ....
```



- method.at()

```js
const name = ["Zz","zzzz","sadad"]
console.log(name.at(-1))  // "sadad"
// 也可以用于字符串
```



- Object.hasOwn(obj,property) : 判断对象是否具有某个属性, 可用地方更广



- Object.create(obj) : 创建一个对象,该对象的隐式原型(`__proto__`)指向obj



- class中的新成员

```js
class Person {
    // 这里定义的属性是公共的属性,
    height = 1.88
    
    // 约定熟成的私有方式
    _intro = "zzz"
    
    // 真正的私有属性, 这个属性外界实例不能访问, 只能在类中使用
    #intro1 = "zzzzz"
    constructor(name){
        this.name = name
    }
    
    // 静态代码块, 会在new之前执行
    static {
        console.log("创建了一个实例")
    }
}
```



