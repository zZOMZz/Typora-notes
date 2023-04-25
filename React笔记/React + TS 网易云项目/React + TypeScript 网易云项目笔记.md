[toc]

# React + TypeScript 网易云项目笔记

## 一. 配置项目

### 1. 搭建环境

- 搭建ts环境

```shell
create-react-app [name] --template typescript
```

### 2. 配置项目icon

- 将一个同名的icon图片`favicon.ico`替换掉原public文件夹下的文件就行

### 3. 配置标题

- 直接在public文件夹下的`index.html`中修改title标签, 如果需要代码动态修改, 则利用`document.title`来进行修改

### 4. 自行修改webpack(比如配置别名)

- 利用craco这个第三方库
- 再将`script`中的脚本也改为由craco启动

[Getting Started | CRACO](https://craco.js.org/docs/getting-started/)

```shell
npm i -D @craco/craco
npm i -D @craco/types
```

```js
// craco.config.js 一定要有这个文件, 不然会报错
const path = require("path");
const resolve = (src) => path.resolve(__dirname, src);
module.exports = {
  webpack: {
    alias: {
      "@": resolve("src"),
    },
  },
};
```

- **上面这个配置只是保证了编译时不会出错, 但在编写时TypeScript还是识别不了这种写法, 所以还需要配置tsconfig.json**

```json
// tsconfig.json
{
    "compilerOptions": {
        // 指明当前文件所配置的位置, "."表示是当前目录下, 如"src"就表示当前目录下的src
        "baseUrl": ".",
        // 配置别名
        "paths": {
            "@/*": ["src/*"]
        }
    }
}
```



### 5. 配置代码规范

#### 1. .editorconfig

```yaml
# https://editorconfig.org

# 表示当前文件是在根目录
root = true

[*]  # 表示所有文件都适用
charset = utf-8     # 文件字符集
indent_style = space  # 缩进风格(tab | space )
indent_size = 2     # 缩进大小
end_of_line = lf    # 控制换行符类型(lf|cr|crlf)
insert_final_newline = true   # 去除行首的任意空白字符
trim_trailing_whitespace = true # 始终在文件末尾插入新行

[*.md]  # 只.md文件适用
insert_final_newline = false
trim_trailing_whitespace = false
```



#### 2. prettier

```shell
npm install prettier -D
```

```json
// .prettierrc
{
  "singleQuote": true,
  "semi": false
}
```

```yaml
# .prettierignore
/dist/*
.local
.output.js
/node_modules/**

**/*.svg
**/*.sh

/public/*
```

```json
// package.json
{
    "scripts": {
        // 自动格式化所有不被忽略的代码
        "prettier": "prettier --write ."
    }
}
```



#### 3. eslint

```shell
# 下载eslint
npm i eslint -D

# 自动初始化eslint
 npx eslint --init
```

- **npx调用node_modules/bin下面的命令**

 ```shell
 # 配置插件, 使prettier和eslint保持一致的风格
 npm install eslint-plugin-prettier eslint-config-prettier -D
 ```

```json
// eslintrc.js
extends: {
    "plugin:prettier/recommend"
}
```



### 6. 配置less(可选)

- 需要在craco中配置less的插件, 才能使less语法被正确解析

[DocSpring/craco-less: A Less plugin for craco / react-scripts / create-react-app (github.com)](https://github.com/DocSpring/craco-less)

```shell
npm i craco-less@2.1.0-alpha.0 # 需要下载craco-less, 未来可能会发生变化
```

```js
// craco.config.js, 未来发生变化了再查阅文档
const path = require('path')
const CracoLessPlugin = require('craco-less')
const resolve = (src) => path.resolve(__dirname, src)

module.exports = {
  webpack: {
    alias: {
      '@': resolve('src'),
      components: resolve('src/components'),
    },
  },
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        noIeCompat: true,
      },
    },
  ],
}
```





## 二. 目录结构

```
react18_ts_wyymusic
├── README.md
├── craco.config.js
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
├── src
│   ├── App.tsx
│   ├── assets
│   ├── base-ui
│   ├── components
│   ├── hooks
│   ├── index.tsx
│   ├── network
│   ├── react-app-env.d.ts
│   ├── router
│   ├── store
│   ├── utils
│   └── views
└── tsconfig.json

```



## 三. CSS样式重置

```shell
 npm install normalize.css  # 一个用来重置css的第三方库
```

- 写一些全局的重置CSS, 使用less时需要额外配置一些插件(见上)



## 四. 配置路由

```shell
npm install react-router-dom  # 因为只运行在浏览器中, 因此安装这个包
```

