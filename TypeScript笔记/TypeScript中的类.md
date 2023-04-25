[toc]

# TypeScript中的类

## 一. 基本使用

```tsx
class Person {
	// 1. 成员属性需要先声明, 也可以设置初始化值
	name: string = "zzt"
    age: number
    
    constructor(name: string, age: number) {
        this.name = name
        this.age = age
        // constructor会自动返回当前实例
    }
}

const p1: Person = new Person()
```



## 二. 类的继承

- 使用extends 和 super(这里与JS相同)

```tsx
class Student extends Person {
	sno: number
    
    constructor(name: string, age: number, sno: number) {
        super(name,age)
        this.sno = sno
    }
}
```



## 三. 类的修饰符

### 1. 介绍3种修饰符

- `public`:公有的属性和方法, 任何地方都可使用(类与实例), **默认定义**的属性和方法就是这种
  - 只有用`public`定义的属性和方法能在类的外部(实例)中被使用

- `private`: 仅在自身中可见, 私有的属性和方法, 实例和子类无法访问
  - 在`private`属性和方法上, 子类除了能继承外, 其与一切都得自己实现
  - 因为如果父类中有`constructor`构造函数, 子类也必须传入对应的参数, 因此即使是私有属性也得继承, 但无法访问

- `protected`: 仅在自身和子类中可见, 受保护的属性和方法, 实例无法访问
  - 与`private`修饰符不同, `protect`修饰符定义的属性和方法能在子类中直接使用: `this.age`, `this.getAge()`, 调用父类中直接定义的方法




- `readonly`: 只读属性, 不能写入操作, 只可以在初始化时写入数据
  - 也可以在对象中使用(type | interface)

### 2. 使用修饰符的技巧

```tsx
// 利用public可以简写constrctor函数
class Person {
    name: string
    age: number
    
    // 这样就不用在内部再额外定义了
    constructor(public name: string, public age: number){}
}
```




## 四. 类的setter和getter

- 对类中属性的访问进行**拦截操作**, 可以在中间进行一些额外的操作
- 需要使用`private`

```tsx
class Person {
    private _name: string
    age: number
    
    constructor(name: string, age: number){
        this._name = name
    }
    
    set name(newValue: string) {
        this._name = newValue
    }
    
    get name(): string {
        return this._name
    }
}
```



## 六. 参数属性

- 可以看作一种语法糖

- 可以在构造函数参数前**添加一个可见性的修饰符(public/private/protected/readonly)来创建参数属性**

```tsx
class Person {
    // 使用修饰符的参数不需要再在上面先声明
    
    constructor(public name: string, private _age: number){
        // 也不需要在内部在定义了
        // 不需要 this.name = name
    }
}
```



## 七. 抽象类abstract

```tsx
// 父类, 一个抽象类
// 抽象类无法被实例化
abstract class Shape {
    // 定义一个抽象方法, 声明没有实现体, 具体的实现体得有子类自己定义, 且必须得定义
    // 抽象方法只能存在于抽象类中
    abstract getArea()
}

class Circle extends Shape {
    
    // 子类中必须实现这个抽象方法, 不然会报错
    getArea() {
        ...
    }
}
```

### 1. 抽象类和接口的区别

```tsx
interface IShape {
    getArea: () => number
}
class Circle implements IShape {
    getArea() {
        // 
    }
}
```

- 抽象类是事物的抽象, 抽象类用来捕捉子类的通用特性;  **接口通常是一些行为的描述, 对类的关系并不关心**
- 抽象类通常用于一系列关系紧密的类之间, 接口只是用来描述一个类应该具有什么样的行为
- 接口可以被多层实现,同一个类可以使用多个接口 , 而抽象类只能单一继承
- 抽象类中可以有实现体, 接口中只能有函数的声明
- 抽象类是一种`is a`的关系; 接口是`has a`的关系



## 八. 鸭子类型

 ```tsx
 class Person {
     constrctor(public name: string, public age: number) {}
 }
 
 class Dog {
     constrctor(public name: string, public age: number) {}
 }
 
 function printPerson(p: Person){
     ...
 }
 // 不会报错
 printPerson(new Dog("zz",15))
 printPerson({ name: "ss", age: 15 })
 ```

- TypeScript对类型进行检测时, 是**鸭子类型**: 如果一只鸟看起来像鸭子, 那他就是鸭子
- **只关心属性和行为, 不关心你具体是否是对应的类型**