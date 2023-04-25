[toc]

# 跨域问题(CORS)

## 一. 问题产生

- CORS(Cross origin request)

- 从一个部署的服务器`natours.com`中调用和访问另一台服务器`example.com`上的API和资源, 这两个服务器存在于不同的域中, **设置跨域请求甚至存在于不同的子域, 不同的协议, 不同的端口中**
- **跨域请求一般都会被浏览器阻止而失败, 除非实现跨域资源共享(cross origin resource share)**
- 在服务器端发出跨域请求能正常通过, 只有在浏览器中会被阻止



## 二. 解决跨域

阅读源码expressjs/cors

[expressjs/cors: Node.js CORS middleware (github.com)](https://github.com/expressjs/cors)

```shell
npm install cors
```

```js
const cors = require('cors')

// 1. 只适用于简单的get, post request
// 将会给我们的response添加几个特定的headers
// 主要是: Access-Control-Allow-Origin: '*'
// 也可以只配置几个特定的API接口
app.use(cors())

// 2. 复杂的request: delete,put 和 patch, 或者发送cookie
// 这些请求需要一个预检(preflight)阶段, 浏览器自动发送option request, 判断这个复杂请求是否可以安全发送
app.options('*',core())
```

