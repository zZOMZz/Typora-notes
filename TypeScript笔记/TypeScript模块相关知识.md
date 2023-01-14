[toc]

# TypeScript模块相关知识

## 一. 模块化相关知识

- JavaScript的规范声明中表示**没有export的JavaScript文件都应该被认为是一个脚本, 而非一个模块, 在脚本中定义的变量会被声明在共享的全局作用域**
- 如果需要将一个文件作为模块进行处理可以写`export {}`
- TypeScript与Javascript有相同的特性

```tsx
import type { ... } from ...
import { type IPerson } from ...
```



## 二. 命名空间namespace

- 在ES Module标准出现之前, 称之为内部模块
- **更推荐ES Module**

```tsx
export namespace price {
    // 里面的函数也需要导出
    export function format() {
        // 
    }
}

// 不会出现冲突
export namespace date {
    function format() {
        //
    }
}


price.format()
```



## 三. 类型查找规则

查找类型声明的方式

1. 内置类型声明(ts自带的类型)
2. 外部定义类型声明(第三方库)
3. 自己定义类型声明

### 1. 类型声明文件(.d.ts)

- 这个文件中定义的类型, 默认是全局的
- 其中不写逻辑代码



### 2. 外部定义类型声明

- 外部类型声明通常是我们使用一些库(第三方库)时, 需要进行的一些类型声明

- 两种方式:

  - **在自己库中进行了类型声明(编写了.d.ts文件), 如axios**
  - 也有一些库没有做类型声明(react), **通过社区的一个公有库DefinitelyType存放类型声明文件**[DefinitelyTyped ](https://github.com/DefinitelyTyped/DefinitelyTyped)

  ```shell
  npm install @types/react -D
  ```

  

### 3. 自己声明类型文件

```tsx
// [name].d.ts
// 这个文件里只写声明, 不写实现
// 1. 自己编写类型文件
declare module "loadsh" {
    // 1.1 模块中的类和函数需要export导出
    export function join(...args: any[]): any[]
}

// 2. 定义全局类型
// 这里面定义的类型是全局的, 一般不这么用
type IDtype = string | number

// 3. 定义全局变量
// 为一些变量定义类型声明
// 使用一些不在当前ts文件中定义的变量时(全局变量), ts不会识别, 需要自己编写声明
declare const name: string

// 4.声明文件模块
declare module "*.png"
declare module "*.jpg"
    
// 5. 通过CDN引入一些包时: 
declare namespace $ {
    export function ajax(url: string)
}
// 使用需要 $.ajax()这样就不会报错
```

