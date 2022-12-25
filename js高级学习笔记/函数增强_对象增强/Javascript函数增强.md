[toc]



# Javascript函数增强

## 一. 函数对象的属性

### 1. 函数可以自定义添加属性

```js
function foo(){}
foo.message = 'Hello world'
console.log(foo.message)
```



### 2. 函数具有一些默认属性

- length : 传入参数的个数, 具有默认值的参数会忽略
- name: 函数的赋值名字



## 二. 函数的arguments

```js
function foo(m,n){
    // 打印传入的所有参数(以列表)
	console.log(arguments)
    // 可以用索引值
    console.log(arguments[0]
}
foo(10,50,10,20)
```

arguments 是一个对应于传递给函数的参数的类数组(array - like)对象

- 它是一个对象类型, 但是能用一些数组的特性,如:length,还可以通过index进行访问, 但它没有数组的一些方法,如filter,map等
- 为了使用其中的一些数组方法,我们可以将arguments转为数组
  1. 通过for of 和 push新建一个新的数组
  2. 利用Array.from(iterable)传入可迭代对象,自动生成新数组
  3. 利用展开运算符(...), newArray = [...arguments]
  4. 调用slice方法`const newArray = [].slice.apply(arguments)`,将slice的this绑定给arguments,进行一次浅拷贝, 或者`const arr = Array.prototype.slice.apply(arguments)`

- 箭头函数内,不绑定arguments



## 三. Javascript纯函数

- **在相同的输入值时,产生相同的输出(不能随机不定)**
- **函数的输出和输入值以外的其他隐藏信息或状态无关, 也和I/O设备产生的外部输出()**
- **该函数不能有可观察的函数副作用**

副作用: 

- 表示在执行一个函数时,除了返回函数值之外,还对调用函数产生了附加的影响,比如修改了全局变量,修改了参数或者改变外部的存储.(不能直接修改外部变量)



- slice(): 生成一个新的列表,不改变原列表,因此是一个纯函数
- splice(): 会改变原列表,因此是一个纯函数



## 四. 函数增强

### 1. 纯函数的优势

- 在编写时不需要担心外层的作用域的变量的值
- 在调用时不需要担心改变外面的值, 同时也明白确定的输出能得到确定的输入



### 2. 柯里化(Currying)

柯里化属于函数式编程里面非常重要的概念:

- 是一种关于函数的高阶技术
- 其他支持函数式编程的编程语言也有

解释:

- 把接收多个参数的函数, 变成接收一个单一参数(最初函数的单一参数),并且返回接收余下的参数的函数的技术
- **如果你固定某些参数,你将得到接收余下参数的一个函数**, f(10,20,30) -> f(10)(20)(30)
- 只给函数传递一部分参数,让它返回一个函数去处理剩余的参数

```js
// 柯里化函数
function foo(x){
    return function(y){
        return function(z){
            console.log(x+y+z)
        }
    }
}
foo(10)(20)(30)

//另外一种写法
const foo = x => y => z => console.log(x+y+z)
```

```js
// 编写自动化柯里化函数
function Curring(fn){
    return function curringFn(...args){
    	if(args.length > fn.length){
            fn(...args)
            // fn.apply(this, args)
        } else {
            return function newCurringFn(...newArgs){
                curringFn(...args,...newArgs)
            }
        }
    }

}
```



### 3. 组合函数

```js
function composeFn(...fns){
    const length = fns.length
    if(length < 0) return 
    fns.map((fn) => {
        if(typeof fn !== 'function') throw new Error('must be function')
    })
    
    return function(...args){
        const result = fns[0].apply(this,args)
        for(const i = 1; i< length ; i++){
            result = fns[i].apply(this, [result])
        }
    }
}
```



### 4. with语句

- with语句用来扩展作用域链(不建议使用)

```js
const obj = {
    message: 'Hello world'
}

with(obj){
    console.log(message)
}
```



### 5. eval函数

- 内建函数eval**允许执行一个代码字符串**(不建议使用)
- 将最后一句执行语句的结果,作为返回值

```js
const message = 'Hello world'
const codeStr = 'const name = "zzt"; console.log(message); "abc" '
const result = eval(codeStr) // "abc"
```



## 五. 严格模式

- JavaScript新的特性被加入,旧的功能也没有改变, 有利于兼容新旧代码, 因此任何错误或不完善也会永远留在JavaScript语言中
- 因此通过严格模式(Strict Mode),可以使浏览器进行更加严格的检查和执行
- 严格模式通过抛出错误, 来消除一些与那有的静默(silent)错误(发现错误但不报错)
- 严格模式让js引擎在执行代码的时候可以进行更多的优化
- 严格模式禁用了在ECMAScript未来版本中可能会定义的一些语法

严格模式的开启:

- 在文件的最上方添加"use strict"
- 给函数开启严格模式, 在函数的开头添加"use strict"
- 现代JavaScript的module和class**会默认开启严格模式**
