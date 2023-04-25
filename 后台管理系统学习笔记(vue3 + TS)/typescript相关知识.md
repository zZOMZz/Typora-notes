[TOC]

## Typescript

### 1.tsconfig.json配置

[tsconfig.json简单介绍](https://www.typescriptlang.org/zh/docs/handbook/tsconfig-json.html)

[tsconfig.json详细参考文档](https://www.typescriptlang.org/zh/tsconfig)

```ts
{
   // 编译选项
  "compilerOptions": {
     // 目标代码（ts => js(esnext指es6之后的版本)）
    "target": "esnext",
     // 目标代码使用什么模块化
     // 如：commonjs: require/module.exports 或 es module: import/export
    "module": "esnext",
     // 启用严格检查
    "strict": true,
     // 对jsx进行怎么样的处理， 这里不转换preserve保留
    "jsx": "preserve",
     // 按照node方式解析模块， import，默认找index...一种解析方式
    "moduleResolution": "node",
     // 跳过对一些库的类型检测
    "skipLibCheck": true,
   	 // 是否允许export default/ module.export={}混合使用
     // 即es module和commonjs
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "useDefineForClassFields": true,
    "sourceMap": true,
    "baseUrl": ".",
    "types": ["webpack-env"],
     // 路径解析, 方便文件的引用
    "paths": {
      "@/*": ["src/*"],
      "components/*": ["src/components/*"]
    },
    // 可以指定项目中能使用哪些库的类型(Proxy/Window/Document)
    "lib": ["esnext", "dom", "dom.iterable", "scripthost"]
  },
  // 当前哪些ts代码需要编译解析
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  // 排除不需要ts代码解析的库,避免node_modules中的文件被include中文件引入而被迫解析
  "exclude": ["node_modules"]
}

```



### 2.shims-vue.d.ts

给项目做一个垫片(shims), 做一些基础声明

```ts
declare module '*.vue' {
  // 拿到定义组件类型
  import type { DefineComponent } from 'vue'
  // 声明实例, 每个*.vue文件返回的都是这个实例
  const component: DefineComponent<{}, {}, any>
  export default component
}
```

