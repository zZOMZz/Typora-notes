[toc]

# Koa框架

- 旨在为Web应用提供**更小, 更丰富和更强大的能力**
- 相对于express具有更强的异步处理能力
- 我们可以根据需要再安装和使用中间件

## 一. 简单使用

```js
const Koa = require('koa')

// 注册中间件, koa的中间件只有两个参数
app.use((context,next) => {
    context.body = "zzz"
    next()
})

const app = new Koa()
```



## 二. context的参数的解析

- koa中并未将req和res分开, 而是将其作为context的属性
  - `context.request :koa封装的请求对象`;  `context.req: Node封装的请求对象`
  - context.response
- `context.params.id`拿到`/:id`等数据
- `context.query` 拿到`/list?offset=25$size=10`等数据
- `context.method`确定是get方法,post方法...
- `context.path`确定路由, 如`/users`

```js
// 使接口适配多种动态参数, 如
// comment/:commentId   icons/:iconsId   users/:usersId
// 避免固定的使用 ctx.params.commentId
const keyName = Object.keys(ctx.params)[0]
const resourceId = ctx.params[keyName]
const resourceName = keyName.replace("Id",'')
```





### 三. Koa路由

- **Koa注册中间件只能使用use方法, 传入的必须为一个回调函数**
- 使用第三方库`@koajs/router`

```js
const KoaRouter = require('@koa/router')

const userRouter = new KoaRouter({ prefix: '/user' })

userRouter.get('/',(ctx,next) => {})
userRouter.post('/login', (ctx,next) => {})
// 注册全部路由
app.use(userRouter.routes())

// 使当进入未注册的方法路由时返回notfound
app.use(userRouter.allowedMethods())
```



## 四. Koa解析传递参数(req)

- **切记不要再`ctx.body`中拿取数据, 因为这是留给给客户端返回的数据的**, 一般都是在koa定义的request对象中拿取数据`ctx.request.body`



### 1.json解析:

-  `npm install koa-bodyparser`使用**koa-bodyparser**中间件

- express中直接使用内置的解析工具`app.use(express.json())`

### 2.urlencoded解析

```js
// 1) params
userRouter.get('/:id',(ctx,next) => {
    const id = ctx.params.id
})

// 2) query
userRouter.get('/',(ctx,next) => {
    const query = ctx.query
    const { offset, size } = ctx,query
})

// 3) post/json
const bodyParser = require('koa-bodyparser')
app.use(bodyParser())
userRouter.post('/json', (ctx,next) => {
    // 解析完成后就能在request中拿到json数据
    const jsonData = ctx.request.body
})

// 4) urlencoded
// bodyParser既可以解析json数据, 也可以解析urlencoded数据
userRouter.post('/urlencoded', (ctx,next) => {
    console.log(ctx.request.body)
    
    ctx.body = "给客户端返回的信息"
})
```

### 3. formData(表格数据)

- 解析需要第三方库`koajs/multer`来帮助解析

```shell
npm install @koa/multer multer
```

```js
const multer = require('@koa/multer')
const formParser = multer()

userRouter.post('/form',formParser.any(), (ctx,next) => {
    // 这样就能拿取到formdata中的数据
    console.log(ctx.request.body)
})
```

### 4. 图片解析(文件上传)

```js
const multer = require('@koa/multer')
const upload = multer({
    // dest: './uploads'
    storage: multer.diskStorage({
        destination(req,file,callback) {
            callback(null,'./uploads')
        },
        filename(req,file,callback) {
            callback(null,file.originalname)
        }
    })
})

userRouter.post('/form',upload.single("avatar"), (ctx,next) => {
    // 拿到文件
    // 多文件上传就是ctx.request.files
    console.log(ctx.request.file)
})
```



### 5. 浏览器拿到图片

- 输入url直接显示图片

```js
// 从数据库中读取到头像的相关数据
const { filename, minetype } = avatarInfo
// 设置响应为图像类型, 避免浏览器直接下载
ctx.type = minetype
// 创建阅读流
ctx.body = fs.createReadStream(`${UPLOAD_PATH}/${ filename }`)
```





## 五. static静态资源

```js
const static = require('koa-static')

app.use(static('./build'))
```



## 六. Koa响应数据(res)

- **通过`ctx.body`的方式响应数据**
- `ctx.status`更改状态码

响应类型:

- string: 字符串类型
- Buffer: Buffer数据
- Stream: 流数据
- Object Array: 对象或者数组
- null: 不输出任何内容
- 如果response.status尚未设置, Koa会自动将状态设置为200或400



## 七. 错误处理方案

- `ctx.app`拿到app, 发出error事件

```js
ctx.app.emit("error", -1003,ctx)

app.on("error", (code) => {
    switch(code) {
        case -1001:
            ctx.body = "未验证"
            break
    }
})
```



## 八. Koa和Express的区别

架构设计上:

- express是完整且强大的, 其中帮我们内置了很多的好用的功能, 如`express.json()`
- koa是简洁和自由的, 它只包含最核心的功能, 并不会对我们使用其他的中间件进行限制.

express和koa框架的核心都是中间件

- 但是他们的中间件执行机制是不同的, 特别是某个中间件包含异步操作时
  - **Koa中的next返回的类型是promise(洋葱模型), express是void**
  - 因此异步的情况下, express只能从上往下执行代码, 无法回头
- **==当后一个中间件是异步的时候, 前一个中间件的next()前必须添加await等待它返回结果, 否则会显示NotFound==**

```js
// Koa 会先执行到底再返回回去
// Koa中的next函数返回值是promise
const Koa = require('koa')
const app = new Koa()

app.use((ctx,next) => {
    ctx.message = "zz"
    next()
    // 第一次会跳过, 后续中间件执行完再返回来执行
    console.log(ctx.message)  // "zz tt"
})

app.use((ctx,next) => {
    ctx.message += "tt"
})

// 执行异步代码
app.use(async (ctx,next) => {
    ctx.message = "zzzz"
    await next()
    // 一般情况下是不会等待异步请求返回结果的
    // 给next前加上await会使next函数等待后续中间件的异步请求完成后, 再调用下方的代码
    console.log(ctx.message)
})

app.use(async (ctx,next) => {
    const message = await axios.get(..)
    ctx.message += message
})

// express是只能从上到下依次返回结果
```

**洋葱模型**

![2892151181-5ab48de7b5013](.\图片\2892151181-5ab48de7b5013.png)

