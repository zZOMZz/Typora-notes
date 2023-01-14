[toc]

# TypeScript知识拓展

## 一. webpack环境配置

```shell
tsc --init # 创建tsconfig.json
npm install webpack webpack-cli -D
npm install ts-loader -D
npm install html-webpack-plugin -D
npm install webpack-dev-server -D
```

```tsx
// webpack.config.js
const path = require("path")
const HtmlWebpackPlugin = require("html-webpack-plugin")

module.exports = {
    mode: "development",
    entry: "./src/index.ts",
    output: {
        path: path.resolve(__dirname, "./dist")
        filename: "bundle.js"
    },
    resolve: {
        extension: [".ts",".js",".cjs",".json"]
    },
    module: {
        rules: [
            {
                test: /\.ts$/
                loader: "ts-loader"
            }
        ],
        plugins: [
            new HtmlWebpackPlugin({
                template: "./index,html"
            })
        ]
    },
    devServer: {}
}

// tsconfig.json
module.export = {
    // 目标转换的JS版本
    target: "es2016"
}
```



## 二. tsconfig.json

[TypeScript: TSConfig Reference - Docs on every TSConfig option (typescriptlang.org)](https://www.typescriptlang.org/tsconfig)

- 作用:

  - 让TypeScript Compiler在编译时, 知道如何去编译TypeScript代码和进行类型检测

  - 让编辑器(VSCode)可以按照正确的方式识别TypeScript代码



- 如何被使用
  - 在调用tsc命令并且没有其他输入文件参数时, 编译器将由当前目录开始向父级目录寻找包含tsconfig文件的目录
  - 在调用tsc时, 也可以输入--project(-p), 来指定包含了tsconfig.json的目录
  - 当命令行指定了输入文件参数, tsconfig.json文件会被忽略



```json
{
    compilerOptions: {
        // 目标代码的版本
        "target": "esnext"
        // 生成代码使用的模块化
        "module": "exnext",
        // ts中进行严格的类型检测
        "strict": true
        // jsx的处理方式(保留原有的jsx格式), 留给babel转换
        "jsx": "preserve"
        // 跳过对整个库进行检测, 避免冲突
        "skipLibCheck": true
        // 路径的映射, 类似webpack的alias, 配置之后再编写代码时会有提示
        path: {
        	"@/*": ["src/*"]
    	}
    }
    // 编写一个数组, 用于指定项目中包含哪些文件, 在文件数较少时使用
    files: [] 
    // 编写一个数组, 用于指定在项目中包括哪些文件, 默认匹配的时所有文件
    include: []  
    // 编写一个数组, 用于指定从include中排除哪些文件
	exclude: []
}
```

