[toc]

# JavaScript 对象的增强

## 一. 对象属性的控制

- 默认情况下,我们能对对象的属性进行许多控制, 但我们可以对**对象的属性的控制进行限制**



- 可以通过**属性描述符**对一个对象的属性进行比较精准的操作控制
- 属性描述符需要使用Object.defineProperty 来对属性进行添加或者修改

### 1. Object.defineProperty

- Object.defineProperty() 方法会直接在一个对象上定义一个新属性, 或者修改一个对象的现有属性, 并返回此对象

`Object.defineProperty(obj, prop, descriptor)`

接收的三个参数:

- obj: 需要定义属性的对象
- prop: 需要定义或者修改的属性名称或Symbol
- descriptor: 要定义或修改的属性描述符

返回值: 修改后的对象



descriptor属性描述符:

- 数据属性描述符
- 存取属性描述符

 

|            | configurable | enumerable | value      | writable   | get        | set        |
| ---------- | ------------ | ---------- | ---------- | ---------- | ---------- | ---------- |
| 数据描述符 | 可以         | 可以       | 可以       | 可以       | ==不可以== | ==不可以== |
| 存取描述符 | 可以         | 可以       | ==不可以== | ==不可以== | 可以       | 可以       |

#### 1.1 数据属性描述符

-  [[Configurable]]: 表示该属性是否可以通过delete删除属性, 后续是否可以再次修改它的特性, 或者是否可以将它修改为存取属性描述符
  - 当我们直接在对象上定义一个属性时, 这个属性的[[Configurable]]默认为true
  - 当我们通过属性描述符定义一个属性时, 这个属性的[[Configurable]]默认为false
- [[Enumerable]]: 表示属性是否可以通过for-in进行遍历或者 Object.keys()是否会返回该属性
  - 在对象上定义某一个属性时, 默认为true
  - 通过属性描述符定义的某个对象属性, 默认为false
- [[Writable]]: 表示是否可以修改属性的值, 可以设置为只读
  - 在对象上直接定义时为true
  - 通过属性描述符定义新属性时默认为fasle
- [[Value]]: 表示该属性的值
  - 修改属性的值

#### 1.2 存取属性描述符

主要是为了监听属性的变化

```js
const obj = {
    name: "zzt"
}
let _name = ''
Object.defineProperty(obj,"name",{
    // 可以用来监听设置变化
    set: function(value){
        console.log("set方法被调用")
        console.log("新的值为:", value)
        _name = value
    },
    get: function(){
        return _name
    }
})
// 会调用描述符内的set方法
obj.name = "邹哲韬"
// 会调用get方法
console.log(obj.name)
```

也可以直接在对象中编写存取描述符

```js
const obj = {
    _name: ''
    set name(value){
        this._name = value
    }
	get name(){
        return this._name
    }
}
```





### 2. Object.defineProperties

直接在一个对象上定义多个新的属性或修改现有的属性,并且返回该对象

格式:

```js
Object.defineProperties(obj,{
	name:{
		configurable: true
	},
	address:{
		configurable: true
	}
})

```



### 3. 额外方法补充

- getOwnPropertyDescriptor: 获取当前对象某个属性的描述符

- getOwnPropertyDescriptors: 获取多个属性的描述符

  