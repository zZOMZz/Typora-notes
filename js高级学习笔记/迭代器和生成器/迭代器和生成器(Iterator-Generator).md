[toc]

# 迭代器和生成器(Iterator-Generator)

## 一. for in 和 for of 的区别

- for...of主要用来遍历迭代器, **遍历的是value**
- for...in主要用来获取Array的index和Object的key, **遍历的是key**
  - **迭代器的一种语法糖**


![Snipaste_2022-12-17_22-19-05](.\图片\Snipaste_2022-12-17_22-19-05.png)

## 二. 迭代器

### 1. 迭代器介绍

- 迭代器使用户在容器对象上遍访的对象, 使用该接口无需关心对象的内部实现细节
  - 例如数据库中的光标
- JavaScript中,迭代器是一个具体的对象, 这个对象需要符合迭代器协议
  - 迭代器协议定义了产生一系列值的标准方式
  - **在JavScript中这个标准就是要有一个next方法**
- next方法要求:
  - 无参数或者一个参数,需要返回一个对象`{ done: boolean, value: ... }`
  - 返回的对象需要具有两个属性
    - done(boolean)
      - false: 迭代器可以产生序列中的下一个值
      - true: 迭代器已经将序列迭代完毕
    - value: 返回的任何JavaScript的值



### 2. 可迭代对象

- 包含一个属性: [Symbol.iterator], **如果对象原型链上的父类实现了Iterable接口, 那么这个对象也就实现了这个接口**
- 这个属性是一个函数, 该函数返回一个含有`next`属性的对象, 该next是一个可被执行的函数, next函数返回的值得格式和上面相同
- 因此可以在这个next函数中自由定义该返回什么样的数据

```js
// 1.迭代对象中的属性
const infos = {
    friends: ["kobe", "jams", "zzt"],
    // 可迭代对象得含有这个属性用来返回一个迭代器函数
    [Symbol.iterator]: function(){
        let index = 0
        return {
            next: () => {
                if(index < this.friends.length){
                    return { done: false, value: this.friends[index++] }
                } else {
                    return { done: true }
                }
            }
        }
    }
}
// 通过调用这个函数拿到迭代器
const iterator = infos[Symbol.iterator]()
console,log(iterator.next())
console,log(iterator.next())
console,log(iterator.next())

// 可迭代对象能使用for of
for(const item of infos){
    console.log(item)
}

const students = ["A","v","s"]
const iterator = students[Symbol.iterator]()
interator.next()...


// 2.迭代获取对象的key
const obj = {
    name: "zz",
    age: "18",
    height: "1.8"
    
    [Symbol.iterator]: function(){
        const values = Object.keys(this)
        let index = 0
        return {
            // 1. 可被外界遍历得函数
            next(){
                if(index < keyArr.length){
                    return { done: false, value: values[index++] }
                } else {
                    return { done: true }
                }              
            },
            
            // 2.检测到break就调用这个函数而不是继续调用next
            return(){
                console.log("被中断了")
                return { done: true }
            }
        }
    }

	// 利用生成器改进
	*[Symbol.iterator]: function() {
        const values = Object.keys(this)
        yield* values
    }
}
```

- String, Array, Map, Set, arguments对象, NodeList集合等原生对象都已经默认实现了可迭代协议, 即可以使用for of 循环

可迭代对象使用场景:

- JavaScript中的语法: for...of, 展开运算符,解构赋值, yield
- 创建一些新对象时,传入的需要是可迭代对象: new Map([iterable]), new WeakMap([iterable])
- 一些方法的调用: Promise.all([iterable])
- [iterable]不是指必须传入数组,只需要传入一个可迭代对象就行,比如Set





## 三. 生成器

- 生成器: 是ES6中新增的一种函数控制,使用的方案, 它可以让我们**更加灵活的控制函数什么时候继续执行,暂停执行**
- 生成器函数: 
  - 只能被next调用返回值
  - 需要在function的后面加上一个`*`
  - 生成器函数需要通过yield关键字来控制函数的执行流程
  - 生成器函数的返回值是一个Generator(生成器) **一种迭代器**
  
- ==yield可以看成是一种return==

###  1. 简单介绍:

```js
function* foo(){
    console.log("zzz")
    // yield 后添加参数为value, 前面为第二次next传入的参数
    // 即在next中添加参数,会被添加前所停在的yield接收, 第一次传参数一般直接在函数中传,不在next中传.
    
    const name = yield "aaaa" 
    
    console.log("sss",name)
    const name = yield
    console.log("rrr")
    return undefined
}

const generator = foo()
// generator.next()返回一个对象: { done: false (取决于是yield返回的还是return返回的), value: yield和return后的值 }
// 一旦return,done就是为true

generator.next() // "zzz" 
generator.next("第二次执行") // "sss 第二次执行"
generator.next() // "rrr"
```

```js
generator.return("aaa") // 返回{ done: true, value: "aaa" }
```

### 2. yield语法糖(yield*)

- `yield*` 需要返回一个**可迭代对象**, 如数组, `for...of`遍历这个结果时, 会依次调用这个可迭代对象的迭代器

```js
const names = ["aa","ss","cc","aaa"]
function* createArrayIterator(arr){
    // 一种语法糖, 会依次迭代这个可迭代对象
    yield* arr
}
const namesIterator = createArrayIterator(names)
namesIterator.next()
namesIterator.next()
namesIterator.next()
```

```js
// 递归
function* nTimes(n) {
    if(n>0) {
        yield* nTimes(n-1)
        yield n-1
    }
}

for(const x of nTimes(3)) {
    console.log(x)
}

// 0
// 1
// 2
```



### 3. await async 解析

- await 和 async 是生成器(generate)和Promise的语法糖

```js
function* getData(){
    // 多次执行异步操作, 一个操作需要等到上一个操作的结果
    const res1 = yield requestData(data1)
    const res2 = yield requestData(res1 + data2)
    const res3 = yield requestData(res2 + data3)
}

const generator = getData()
// 发动第一次请求
generator.next().value.then((res1) => {
    // 第二次请求
    generator.next(res1).value.then((res2) => {
        // 第三次请求
        generator.next(res2).value.then((res3) => {
            generator.next(res3)
        })
    })
})
```





## 四. 利用生成器改进迭代器函数

### 1. 改进迭代器(iterator)

```js
// 1. 在函数中
const names = ["zz","ss","ssss"]

function* createArrayIterator(arr){
    for(let i = 0; i < arr.length; i++ ){
        yield arr[i]
    }
}

const namesIterator = createArrayIterator(names)
namesIterator.next()
namesIterator.next()
namesIterator.next()

// 2. 在对象中
const obj = {
    name: "zz",
    age: "18",
    height: "1.8"
    
    *[Symbol.iterator](){
    	yield* Object.keys(this)
	}
}

// 3.在类中
class Person {
    constructor(friends){
        this.friends = friends
    }
    // 加*表示是生成器函数
    *[Symbol.iterator](){
        // 加*是一种语法糖
        yield* this.friends
    }
}
// 这样即使是Person的实例也能使用for...of遍历
```



## 五. 对象解构和数组解构

```js
// 数组解构
let [a,b,c] = arr
// 对象解构
let { name, age } = obj
// 扩展运算符
let arr2 = [...arr1]
let obj2 = { ...obj1 }
```



- 解构可以分为两种:
  - **数组解构([...rest])**: 会调用`[Symbol.iterator]`函数返回的迭代器
    - 使用`let[a,b] = arr`和`const arr2 = [...arr]`均会调用
  - **对象解构({...rest})**: 不具备`[Symbol.iterator]`方法, 因此会通过**枚举属性进行赋值**的方法进行解构, 会查看属性的`enumerable`描述符, 本质是**创建新对象, 属性赋值**
    - `const obj1 = { ...obj2 };`不会调用`obj2`的`[Symbol.iterator]`即使他已经自定义了
    - 包括解构`let { name, age } = obj1`
  - 上面的写法不严谨, ...是扩展运算符, 不能算解构
