[toc]



# ES6 原型继承和类

<img src=".\图片\Snipaste_2022-12-12_20-02-53.png" alt="Snipaste_2022-12-12_20-02-53" style="zoom: 50%;" />



## 一. class类

ES6 -> ECMA2015

### 1. 基础介绍

```js
class Person {
    // 当我们创建实例使用new关键字时,会调用这个constructor函数
    constructor(name,age){
        this.name = name
        this.age = age
    }
    
    // 定义实例方法
    // 这里定义的方法是直接定义在Person.prototype上
    running(){
        console.log("running")
    }
}
```

class定义类和function类的区别: class定义类只能通过new关键字来调用



### 2. class中设置访问器(set, get)

```js
class Person {
    constructor(name){
        this._name = name
    }
    set name(value){
        this._name = value
    }
    get name(){
        return this._name
    }
}
```



### 3. 类的静态方法

https://www.scaler.com/topics/javascript/static-methods-in-javascript/

通过添加关键字static, 可以在不创建实例的情况下直接调用类中的这个方法

```js
class Person {
    static randomPerson(){
        
    }
}
// 直接调用
Person.randomPerson()
```

- staic在定义后会被加到constructor函数中, 而不是在prototype中, 因此static method会被子类继承, 但不能被子类的实例调用.![Snipaste_2022-12-15_15-12-19](.\图片\Snipaste_2022-12-15_15-12-19.png)
- static method中的this指向static method所在的类, 因此可以在一个static method中用this访问其他static metnod



### 4. ES6中通过extends实现继承

```js
class Person {
    constructor(name){
        this.name = name
    }
}

class Student extends Person {
    constructor(name,height){
        // super调用父类的constructor方法
        super(name)
        this.height = height
    }
}
```

通过继承将子类的`_proto_`指向父类的prototype



### 5. super关键字

- super.method(...): 来调用父类的方法. `super.running()`
- super.staticMethod(...): 调用静态方法
- super(...): 来调用一个父类的constructor, 这个用法只能在子类的constructor中



### 6. mixin

```js
function mixinAnimal(BaseClass) {
    // 对我们传入的类进行了扩展
    return class extends BaseClass {
        running() {
            console.log("running")
        }
    }
}
```



## 二. babel

babel可以将ES6转成ES5代码

通过去babel官网将class转换成源码阅读学习



## 三. JavaScript中的多态

- **为不同数据类型的实体提供一个统一的接口, 或使用一个单一的符号来表示多个不同的类型**
- 不同的**数据类型**进行同一个操作,表现出不同的行为, 就是多态的体现



- 在父类和子类中定义一个同名函数, 但当进行同样操作时,表现出的结果不同

```js
// 多态的一种表现
function sum(a,b){
    return a+b
}

sum(5,10)
sum("A","B")
```



## 四. 字面量增强

### 1. 属性增强

```js
const name = "zzt"
const obj = {
    name
}
```



### 2. 方法的增强

```js
const obj = {
    // running: function(){}
    running(){
        
    }
}
```



### 3. 计算属性名的增强

```js
const key = "keysads"
const obj = {
    [key]: "zzz"
}
```

