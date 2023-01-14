[toc]

# Web服务器开发, 文件上传(非框架)

## 一. Stream流

- **一连串的字节, 连续字节的一种表现形式和抽象概念**
- 当我们从文件中读取数据时, 文件的二进制数据会源源不断的被读取到我们的程序中. (一些较大的视频文件)
- 流是可读的也是可写的
- **所有的流都是EventEmitter的实例**



四种基本的流:

- **Writable**: 可以向其写入数据的流(`fs.createWriteStream()`)
- **Readable**: 可以从中读取数据的流(`fs.createReadStrream()`)
- **Duplex**: 同时为Writeable和Readable(`net.Socket`)
- **Transform**: Duplex可以在写入和读取数据时修改或者转换数据(先压缩再写入等..)的流(`zlib.createDeflate()`)

```js
const readStream = fs.createReadStream('./bbb.txt',{
    start: 5,
    end: 18,
    highWaterMark: 3	//  每隔这个间隔调用一次监听函数, 分批次读取
})

// 1. 读取文件触发
readStream.on('data', (data) => {
    console.log(data.toString())
    
    // 暂停
    readStream.pause()
    
    setTimeout(() => {
        // 恢复
        readStream.resume()
    },2000)
})

// 2. 打开文件触发
readStream.on('open',() => {
    console.log("打开了文件")
})
```

```js
const writeStream = fs.createWriteStream('./bbb.txt', {
    flags: 'a'
})
// 写入
writeStream.write('zzzt',(err) => {
    console.log(err)
})

writeStream.on('close',() => {
    console.log("文件被关闭了")
})

readStream.on('end',() => {
    // 关闭可写流
    writeStream.close()
})

// 写入并且自动关闭, 触发close事件
writeStream.end('zz')
```

### 建立管道(pipe)

- 在可写流和可读流之间直接建立管道, 将数据传输, 避免多层监听执行

```js
const writeStream = fs.createWriteStream('./bbb.txt', {
    flags: 'a'
})

const readStream = fs.createReadStream('./bbb.txt',{
    start: 5,
    end: 18,
    highWaterMark: 3	//  每隔这个间隔调用一次监听函数, 分批次读取
})

// 写入流在括号内
readStream.pipe(writeStream)
```



## 二. Http模块



```js
const http = require('http')

// 1. 创建一个服务器
// 传入的回调函数会在被浏览器访问的时候执行
// 初始时会执行两次, 额外的一次是访问favicon.ico, 所以尽量使用postman测试
const server = http.createServer((req,res) =>{
    
    const urlString = req.url
    const urlInfo = url.parse(urlString)
    // pathname: /home/main   query: ?age=18&name="zz"这种
    console.log(urlInfo.query,urlInfo.pathname)
    const queryInfo = new URLSearchParams(urlInfo.query)
    
    res.end("8080端口返回的数据")
})

// 2. 开启对应的服务器, 并且告知其所要监听的端口, 1024以下的端口默认被操作系统分配给别的服务了
// 1024 - 65535
server.listen(8080, () => {
    console.log("服务器成功开启")
})
```

- **createServer中的req和res实际上是一个可读流**

listen函数的三个参数:

- 端口port(不传时, 系统会默认分配)
- 主机host(不传时默认为0.0.0.0)
  - localhost: 本质是一个域名, 通常情况下会被解析成127.0.0.1
  - 12.0.0.1(回环地址): 我们主机发出去的包会被自己接收到
    - 在网络层直接截获, 不往下面的数据链路层传
  - 0.0.0.0 监听IPV4上所有的地址, 再根据端口找到不同的应用程序
    - 在监听0.0.0.0时, 在同一网段下的主机中, 通过IP地址是可以访问的

### 1. query参数

```js
const server = http.createServer((req,res) =>{
    
    const urlString = req.url
    const urlInfo = url.parse(urlString)
    // pathname: /home/main   query: ?age=18&name="zz"这种
    console.log(urlInfo.query,urlInfo.pathname)
    const queryInfo = new URLSearchParams(urlInfo.query)
    
    res.end("8080端口返回的数据")
})
```



### 2. body参数

```js
const server = http.createServer((req,res) =>{
    
    req.setEncoding('utf-8')
    
    req.on('data',(data) => {
        console.log(data)
    })
    
    req.on('end',() => {
        res.end("登录成功")
    })
    
    res.end("8080端口返回的数据")
})
```



### 3. header中的信息

```js
const server = http.createServer((req,res) =>{
    
    console.log(req.header)
    // body中的数据形式, 如application/json
    console.log(req.header["content-type"])
    res.end("8080端口返回的数据")
})
```

- **content-type:**

  - application/x-www-form-urlencoded: 表示数据被编码成以"&"分割的键-值对, 同时以"="分隔键和值

  - application/json: 表示是一个json类型

  - text/plain: 表示是文本类型

  - application/xml: 表示是xml类型

  - multipart/form-data: 上传的是文件

- content-length: 文件的大小长度

- keep-alive:

  - http是基于tcp协议的, 但是在一次请求和响应结束后就会中断
  - 如果头部中包含`connection：keep-alive`,则会一直保持连接, 直到一方中断
  - http1.1中默认添加

- accept-encoding: 告知服务器, 客户端支持的文件压缩格式, js文件可以使用gzip编码, 对应.gz文件

- accept: 告知服务器客户端可接受的文件的格式类型

- user-agent: 客户端相关的信息

- **Authorization**: 获取到客户端传送的token



### 4. res返回响应结果

- **如果没有调用end方法, 客户端会一直等待结果(设置超时时间)**

```js
const server = http.createServer((req,res) =>{
    // 1. 状态码
    res.statusCode = 200
    // 2. 另一种写法
    res.writeHead(200,{
        content-type: "application/json"
    })
    
    // 设置header的信息, 数据的类型及数据的编码格式
    // 1. 单独设置某一个header
    res.setHeader("Content-Type","application/json:charset=utf-8;")
    res.write("Hello world")
    res.end("8080端口返回的数据")
})
```



### 5. axios(底层)

- 在浏览中使用的axios
  - XHR: xmlhttprequest
  - fetch
- 在Node中使用的axios
  - 不存在浏览器中的xhr/fetch函数
  - 使用http模块

```js
const http = require('http')

http.get("http://localhost:8000",(data) => {
    // 解码
    const dataString = data.toString()
   
    console.log(dataString)
})

const req = http.request({
    method:　'POST',
    hostname: 'localhost',
    port: 8000
},(res) => {
    // data也是一个可读流
    res.on('data',(data) => {
        console.log(data)
    })
})

// post请求一定要主动关闭, 表示写入完成
req.end()
```



### 6. 文件上传

- 需要刨除没有用的数据, 只留下文件内容本身, 否则像图片类型的文件就无法正确显示
  - 进行字符串截取, 正则匹配

```js
const http = require('http')
const fs= require('fs')

const writeStream = fs.createWriteStream('./aa.png',{
    flags: 'a+'
})

const server = http.createServer((req,res) => {
    // 图片必须是二进制的
    req.setEncoding('binary')
    
    // 由于req和res都是流
    // req.pipe(wirteStream)
    // 这里的data会包含除image(file)等额外的数据, 字符串截取
    // 一些库会帮我们截取其中的有效数据
    req.on('data',(data) => {
        writeStream.write(data)
    })
    
    req.on('end',() => {
        writeStream.close()
    })
})

server.listen(8000,() => {
    
})
```

```js
// html浏览器发送表单数据
const formData = new FormData()

const inpuEl = document.querySelector('input')
formData.set('photo',inputEl.files[0])

// 发送post请求
axios({
    method: 'post',
    url: '',
    data: formData,
    headers: {
        'Content-type': 'multipart/form-data'
    }
})
```

