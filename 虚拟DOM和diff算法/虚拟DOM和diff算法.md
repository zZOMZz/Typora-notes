[toc]

# 虚拟DOM

- 由真实DOM变为虚拟DOM是属于**模板编译原理**范畴

```ts
export interface VNode {
  sel: string | undefined;				// 标签名div等
  data: VNodeData | undefined;			// props属性
  children: Array<VNode | string> | undefined;	//子元素,内部不含有其他element时为undefined
  elm: Node | undefined;				// 表示一个真实的DOM element
  text: string | undefined;				// 内部文本
  key: Key | undefined;					// diff算法需要的唯一性key标识
}
```



## 一. 简单介绍虚拟DOM

```html
<!-- 真实DOM -->
<div>
    <h3>标题</h3>
    <ul>
        <li>牛奶</li>
    </ul>
</div>
```

```js
// 虚拟DOM
{
    "sel": "div",
    "data": {
        "class": { "box": true }
    },
    "children": [
        {
            "sel": "h3",
            "data": {},
            "text": "标题"
        },
        {
            "sel": "ul",
            "data": {},
            "children": [
                { "sel": "li", "data": {}, "text": "牛奶" }
            ]
        }
    ]
}
```



## 二. snabbdom(虚拟DOM库)

- snabbdom是著名的虚拟DOM库, 是diff算法的鼻祖, Vue源码借鉴了snabbdom

```shell
npm i snabbdom   #　github上的是使用ts写的, npm是build出来的JavaScript版本
```



### 1. h函数

- 不是用来从真实DOM转换为虚拟DOM的, 那是**模板编译**的范畴

- h函数用来产生**虚拟节点(vnode)**
  - `h(sel,data,text)`

```jsx
import { h } from 'snabbdom/h'

// 简单介绍, 创建虚拟节点
const myVnode = h("a", { props: { href: "http:.." } }, "这张图")

// 得到, myVnode
{
    "sel": "a",
    "data": {
        props: {
            href: "http:..."
        }
    },
    "text": "这张图"
}

// 表示这个DOM节点
<a href="http:..." >这张图</a>
```

- **h函数可以嵌套调用, 得到多层的虚拟DOM树**
- ==是一种嵌套, 而不是递归, 上层的h函数只需要收集下层h函数调用的结果就行==
  - **因为这里直接通过`()`调用了函数, 而不是直接将函数传入**

```js
h('ul',{}, [
    h('li',{},"牛奶"),
    h('li',{},"咖啡")
    h('li',{},"可乐")
])

// 生成
{
    "sel": "ul",
    "data": {},
    "children": [
        { "sel": "li", "data": {}, "text": "牛奶" }
    ]
}
```

