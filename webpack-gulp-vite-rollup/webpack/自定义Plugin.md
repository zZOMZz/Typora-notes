[toc]

# 自定义Plugin

## 一. tapable简单介绍

- **webpack有两个非常重要的类: Compiler和Compilation**
  - 它们通过注入插件的方式, 来监听webpack的所有生命周期
  - 通过创建Tapable库中的各种Hook的实例, 使插件的注入可以使用各种的Hook

- **Tapable**是官方编写和维护的一个库, **它管理着需要的Hook, 这些Hook可以被应用到我们的插件中**



## 二. tapable的Hook分类

- **bail:** 当有返回值时, 就不会执行后续的事件触发了
- **loop**: 当返回值为true时, 就会反复执行该事件, 当返回值会undefined或者不返回时, 就退出事件
- **waterfall**: 当返回值不为undefined时, 会将这次返回的结果作为下次事件的第一个参数
- **parallel**: 并行, 不会等待上一个事件回调结束, 并行执行,  会立即执行下一次事件处理回调
- **series**: 串行, 会等待上一次事件回调的结果

![Snipaste_2023-01-23_21-09-38](.\图片\Snipaste_2023-01-23_21-09-38.png)



## 三. 使用hooks

```js
const { SyncHook, AsyncParalleHook } = require('tapable')

// 自定义模拟一个compiler类
class ZTCompiler {
    constructor() {
        this.hooks = {
            // 1. 创建hook, 传入的参数为接收的参数名
            syncHook: new SyncHook(['name','age']),
            parallelHook: new AsyncParalleHook(['name','age'])
        }
        
        // 2.用hooks监听事件(自定义plugin)
        this.hooks.syncHook.tap('event1',(name,age) =>{
            console.log("event1被触发了")
        })
        this.hooks.syncHook.tap('event2',(name,age) =>{
            console.log("event2被触发了")
        })
        // 2.1 异步的hook
        this.hooks.parallelHook.tapAsync('event3',(name,age) =>{
            // 可以执行异步的事件
        })
        // 2.2 异步series
         this.hooks.seriesHook.tapAsync('event3',(name,age,callback) =>{
            // 可以执行异步的事件
            // 最后一个series事件callback会真正执行, 前面的callback是为了执行下一个事件
            callback()
        })
    }
}

const compiler = new ZTCompiler()
// 3. 发出事件, event1和event2都会被触发
compiler.hooks.syncHook.call("zzt",18)
// 3.1 触发异步事件, 使用callAsync
compiler.hooks.parallelHook.callAsync("zzt",18)
// 3.2  series需要传入callback来明确告知执行下一个事件, 并且给最后一个事件执行
compiler.hooks.seriesHook.callAsync("zzt",18,callback)
```



## 四. 自定义plugin

- Plugin是如何被注册到webpack的生命周期中:
  - 第一: 在webpack函数的createCompiler方法中, 注册了所有的插件
  - 第二: 在注册插件的时候, 会调用插件函数或者插件对象的apply方法
  - 第三: 插件方法会接收compiler对象, 我们可以通过compiler对象来注册Hook的事件(`this.hooks.compiler.tap()`注册事件), 注册的事件会在compiler被调用时触发
  - 第四: 某些插件也会传入一个compilation的对象, 我们也可以监听compilation的Hook事件



## 五. 案例(静态资源部署到服务器)

- 等待assets已经输出到output上时(**afterEmit Hook**)
- node中远程连接服务器需要借助`node-ssh`第三方库

```shell
npm install node-ssh -D
```

```js
const { NodeSSH } = require('node-ssh')
class AutoUploadWebpackPlugin {
    constructor() {
        this.ssh = new NodeSSH()
    }
    
    apply(compiler) {
        // 通过hook在不同的生命周期进行调用触发
      compiler.hooks.afterEmit.tapAsync('autoUpload',async(compilation,callback) => {
			// 1. 获取到输出文件夹的路径
            const outputPath = compilation.outputOptions.path
            
            // 2. 连接服务器
            await this.connectServer()
          	
          	// 3. 上传文件
            await this.ssh.execCommand("rm -rf /root/test/*")  // 先删除原有文件
          	await this.uploadFiles(localPath,remotePath)
          
          	// 4. 关闭ssh连接
          	this.ssh.dispose()
          	
        	// 5. 完成所有的操作后, 调用callback
            callback()
        })
		       
    }
    
    async connectServer() {
        await this.ssh.connect({
            host: "",
            username: "root",
            password: "xxx"
        })
    }
    
    async uoloadFiles(localPath,remotePath) {
        const status = await this.ssh.putDirectory(localPath,remotePath,{
            recursive: true  // 递归的上传
            concurrency: 10  // 并发的上传
        })
        
        if(status) {
            console.log("文件上传成功")
        }
    }
}
```

