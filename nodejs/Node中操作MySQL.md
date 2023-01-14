[toc]

# Node中操作MySQL

- 使用mysql2(在早期的mysql库中进行了优化和改进)
  - 更好/更快的性能
  - Prepared Statement(预处理语句) **提高性能, 防止SQL注入**
  - 支持Promise, 可以使用async和await语法

## 一. 简单使用

### 1. 安装

```shell
npm install mysql2
```

### 2. 使用

```js
const mysql = require('musql2')
// 1. 建立连接
const connection = mysql.createConnection({
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: '...',
    database: ''
})
// 2. 执行操作语句
const statement = ' SELECT * FROM `student`; '
// structure query Language: DDL/DML/DQL/DCL
connection.query(statement,(err,values,fields) => {
    if(err) return 
    // 查看结果
    console.log(values)
})

// 3. 断开连接, 不能一直保持连接状态
connection.destory()
```



## 二. Prepared Statement(预处理语句)

- **提高性能:**将创建的语句发送给MySQL, 然后MySQL编译(解析, 优化, 转换)语句模板, 并且存储它但是不执行, 之后我们在真正执行是会给`?`提供实际的参数才会执行, 就算执行多次, 也只会编译一次, 所以性能最高
- **防止SQL注入**: 之后传入的值不会像模块引擎那样就编译, 那么一些SQL注入的内容就不会被执行; or 1 = 1不会执行, 因为SQL语句已经解析过了, 因此`or 1 = 1`不会被当成是一个条件语句 ,而是普通的字符串

```js
const mysql = require('musql2')
const connection = mysql.createConnection({
    host: 'localhost',
    port: 3306,
    user: 'root',
    password: '...',
    database: ''
})
// 1. 编写一个预处理语句
const statement = 'SELECT * FROM `products` WHERE price > ? AND score > ?;'
// 2.执行一个预处理语句
// 之后执行时就不会再编译了
connection.execute(statement,[1000,8],(err,value) => {
    console.log(value)
})
```



## 三. Connection Pools(连接池)

- 连接池可以在需要的时候**自动创建连接**, 并且**创建的连接不会销毁**, 会被**放到连接池**中, 后续可以继续使用
- 可以在创建连接池的时候设置最大创建个数LIMIT

```js
// 1. 创建一个连接池
const connectionPool = mysql.createPool({
    host: 'localhost',
    port:3306,
    database: 'music_db',
    user: 'root',
    password: '...',
    connectionLimit: 5
})

// 2. 使用时就当成正常的connection就行
```



## 四. Promise的使用

- 避免回调地狱

```js
connectionPool.promise().execute(statement,[1000,8]).then(res => {
    const [values,fields] = res
    console.log(res)
}).catch(err => {
    console.log(err)
})
```

