[toc]

# Proxy-Reflect使用详解

## 一. 监听对象属性的操作

### 1. Object.defineProperty()

```js
// 一种方法, VUE2
const keys = Object.keys(obj)
for( const key of keys){
	let value = obj[key]
	Object.defineProperty(obj,"name",{
        set(newValue){
        	// 监听属性操作
        	value = newValue
    	},
    	get(){
        	return value
    	}
})
}
```

- 利用Object.defineProperty()监听的缺点:
  - 设计初衷并不是用来监听对象中的属性, 它将普通属性强行变为数据属性描述符
  - 对于在遍历后新增属性和删除属性无能为力



### 2. Proxy(代理)

- ES6中新增的一个类,用来帮我们创建一个代理对象
  - 如果我们希望监听一个对象的相关操作, 那么我们可以先创建一个代理对象
  - 之后所有对该对象的操作, 都可以通过代理对象来完成

```js
// new Proxy(obj,handler)
const obj = {
    name: "zzt",
    age: 18,
    height: 1.78
}

const objProxy = new Proxy(obj,{
    set(target,key,value){
        console.log(`${key}属性发生改变`)
        target[key] = value
    },
    get(target,key){
        return target[key]
    },
    has(target,key){
        return key in target
    }
})

console.log( "age" in objProxy ) // true
```

1. 当改变objProxy对象时, 会**默认修改**被代理对象obj中的值,但无法做其他操作
2. 如果想要监听某些具体操作, 就在handler中添加捕获器
   1. set函数有四个参数
      1. target: 目标对象, 被监听的对象obj
      2. property: 将被设置的属性key
      3. value: 新属性值
      4. receiver: 调用的代理对象,**如用于改变调用的set/get中的this指向,使其指向receiver**
   2. get函数的三个参数
      1. target: 目标对象, 被监听的对象obj
      2. property: 被获取的属性key
      3. receiver: 调用的代理对象

3. 即使后续在代理对象objProxy上添加一个属性,原obj上也会被添加
4. Proxy的handler中还有许多其他的**捕获器**用来监听对代理对象的其他方法
   1. handler.getPrototypeof()
      1. Object.getPrototypeOf()获取原型的方法捕获
   2. handler.setPrototypeOf()
      1. Object.setPrototypeOf() 设置原型方法捕获
   3. handler.isExtensible()
      1. Object.isExtensible()方法的捕获（判断是否可以新增属性）
   4. handler.preventExtensions()
      1. Object.preventExtensions()方法的捕获
   5. handler.getOwnPropertyDescriptor()
      1. Object.getOwnPropertyDescriptor()方法的捕获
   6. handler.defineProperty()
      1. Object.defineProperty() 方法的捕获
   7. handler.ownKeys()
      1. Object.getOwnPropertyNames()方法
      2. Object.getOwnPropertySymbols()方法的捕获
   8. **handler.has()**
      1. in 操作符的捕获
   9. **handler.get()**
      1. 属性读取
   10. **handler.set()**
       1. 属性设置
   11. **handler.deleteProperty()**
       1. delete操作符的捕捉器
   12. handler.apply()
       1. 函数调用操作的捕捉器
   13. handler.constructor()
       1. new操作符的捕捉器



```js
function foo(){
    
}

const fooProxy = new Proxy(foo, {
    apply(target,thisArg, otherArgs){
        console.log("执行了apply")
        target.apply(thisArg,otherArgs)
    },
    constructor(target,argArray){
        return new target(...argArray)
    }
    
})
```



## 二. Reflect的作用

- Reflect是ES6新增的API
- 提供了许多操作JavaScript对象的方法, 其中一些与Object中操作对象的方法**相似**

为什么要用Reflect:

- Object是一个构造函数, 很多操作放到它身上并不合适, 所有集成了Reflect,把这些操作都放到Reflect上
- 因此Reflect上的一些API更加严谨,可用性更高, 可以处理操作失败的情况

```js
const obj = {
    name: "zz"
    age: 18
}

const objProxy = new Proxy(obj, {
    set(target,key,newValue,receiver){
        // Reflect的set会返回一个boolean值,用于判断赋值是否成功, recevier可以改变接收器中this的指向
        const isSuccess = Reflect.set(target,key,newValue,receiver)
        if(!isSuccess){
            throw new Error("Set failure")
        }
    }
})
```



- Reflect.constructor

```js
function Person(name.age){
    this.name = name
    this.age = age
}
function Student(name,age){
    // Person.call(this,name,age)
}
// 通过Reflect.constructor来构建实例
const stu = Reflect.constructor(Person,["zzt",18], Student)
console.log(stu.__proto__ === Student.prototype)
```

