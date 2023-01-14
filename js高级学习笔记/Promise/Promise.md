[toc]

# Promise

**状态:**

- pending
- fulfilled
- rejected

## 一. Promise中resolve的值

1. resolve返回的是一个普通值时, Promsie成功被resolve进入fulfilled的状态
2. resolve返回的是另一个Promise, 则当前Promise的状态还是由返回的这个Promise决定, 如果返回的Promise被reject了, 那么当前的Promise会进入rejected状态, 调用catch里的callback

```js
const promise = new Promise((resolve,reject) => {
    // 1. resolve一个普通值, 正常获取
    
    
    // 2. resolve一个promise
    // 如果传入的是一个promise, 那么当前的Promise的状态就会由传入的Promise的状态决定, 只有传入的Promise resolve了, 当前Promise才会resolve
    
    // 3. resolve一个含then的对象
    resolve({
        name: "zz",
        then: function(resolve){
            // 这里传入的值,会传入外面的then
            resolve(111)
        }
    })
})

promise.then((res) => {
    console.log(res)
})。catch((err) => {
    console.log(err)
})
```



## 二. Promise的类方法

### 1. Promise.resolve

**在已有异步数据的情况下使用**

```js
const promise = Promise.resolve(promiseArr)
promise.then()
```

Promsie.reject()同理



### 2. Promise.all()

- Promise.all([arr])
  - 等待传入的所有Promise变为fulfilled状态, 并将返回值放入一个数组
  - 一旦有一个promise被reject了,那么整个promise就是reject的
- 被接收的的`res`也会是一个对应顺序的数组



### 3. Promise.allSettled()

- 大致和all相同
- 但不会被reject的Promise所影响, 这个Promsie的结果一定是fulfilled



### 4. Promise.race()

- 等待第一个返回的Promsie

- 获得第一个Promise的值
- 无论是fulfilled还是reject的结果



### 5. Promise.any()

- 等待返回的第一个`fulfilled`状态的Promise

- 与race相似,会返回第一个返回的Promise的值
- 但这个值必须是fulfilled的状态
