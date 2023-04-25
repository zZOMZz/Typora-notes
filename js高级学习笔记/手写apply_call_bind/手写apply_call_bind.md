[toc]



# 手写apply_call_bind

- **需要使用隐式绑定(this)**

## 一. apply_call

```js
function foo(){}

console.log(foo.apply === Function.prototype.apply ) // true
```



```js
// 只是大致思想, 并不全面
Function.prototype.ztapply = function(thisArg){
    thisArg = (thisArg === null || thisArg === undefined)? window : Object(thisArg)
    // 默认这里面到的this指向的是调用它的函数(隐式调用)
    thisArg.fn = this
    // 将this绑定到传入的第一个参数中
    thisArg.fn()
    delete thisArg.fn // 也可以使用修饰符进行隐藏
}

function foo(){}

foo.ztapply({name: "zzt"})
```



## 二. bind

```js
Function.prototype.ztbind = function(thisArg,...otherProps){
    thisArg = (thisArg === null || thisArg === undefined ) ? window : Object(thisArg)
    Object.defineProperty(thisArg, "fn" , {
        enumerable: false,
        configurable: true,
        writable: false,
        value: this
    })
    // 利用隐式绑定this到thisArg上
    return (...props) => {
        thisArg.fn(...otherProps, ...props)
    }
}
```