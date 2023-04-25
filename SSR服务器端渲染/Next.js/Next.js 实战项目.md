[toc]

# Next.js 实战项目

## 一. Layout

```tsx
// components/layout/index.tsx

// 传入这个Layout中的内容会被自动放入children中
const Layout = memo((props) => {
    const { children } = props
    return (
    <div className="layout">
    	<NavBar></NavBar>
        { children }
        <Footer></Footer>
    </div>
    )
})
```



## 二. 样式

- 采用CSS Module的方式来编写样式
- 在页面的当前文件夹下编写如: `index.module.scss`文件, 再通过`import style from "./index.module.scss"`来引入
- 再通过`styles['content-right']`来引入, 也可以通过`style.name `来进行引入, 效果: `className="content-right"`



## 三. 搜索框推荐下拉页面

- 通过监听`input`的事件, 改变是否添加css类, 显示`display`, 默认为none

```tsx
<input
    onFocus={() => handleInput(true)}
    onBlur={() => handleInput(false)}
    >
    
<div className={classNames({show: IsShow})} >
    
</div>
```

