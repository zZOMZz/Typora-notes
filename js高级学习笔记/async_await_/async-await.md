[toc]

# async-await

## 一. 异步函数

- async函数返回的是一个Promise
  - 普通的值是resolve
  - 抛出错误是reject
  - 返回一个promise,要由新的promise决定
- async函数中出错,错误会被包裹到reject中, 而不是直接报错, 因此可以被外层的catch所捕获