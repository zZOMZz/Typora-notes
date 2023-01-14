[toc]

# Express框架

- 基于http模块
- **Express框架的核心就是中间件(middleware)**
- express是一个由路由和中间件的Web框架, 本质上是一系列中间件函数的调用



## 一. 基本使用

### 1. 安装

- 直接利用脚手架, 创建一个应用的骨架

```shell
# 安装脚手架
npm install -g express-generator
# 创建项目
express [name]
```

- 从零搭建自己的express应用结构

```shell
# 先生成package.json
npm init -y
# 安装express
npm install express
```



### 2. 使用

```js
const express = require('express')

// 1. 创建服务器
const app = express()

// 3. 处理请求
app.post('/login', (req,res) => {})
app.get('/home', (req,res) => {})


// 2. 启动服务器, 并且监听端口
app.listen(9000, () =>{
    
})
```



## 二. 中间件

**作用:**

- 中间件可以执行任何代码
- 修改req,res对象
- 结束请求-响应周期(返回数据`res.end(); 或者 res.json({ message:＂zz" })`)
- 调用栈中的下一个中间件, **如果当前中间件功能没有结束请求-响应周期, 就必须调用next()将请求和响应传入下一个中间件中**

```js
// 1. 路由中的回调函数也可以看成是一个中间件
app.get('/login',(req,res,next) => {
    next()
})
// 2. 注册一个中间件, use注册的中间件任何路由任何方式的请求都能匹配
app.use((req,res,next) => {
    next()
})

// 3. 注册一个路径匹配中间件
app.use('/main',(req,res,next) => {
    next()
})
```

- `app.get('router',[中间件1],[中间件2],...)`

### 1. 解析JSON的中间件

```js
// 1) 自己实现
app.use((req, res, next) => {
  if (req.headers["content-type"] === "application/json") {
    req.on("data", (data) => {
      req.body = JSON.parse(data.toString());
      // console.log(req.body);
    });
  } else {
    next();
  }
  req.on("end", () => {
    next();
  });
});
// 之后的中间件通过req.body获取传过来的JSOn数据

// 2) 通过express中间件实现, 传入的json数据就能在body中拿取
app.use(express.json())
```

### 2. urlencoded解析

- 解析客户端传过来的urlencoded数据(`x-www-form-urlencoded`)

```js
// 默认是使用querystring解析, 但过旧不推荐, 因此使用extended可以使用qs第三方库解析
app.use(express.urlencoded({ extended: true }))
```



### 3. 一些第三方中间件

#### 3.1 morgan请求日志的记录

```js
const fs = require('fs')
const morgan = require('morgan')
const writeStream = fs.createWriteStream('./zz.log')
app.use(morgan('combined',{ stream: writeStream }))
```

#### 3.2 multer文件上传中间件

```js
// 简单使用
const multer = require('multer')

// destination文件存储路径
const upload = multer({
    dest: './upload'
})

// 上传一个文件使用single, 传入文件的key名
app.post('/avatar', upload.single('avater'), (req,res,next) => {
    // 获取上传文件的信息
    console.log(req.file)
})
```

```js
// 自定义文件名和地址
const upload = multer({
    storage: multer.diskStorage({
    	// 决定文件的路径, req: req请求相关信息, file: 文件相关信息, callback: 
    	destination(req,file,callback) {
    		// 第一个参数有无错误, 第二个文件路径
    		callback(null,'./upload')
		},
        // 自定义文件名字
        filename(req,file,callback) {
            // file里的originalname自带后缀名
            callback(null,file.originalname)
        }
	})
})
```

```js
// 上传多文件
app.post('/photos',upload.array('photo'),(req,res,next) => {
    // 变成一个数组
    console.log(req.files)
})
```



#### 3.3 multer解析formData

```js
const multer = require('multer')

const formData = multer()

app.post('/login',formData.any(),(req,res,next) => {
    // 解析出的表格数据会以key-value对象的形式保存在req.body中
    console.log(req.body)
})
```





## 三. express数据传输和返回

### 1. 匹配客户端传递的参数

下面的两种传递参数的方式, express会默认解析

- **querystring**`/home/list?offset=20&size=20`
- **params**`/users/:id`

```js
// 1. querystring
app.get('/home/list',(req,res,next) => {
    const query = req.query
    // { offset: '20', size: '20' }, value默认都是string
})

// 2. params
app.get('/users/:id', (req,res,next) => {
    const id = req.params.id
})
```



### 2. response

```js
// 1. end方法返回类型
res.end("zzz")
// 2. json方法返回, 可以传入任何类型: object, array, string, boolean, number..他们会被格式为json类型的数据
res.json()  // 使用最多
// 3. 改变状态码
res.status(201)
```



## 四. express路由

- 一个路由Router实例拥有完整的中间件和路由系统
- 因此也被称为**迷你应用程序**

```js
const userRouter = express.Router()

userRouter.get('/')		// 匹配'/users'
userRouter.post('/:id')	// 匹配'/users/:id'

app.use('/users',userRouter)
```



## 五. 静态资源服务器

- 访问一些服务器上的静态资源(图片)

```js
app.use(express.static('./uploads'))
// 使用这个中间件之后可以直接使用http://localhost/[filename]在浏览器中访问资源

// 部署打包完成后的资源, 在浏览器中默认访问index.html会找到build文件夹下的文件, 从而继续将其与代码下载下来
app.use(express.static('./build'))
```





## 六. 返回错误信息

- 方式一

```js
res.status(401)
res.json("未授权")
```

- 方式二

```js
// 不设置statusCode, 默认为200, 使用自己公司定义的状态码
res.json({
    code: -1001,
    message: "未授权"
})
```

```js
app.use((req,res.next) => {
    if(...) {
        // 在next中传入的参数会被传入到err中 
   		next(-1001)   
    }
})

app.use((err,req,res,next) => {
    // 根据传入的错误码来定义错误
    switch(err) {
        case -1001:
            message = "未授权"
    }
    
    // 返回错误信息
    res.json({
        message,
        code
    })
})
```

