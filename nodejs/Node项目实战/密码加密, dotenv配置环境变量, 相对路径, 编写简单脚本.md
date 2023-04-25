# Node项目实战

## 一. 密码加密

- 编写一个中间件, 对用户传过来的密码进行加密

```js
const crypto = require('crypto')

function md5Password(password) {
    const md5 = crypto.createHash('md5')
    const md5pwd = md5.update(password).digest('hex')  // 使用16进制
    
    return md5pwd
}
```



## 二. 登录凭证

- http是无状态的协议, http的每次请求都是单独的请求

### 1. Cookie

- Cookie, **小型文本文件**, 某些网站为了辨别用户身份而存储在用户本地终端上的数据
- **浏览器在发送网络请求时, 会==自动==携带上Cookie**

Cookie的种类:

- **内存Cookie(会话Cookie):** 由浏览器维护, 保存在内存中, 浏览器关闭时Cookie就会消失, 其存在时间是短暂的
- **硬盘Cookie**: 保存在硬盘中, 有一个过期时间(可以设置为负值,需要手动删除), 用户手动清理或者过期时间到了, 就会被删除

```js
// 设置内存Cookie
document.Cookie = "name=zzt"
// 设置硬盘Cookie, 设置过期时间
document.Cookie = "age=12;max-age=60"
```

- **Cookie会有自己的作用域(允许发给哪些URL)**
  - Domain: 指定哪些主机可以接受cookie
    - 不指定默认为origin, 不包括子域名
    - 如果指定Domain, 则包含子域名, 如: 如果设置`Domain = mozilla.org`则Cookie也会包含在子域名如`developer.mozilla.org`
  - Path: 指定主机下的哪些路径可以接受Cookie
    - 例如, 设置`Path=/docs`, 则以下的地址都会匹配`/docs /docs/Web`

​		

```js
// node设置cookie
ctx.cookies.set('slogan','ikun',{
    maxAge: 60		// 60s
})
```

1. 服务器设置cookie

2. 客户端(浏览器)获取到服务器设置的cookie, 并且对它做一个保存

3. 在同一个作用域下进行访问(域名/路径), 会自动携带cookie
4. 服务器可以通过客户端携带的cookie验证用户身份



### 2. Session

- **Session是基于cookie的实现机制**, 可以对value进行加密

- Koa中我们可以通过Koa-session来实现session认证
- 生成sessionid, 防止进行伪造cookie,对原始数据进行加密

```js
const koaSession = require('koa-session')
const session = koaSession({
    key: 'sessionid',
    signed: true	// 加密签名
},app)
// 加盐操作, 会利用数组里的值进行加密
app.keys = ["aa","ssss"]
app.use(session)

// 设置值
ctx.session.slogan = 'ikun'
// 取值
const value = ctx.session.slogan
```



### 3. cookie和session的缺点

- cookie会被附加在每个http请求中, 无形中会增加了流量
- cookie是明文传输的, 所以存在安全性问题
- cookie的大小限制是4kb, 对于复杂的需求来说是不够的
- **对于浏览器意外的客户端(IOS和Android), 必须手动的设置cookie和session**, 不想在浏览器中设置方便
- 分布式系统和服务器集群
  - 分布式系统: 多个系统(可以看成子模块)之间需要共享session_id, 不然难以认证
  - 服务器集群: 部署多个服务器分担高并发的压力





### 4. token

- 在验证用户的账号和密码正确的情况下, 给用户颁发一个**令牌(token)**, 这个令牌可以作为用户后续访问一些接口或者资源的凭证, **我们可以根据这个凭证来判断用户是否有权限来访问** 
- 存在**私钥和公钥**(使用非对称加密)

JWT(JSON Web Token)实现的token由三个部分组成:

- **header**
  - alg: 采用的加密算法, 默认是HMAC SHA256(HS256), **采用同一个密钥进行加密和解密**
  - typ: JWT, 固定值, 通常都写成JWT即可
  - 会通过base64Url算法进行编码(可被解码)
- **payload**
  - 携带的数据, 比如我们可以将用户的id和name传入payload中
  - 默认也会携带iat(issued at), 令牌的签发时间
  - 也可以设置过期时间: exp(expiration time)
  - 会通过base64Url算法进行编码
- **signature**
  - 设置一个**secretKey**, 通过将两个结果合并后进行HMAC SHA256算法
  - HMAC SHA256(base64Url(header)+ . + base64Url(payload) , secretKey)
  - **需要重点保护secretKey**



```shell
# 下载生成token的第三方库
npm install jsonwebtoken
```

```js
// 1. 发送token
const jwt = require('jsonwebtoken')

// 注: 这里用的是对称加密
const secretKey = "aaa"
const payload = {
    name: 'zz',
    id: 15
}
const token = jwt.sign(payload,secretKey,{
    expiresIn: 60  // s
})
```

```js
// 2. 验证token
// 从客户端发送的请求中拿到token
const authorization = ctx.headers.authorization
const token = autorization.replace("Bearer","")
// 如果验证错误会直接抛出异常, 可以用try...catch...捕获
jwt.verify(token,secretKey)
```

#### 非对称加密

- 私钥: private_key 颁发token
- 公钥: public_key 解密验证, 只能用来解密, 不能伪造
- 在分布式系统或者服务器集群中, 只需要暴露公钥, 给予其他服务器验证token的能力即可

**使用openssl生成一对公钥和私钥**

```shell
openssl # 进入OpenSSL命令
genrsa -out private.key 1024 # 生成私钥
						# 输出的是公钥
rsa -in private.key -pubout -out public.key # 生成公钥

# 系统会直接在当前路径下生成两个文件private.key和public.key
```

- 再将生成的文件中的内容利用fs模块读入到系统中

- **验证token的算法要由默认的HS256改为RS256**

```JS
// 生成token时改变设置
const token = jwt.sign(payload,privateKey, {
    expiresIn: 600,
    algorithm: 'RS256',
    allowInsecureKeySizes: true  // 允许生成2048位以下的token
})

// 解密时, 传入算法
const result = jwt.verify(token,publicKey, {
    algorithms: ['RS256'] 
})
```

![](.\图片\Snipaste_2023-01-14_14-46-50.png)

- 解密出来的result结果, 包含传入的payload和过期时间(exp), 创建时间(iat)



### 5. postman编写简单拿取token的脚本

```js
const res = pm.response.json()
pm.globals.set("token",res.data.token)
```



## 三. 相对路径和绝对路径

- **require函数中使用相对路径不会产生问题**
- 但如果使用了fs等模块引用了路径, 就需要注意

```js
// 启动程序, 相对于package.json, 执行文件时是以启动文件的目录为主
nodemon './src/main.js'

// 1. 方式一
// 默认情况下相对目录和node程序的启动目录有关
const PRIVATE_KEY = fs.readFileSync('./src/config/keys/private.key') //正常
const PRIVATE_KEY = fs.readFileSync('./keys/private.key') // 错误

// 2. 方式二, 使用绝对路径
const payh = require('path')
const PRIVATE_KEY = fs.readFileSync(path.resolve(__dirname,'./keys/private.key'))
```



## 四. dotenv配置全局变量

```js
// 根目录下.env
SERVER_PORT = 8000
```

```js
const dotenv = require("dotenv");

// 运行过后在根目录下的.env中定义的环境变量会被自动加在process.env上
dotenv.config();
// 也可以添加别的路径下的.env文件
dotenv.config({ path: '....' })

module.exports = { SERVER_PORT } = process.env;
```

```json
// package.json
// 可能需要安装cross-env, npm i cross-env -D
"scripts": {
    "build": "cross-env NODE_ENV='production' nodemon main.js",
    "dev": "cross-env NODE_env='development' nodemon main.js"
}
```



### 五. 编写自动导入router的脚本

```js
const fs = require("fs");

function registerRouters(app) {
  const files = fs.readdirSync(__dirname);

  for (const file of files) {
    if (!file.endsWith("router.js")) continue;
    const router = require(`./${file}`);
    app.use(router.routes());
    app.use(router.allowedMethods());
  }
}

module.exports = registerRouters;
```

