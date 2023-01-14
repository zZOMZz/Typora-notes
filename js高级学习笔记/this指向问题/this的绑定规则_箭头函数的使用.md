[TOC]



# this的绑定规则_箭头函数的使用

## 一. this的简单介绍

### 1. this的四种绑定规则

#### 1.1 默认绑定

独立函数调用:

```js
function foo(){
    console.log(this)
}
// js会默认的给直接函数调用下的this绑定window对象,在严格模式下绑定undefined
foo()
function foo1(fn){
    // 由于在函数内部还是直接调用, 因此还是默认绑定
    fn()
}
// 高阶函数
foo1(foo)
```



#### 1.2 隐式绑定

通过对象调用:

```js
function foo(){
    console.log(this)
}

const obj = {
    name: 'zzt',
    foo: foo
}
// 通过函数调用对象内的函数,会隐式的给this绑定上对应object对象
obj.foo()

const ab = obj.foo
// ab函数由于直接调用,因此this绑定的还是window
ab()
```



#### 1.3 显示绑定

利用apply, call, bind函数

当给这几个函数传入string,number等,如: 123, 'abcd'时,会先生成它们的**包装类型**String,Number; 当传入undefined时,会绑定window

```js
function foo(){
    console.log(this)
}

// 1.apply 传入参数, 以数组的形式在第二个参数上传入
foo.apply('asd',['as',15])
// 2.call 
foo.call(123)
foo.call({ name:'asd' })
// 传入参数, 以参数列表
foo.call('as', 'ss',16)

// 3. bind 返回一个新的函数, 该函数中的this指向绑定的对象
const foo1 = foo.bind(obj)
```



#### 1.4 new绑定

利用new, 可以将JavaScript中的函数当作类中的构造函数使用

**使用new来调用函数,会有以下四个步骤:**

1. 创建一个全新的对象
2. 这个新对象会被执行prototype连接
3. **函数调用中的this会被绑定到这个对象上**
4. 如果函数没有返回其他对象,表达式则返回这个新对象



### 2. 四种绑定的优先级

1.  默认绑定优先级最低

2.  显式绑定的优先级高于隐式绑定
   1. apply ,bind, call绑定的this优先级高于隐式绑定

3. new绑定的优先级也高于隐式绑定
3. new的优先级高于bind
3. bind的优先级高于apply和call



### 3. this规则之外

- 在显示绑定之外,传入null或者undefined,会忽略此显示绑定,将使用**默认绑定规则**,
- 在严格模式下,直接绑定到this,不需要使用包装类型



## 二. 箭头函数

- 箭头函数不会绑定this, arguments属性
- 箭头函数不能作为构造函数使用(不能与new一起使用)
- 如果箭头函数的默认返回值是一个对象, 则需要用小括号包裹这个对象
- 箭头函数有自己的作用域(即在箭头函数中定义的变量,外界无法访问), 但**箭头函数的作用域中不包含this**, 因此**箭头函数中的this是父作用域的this**
- ==学习React时领悟到的一点：==， 由于箭头函数没有自己的this， 因此它所定义的环境非常重要， 可以看作它会把this放入到自己的作用域链中, 因此无论它在哪里被调用了, 它使用的this**始终都是它定义时的环境中的this**

```js
const obj = {
    name: 'obj',
    foo: function(){
        return () => {
            console.log(this)
        }
    }
}

const fn = obj.foo()
fn.apply('aaa')
// 打印的为obj,因为箭头函数作用域中无this,因此使用apply等显式绑定函数时无法绑定, 因此这个this是箭头函数在定义时上层作用域function中的this,而那个this由于隐式绑定指向obj

const obj2 = {
  name: "zzt",
  fn: () => {
    console.log(this);
  },
};

obj2.fn(); // 打印为window, 因为箭头函数没有this,因此在定义时,使用的时上层作用域的this,而这里由于对象不形成作用域, 因此上层作用域内的this指向的时window
```

