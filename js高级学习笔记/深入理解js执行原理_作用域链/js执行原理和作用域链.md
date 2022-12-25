[Toc]

# js执行原理和作用域链

## 1.初始化全局对象

- 在解析js代码时会初始化一个全局对象(GlobalObject)
- 这个对象所有scope都可以访问
- 里面包含Date,Array, String, Number, setTimeout, setInterval
- 还包含window对象

```js
console.log(message) // 打印undefined
const message = 'Hello World'
console.log(message) // 打印Hello world
```

- 当初次解析时,会给赋值的变量赋值为undefined,给函数赋值为其对应函数对象的内存地址, 因此上方的第一个message不会报错

## 2. 执行上下文

js的引擎可分为两个部分

- 其中stack是执行代码的地方
- HEAP是存储类的地方

![image-20221211164120803](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221211164120803.png)

1. 首先系统会在堆中初始化一个全局对象, 在栈中初始化一个全局的执行栈, 并将这个全局执行上下文的VO(Variable Object)与堆内的全局对象关联起来
2. 在执行上下文中执行的代码,在查询变量时,首先查询的是自己的VO,若在自己的VO找不到,则从上往下依次查找下方执行上下文的VO,直到底部
3. 执行函数会创建一个新的执行上下文, 这个新的上下文的VO对应函数的AO(Activation Object), AO就是函数中自己作用域内定义的变量和函数
4. 当一个执行上下文执行完毕,就会撤出调用栈,底层的全局执行上下文只有在程序执行完成后才会退出



### 3. 函数作用域

- **最重要的一点函数的作用域只与哪定义的有关,与它在哪调用的无关**
- 函数调用时,会在stack中生成新的调用栈
- 函数定义时,会在HEAP堆内生成新的函数对象

1. 无论是在全局中定义函数,还是在函数中定义函数,都会对应的在堆中生成新的函数对象,这个函数对象会有一些name,length属性,还有[[scope]]作用域链属性, **函数的作用域链是在它定义时就已经确定了的**
2. 在堆中的函数对象的**[[scope]]**属性,是一个列表,反应了定义这个函数的定义位置即作用域链, 如在全局内定义一个函数,则**[[scope]]**包含window,在一个全局函数内定义,则**[[scope]]**包含这个全局函数的AO对象和window

```js
const message = 'Global'

function foo(){
	console.log(message)
}

const obj = {
	bar: function(){
		const message = 'Object'
		foo()
	}
}
// 会打印'Global', 因为在其解析函数时,函数的作用域就已经确定了
obj.bar()
```

**在上面这个例子中,之所以打印的是'Global', 是因为bar中message是在它的VO中,而调用foo会创建一个新的执行上下文,这个新的执行上下文中没有定义message,而是调用了它,则系统会到foo的函数对象的[[scope]]中寻找message,因为foo是在全局定义的,因此[[scope]]中也只能找到全局的message,因此打印的是'Global'**

