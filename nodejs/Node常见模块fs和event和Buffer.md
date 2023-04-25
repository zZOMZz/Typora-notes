[toc]

# Node服务器和常见模块

## 一. Node.js是什么

- Node.js是一个基于V8 JavaScript引擎的JavaScript运行时的环境
- Node.js中我们也需要进行一些额外的操作, 比如文件系统读/写, 网络IO, 加密, 压缩解压文件
- 浏览器中包含一些HTML/CSS解析相关的模块(Blink等)



- 如写一个`fs.readFile('.abc.txt',() => {})`, 它会被从左往右**解析成一个操作系统调用**, 然后再一步步返回执行结果.
  - js代码会经过V8引擎, 再通过Node.js的Bindings, 将任务放到Libuv的事件循环中
  - libuv是使用C语言编写的库(可以直接被操作系统执行)
  - **libuv提供了事件循环, 文件系统读写, 网络IO, 线程池等内容**

![Snipaste_2022-12-19_15-23-30](.\图片\Snipaste_2022-12-19_15-23-30.png)



## 二. 内置模块fs

[File system | Node.js v19.4.0 Documentation (nodejs.org)](https://nodejs.org/docs/latest/api/fs.html)

- 借助Node帮我们封装的文件系统, 我们可以在任何的操作系统(window, Mac OS, Linux)上面利用JS直接去操作文件

fs API的三种操作方式:

- **同步操作文件**: 代码会被阻塞
- **异步回调函数操作文件**: 代码不会被阻塞, 需要传入回调函数, 但获取到结果后, 回调函数会被执行
- **异步Promise操作文件**: 代码不会被阻塞, 通过`fs.promise`调用方法操作, 会返回一个Promise, 可以通过then, catch执行处理



```js
const fs = require('fs')

// 1. 同步
const res = fs.readFileSync('./abc.txt',{
    encoding: 'utf-8'
})

// 2. 异步, 有结果后再回调函数
fs.readFile('./abc.txt',{
    encoding: 'utf-8'
},(err,data) => {
    if(err) {
        console.log("出错了")
        return
    }
    console.log(data)
})

// 3. 异步Promise, 避免回调地狱
fs.pormises.readFile().then(res => {}).catch(err => {})
```

- **文件读取出来的data默认是Buffer类型的二进制数组, 可以在读取时就声明encoding解码格式, 也可以之后再toString('utf-8'(默认)), 进行解码;  又或者可以进行单字节修改再解码**



### 1. 文件描述符

- 在操作系统上, 对于每个进程, **内核都会维护着一张当前打开的文件和资源的表格**
- 每个打开的文件和资源都**分配了一个称之为文件描述符的简单的数字标识符**
- 在系统层, 所有的文件系统都使用这些文件描述符来标识和**跟踪**每个特定的文件
- **为了简化用户的操作, Node.js抽象出了操作系统之间的特定差异, 并为所有的文件分配一个数字型的文件描述符**



```js
fs.open('.bbb/txt',(err,fd) => {
    if(err) return
    // 1. 拿到文件描述符
    console.log(fd)
    
    // 2. 可以将文件描述符传入到一些API
    // 读取文件的状态
    fs.fsate(fd,(err,stats) => {
        console.log(stats)
        
        // 3. 关闭文件, 避免浪费性能
        fs.close(fd)
    })
    
})
```

- 可以认为操作系统会为每个打开的文件和资源分配一个唯一的**身份ID(文件描述符)**, 可以利用文件描述符找到这个文件



### 2. 文件的读写

- `fs.readFile(path,[options],callback)`文件的读取
- `fs.writeFile(file,data,[options],callback)`文件的写操作
  - options:
    - encoding: `utf-8`
    - flag:  `w`: 打开文件写入; `w+`:打开文件进行读写,如果不存在则创建文件; `r`打开文件进行读取; `r+`打开文件进行读写, <u>如果不存在则抛出异常</u>; `a`打开要写入的文件, 将流放入到文件的末尾. 如果不存在则创建文件;  `a+`打开文件进行读写, 将流放入到文件的末尾, 如果不存在则创建文件



### 3. 文件夹相关操作

```js
// 1. 创建文件夹
fs.mkdir('./aaa.txt',(err) =>{})


// 2. 读取文件夹
// 返回一个数组, withFileTypes会使其包含每个文件的类型(文件夹还是文件)
fs.readdir('./zzt', { withFileTypes: true }, (err, files) => {
    console.log(files)
    files.forEach(item => {
        if(item.isDirectory()){
            
            fs.readdir(..) //  继续读取, 可以改成递归的方式读取
        } else {
            
        }
    })
})

// 3. 重命名文件夹
fs.rename('./ccc.txt','./bbb.txt',(err) => {})
```





## 三. events模块

- 类似于EventBus



```js
const EventEmitter = require('events')

const emitter = new EventEmitter()

// 1. 监听事件
emitter.on('zzt',() => {
    console.log("发出了事件")
})

// 2. 触发事件
emitter.emit("zzt")

// 3. 取消监听
emitter.off('zzt')
```

其他的一些属性:

- **emitter.eventNames()**: 返回当前EventEmitter对象注册的事件字符串数组
- **emitter.getMaxListeners()**: 返回当前EventEmitter对象的最大监听器数量
- **emitter.setMaxListeners()**: 设置当前最大监听器的数量, 默认是10个
- **emitter.listenerCount(事件名称)**: 返回当前EventEmitter对象某一个事件名称下监听器的数量
- **emitter.listeners(事件名称)**: 返回当前EventEmitter对象某一个事件监听器上所有的监听器执行函数数组
- **emitter.removeAllListener(name)**: 不传入参数的时候会全部移除监听事件, 传入则移除传入的监听事件





## 四. 数据的二进制

- Node中读取到的图片数据都是以二进制的数据形式,**`<Buffer .. .. ..>`**
- 前端中很少会与二进制直接打交道, 服务器中需要直接操作二进制的数据
- Node为了方便开发者完成更多的功能, 提供了我们一个类**Buffer**, 并且他是全局的



```js
// 不推荐使用 const buf1 = new Buffer("world") 来进行使用

// 1. 传入字符串生产Buffer
const buf2 = Buffer.from('world','utf-8')
// buf2   <Buffer 77 6f 72 6c 64>
// 一个单词对应一个字节
// 一个中文对应三个字节

// 2.将Buffer转换为String, 传入的编码格式需要与上面的相同
console.log(buf2.toString('utf-8'))


// 3. 创建一个8字节空间的Buffer, 默认为00填充
const buf3 = Buffer.alloc(8)
buf3[0] = 100
buf3[1] = Ox63
// <Buffer 64 63 00 00 00 00 00 00 >
```

- **Buffer相当于是一个字节的数组, 数组中的每一项对应一个字节的大小**
- 默认创建Buffer时使用的是`utf-8`进行编码

### 性能优化

- 在我们创建Buffer时, 为了避免频繁的向操作系统申请内存, 它会默认的先申请一个8*1024字节大小的内存

  (8 kb) , 之后再进行内存空间申请时, 会直接在这个空间里拿取; 如果空间不够用, 会通过createPool再创建一个新的空间



## 五. nodemon

热更新重启服务器

```shell
npm install nodemon -g

nodemon [filename]
```



## 六. glob

**匹配文件**

```shell
npm install glob
```

```js
const glob = require('glob')

 path: glob.sync(`${path.resolve(__dirname,'../src')}/**/*`,{ nodir: true })
```

