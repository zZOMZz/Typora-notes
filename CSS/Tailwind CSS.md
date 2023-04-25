[toc]

# Tailwind CSS

- tailwind可以理解为是postcss的一个插件
- tailwind只会按需加载引入的类, 不会全部添加进来

## 一. postcss其余插件

- `postcss-import`
  - `@import`导入CSS文件, 用于处理PostCSS的规范插件
  - 只能在文件顶部使用`@import`

```js
// postcss.config.js
module.exports = {
  plugins: [
    require('postcss-import'),
    require('tailwindcss/nesting'),
    require('tailwindcss'),
    require('autoprefixer'),
  ]
}
```

