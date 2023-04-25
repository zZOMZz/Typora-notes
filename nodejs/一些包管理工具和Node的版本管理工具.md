[toc]

# 包管理工具和Node版本管理工具

## 一. npm

既是包管理工具, 也是registry仓库

本地具有一个缓存, 会优先查找缓存里是否有需要的包

```shell
npm install [dependencies] [-g]
npm uninstall [dependencies]  [-g]
```

### 1. 常见属性:

- private属性, 为true时可以防止误操作npm发布项目
- dependencies: 无论开发环境还是生成环境都需要依赖的包
- devDependencies: 在生成环境不需要的包,如: webpack, babel `npm install webpack --save-dev(-D)`
- peerDependencies: 需要使用这个库,必须安装的库, 你的模块所依赖的特定的外部模块, 且需要与之交互
  - 可以负责兼容问题




- `peerDependencies`和`dependencies`的区别
  - `Dependencies`是指项目运行时必需的依赖项, 而`peerDependencies`是指当前包需要兼容的外部依赖项, 即如果别人想要使用这个库, 则得下载你指定的相关库, 你不需要将这些库包含到自己的代码中;
  - `peerDependencies`不会被自动安装, 如果需要指定, 则必需要手动安装



### 2. 版本规范 x.y.z

- `x`: 主版本号

- `y`: 次版本号

- `z`: 修订号



### 3. 生成文件:

- `package.json`里记录了较为宽松的版本
- `package-lock.json`里记录了特定严格的版本, 以解决不同开发人员合作开发下载库版本不同的问题

​	



## 二. npx

### 1. 作用

- `npx`是一个Node.js工具, 用于执行npm包中的命令. 
- 可以临时安装一个包, 并在安装完成后立即执行其中的命令, 执行完后删除安装的包, 这样可以避免全局安装一些只需要临时使用的工具
- 帮助我们在不污染全局环境的情况下运行一些需要临时安装的命令行工具和脚本



### 2. 使用场景

1. 运行本地安装的命令行工具: 如果安装了一些npm包, 并且想要在命令行中使用它们, 但不想将它们全局安装, 可以使用`npx`来执行这些命令行工具, 这样能避免全局安装包

2. 运行远程仓库中的命令: 当你想要运行某些仓库中的命令, 但又不想将它们下载到本地, 可以使用`npx`来运行这些命令
3. 运行本地项目中的命令: 如果你在开发一个项目, 并且在项目的`package.jsom`文件中定义一些脚本命令, 可以使用`npx`来运行这些命令, 而不需要全局安装这些依赖



### 3. 查找命令

1. 优先查找当前目录下是否存在该命令的可执行文件或脚本文件, 如果找到了, 就会直接执行该文件
2. 如果没在当前目录下找到, `npx`会搜索当前项目的依赖项, 即在当前项目的`node_modules/.bin`目录下寻找该命令
3. 如果没找到, 会向上依次查找父级目录的`node_modules/.bin`目录
4. 如果还没有, 则会在npm的全局安装目录中查找该命令并尝试执行
5. 如果全局`node_modules`目录下也没有找到该命令, `npx`会自动下载安装该命令, 并将其安装到一个临时目录(缓存)中(不是全局安装), 执行完后就将其删除
6. 如果没找到命令, 则会抛出一个错误





### 4. 与npm的区别

1. 功能不同, `npx`的主要功能是在命令行中运行Node.js包中的命令; `npm`则是一个包管理工具, 用于安装,升级, 卸载和管理Node.js包及其依赖项
2. 使用方式不同: 使用`npx`时, 不需要实现安装Node.js包, 而是可以直接在命令行中使用它们; 而`npm`需要先进行`npm install`
3. 安装位置不同: `npx`运行命令时, 会将Node.js包临时安装到本地缓存中; 而使用`npm`安装包时, 会把它们安装到本地的`node_modules`目录中
4. 版本管理不同: `npx`可以让你使用特定版本的Node.js包而不会与本地全局安装的包发生冲突



## 三. yarn工具

- yarn时为了弥补早期npm的一些缺陷而出现的
- 使用方式与npm相似



## 四. cnpm工具

- 中国使用, 有些情况下我们无法很好的从registry下载
- taobao的镜像服务器仓库 --> npm config set registry "url"将库换为taobao的, 之后安装库的时候都默认从镜像服务器安装



## 五. 上传npm

需要生成一个package.json的文件

```js
{
    "name": "发布出去的名字"
    ...
    "keywords": ["xx","aa"],
    "license": "MIT"
}
```

1. 注册账号
2. npm login
3. npm publish



## 六. pnpm

[pnpm](https://pnpm.io/zh/motivation)

performant npm 高性能的npm, 速度快,能节省磁盘空间的软件包管理器, 如果有100个项目, 每个项目都有一个相同的依赖包, 那么磁盘就需要保存100份相同依赖包的副本

### 1. 软连接和硬链接

**硬链接( hard link )**

- 电脑文件系统中的多个文件平等的共享同一个文件存储单元
- 删除一个文件名字后, 还可以使用其他名字继续访问该文件

**软连接(符号连接)**

- 符号链接是一类特殊的文件
- 包含有以一条绝对路径或者相对路径的形式指向其他文件或者目录的引用
- 如: 快捷方式

![Snipaste_2022-12-20_19-45-14](.\图片\Snipaste_2022-12-20_19-45-14.png)

- 创建硬链接:
  - window:  `mklink /H [新建文件命] [要复制的文件]`
  - macos: `ln [要复制的文件] [新建文件命]`
- 创建软连接:
  - 无法在新建的文件中修改
  - window: `mklink [新建文件命] [要复制的文件名]`
  - macos: `ln -S [要复制的文件命] [新建文件命]`



### 2. pnpm的作用

- 如果你对同一依赖包使用相同的版本, 那么磁盘上只有这个依赖包的一份文件, 而不是100份

- 版本不同的依赖包会被分别存储

- 所有的文件保存在硬盘的统一位置, 使用了硬连接

  

### 3. 使用pnpm

- 非扁平化, 包与包的依赖并不是全在顶层, 在node_modules顶层中生成的文件夹为符号连接, 真实的文件在顶层pnpm文件夹下

- 下载`npm i pnpm -g`
- 添加: `pnpm add axios`

![Snipaste_2022-12-20_21-13-52](.\图片\Snipaste_2022-12-20_21-13-52.png)

| npm 命令                     | pnpm 命令                  |
| ---------------------------- | -------------------------- |
| npm install                  | pnpm install               |
| npm install [package name]   | pnpm add [package name]    |
| npm uninstall [package name] | pnpm remove [package name] |
| npm run [script]             | pnpm [script]              |

- **pnpm install  --shamefully-hoist(创建一个扁平的node_modules目录结构, 类似npm和yarn)**

  - 也可以在根目录下配置`.npmrc`文件

  ```
  shamefully-hoist=true
  strict-peer-dependencies=false
  ```

  

- pnpm-store 硬链接的总仓库 , 通过`pnpm store path`获取store文件路径



## 七. Node版本工具

- 同一台电脑上存在多个版本的node
- 可以使用n / nvm 来管理版本, 不支持windows(使用nvm-windows)
- nvm
  - nvm install x.x.x
  - nvm use ...
  - `nvm list` 列出当前电脑上的所有的版本
  - `nvm list available`: 列出当前node可下载的版本



- n(不在windows上使用)

  - 可以直接用npm安装 `npm install -g n`
  - `n install lts` 安装最新的lts版本
  - `n install latest` 安装最新的版本
  - `n`查看所有的版本

  - **如果在服务器中下载的不是想要的最新的lts版本的node, 可以使用n**

  ```shell
  # 1. 安装
  npm install n -g
  # 通过n来安装最新的LTS和Current版本
  n install lts
  n install latest
  
  # 通过n来切换版本
  n
  ```

  

