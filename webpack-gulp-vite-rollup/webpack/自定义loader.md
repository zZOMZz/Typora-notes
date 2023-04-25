[toc]

# 自定义loader

## 一. 简单介绍

- Loader本质上是导出一个**函数(function)**的JavaScript模块, 然后**loader runner库**会调用这个函数, **然后将上一个loader产生的结果或者资源文件传入进去**

- `function(content,map,meta)`
  - content: 资源文件的内容
  - map: sourcemap相关的数据
  - meta: 一些元数据



- loader之间的执行顺序是从下往上



## 二. webacpk如何寻找loader

- 通过配置路径, webpack会自动寻找, 不需要手动导入

```js
// webpack.config.js
// 配置自动寻找loader, 无需导入
module.exports = {
    // 搜索loader的路径, 解析webpack的loader包
    resolveLoader: {
        modules: ['node_modules','./zzt-loaders']
    }
}
```



## 三. 同步loader

- 默认创建的loader就是同步loader

- 返回

  - `return`: 返回正确解析后的数据
  - `this.callback`: 在webpack的loader中可以用于将处理后的结果返回给webpack, 在webpack的loader上下文中定义的方法, 每个loader都会继承这个方法
    - `error`: 错误对象
    - `content`: 处理后的结果
    - `sourceMap`: 生成的Source Map, 可选
    - `meta`: 用于传递额外的信息, 可以是一个对象或者null


  - 使用`this.callback`返回时, `return`选项就会被忽略, 通常需要处理错误情况时, 都会使用`this.callback`

```js
module.exports = function(content) {
    
    const result = content.replace(/foo/g, 'bar')
    
    // 1. 通过return返回
    return result + 'aaa'
    
    // 2. 也可以通过this.callback返回
    this.callback(null,content + "aaa")
}
```



## 四. 异步的loader

- 如果在loader中需要进行一些异步的操作, 在异步的操作完成后再返回loader的处理结果, 这个时候就能使用异步的loader
- **使用`this.async()`拿到callback, 异步调用callback**

```js
module.exports = function(content) {
    const callback = this.async();
    
    setTimeout(() => {
        callback(null,content)
    },2000)
}
```



## 五. 传入loader的参数(options)

- 

### 1. 获取参数

- 使用`this.getOptions()`获取options参数

```js
// webpacl.config.js
module.exports = {
    module: {
        rules: [
            {
                test: /\.js$/,
                use: [
                    {
                        loader: "",
                        options: {		//	传入参数
                            
                        }
                    }
                ]
            }
        ]
    }
}
```



### 2. 校验参数

- 使用`schema-utils`来进行校验

```json
// 编写schema校验准则
{
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "请输入字符"
        }
    }
}
```

```js
const { validate } = require("schema-utils")

module.exports = function(content) {
    
    const options = this.getOptions()
    // 进行校验
    validate(schema,options)
}
```



## 六. 案例(md文件解析)

- 使用`marked`第三方库解析生成html结构
- 使用`highlight.js`第三方库对一些代码中的关键字进行高亮

```js
// md-loader.js
const { marked } = require('marked')
const hljs = require('highlight.js')

module.exports = function(content) {
    // 启用highlight, 会给每个关键字都添加一个类
    marked.setOptions({
        highlight: function(code,lang) {
            return hljs.highlight(lang,code).value
        }
    })
    
    // 将md语法转换为html结构
    const htmlContent = marked(content)
    
    //返回的结果需要是模块化的内容
    const innerContent = "`" + htmlContent + "`"
    const moduleContent = `var code = ${ innerContent }; export default code`
    
    retunr moduleContent
}
```

```js
// 入口文件, 或者保证会被引用的文件
// 有了loader解析之后, 就能通过code拿到md文件的内容
import code from './test.md'
// 导入highlight库中的默认样式
import "highlight.js/style/default.css"

document.body.innerHTML = code
```

```js
// webpack.config.js
module.exports = {
    resolveLoader: {
        modules: ['node_modules', './loader']
    },
    ...
	module: {
        rules: [
            {
                test: /\.md$/,
                use: ['md-loader']  // 需要与你自定义的文件名相对应
            }
        ]
    }
}
```



- **如果想让这个md显示的更好看, 可以使用css来对这些html进行优化, 因此还需要配置css-loader**