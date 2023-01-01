[toc]

# airbnb项目过程笔记

## 一. svg图片处理

将官网上的svg图片包含标签直接复制到一个jsx文件内, 形成一个组件, 可以更自由的传入参数, 动态改变参数等



## 二. theme

```js
const theme = {
    color: {
        primaryColor: ..
        
    }
}
```



```jsx
import { ThemeProvider } from "styled-components"

(<ThemeProvider theme={theme}>
 	
 </ThemeProvider>)
```



```jsx
export const LeftWrapper = styled.div`
	
	.logo {
		color: ${props => props.theme.color.primaryColor}
	}
`
```



## 三. 调用组件库组件上的方法

- 给定义好的API组件定义一个`ref`
- 拿到ref, 在自己定义的函数内通过DOM元素`如: sliderRef.current.next()`调用方法