[toc]

# diff算法

![diff算法和虚拟DOM](.\图片\diff算法和虚拟DOM.jpg)

- diff算法是发生在新的虚拟DOM和老虚拟DOM上的(精细化比较), **算出如何进行最小量更新, 最后反映到真正的DOM上**

  

diff算法的**三大原则**

- **key属性对一个节点来说非常重要, 可以告诉diff算法, 在更改前后它们是同一个DOM节点**
  - 老节点的`key`要和新节点相同, 且旧节点的选择器要和新节点的相同
- **只是同一个虚拟节点才会进行精细化比较**, 否则就是暴力拆除旧节点, 插入新的
- **只进行同层比较, 不会进行跨层比较**, 即使是同样的虚拟节点, 但是改变了层级, 依旧会暴力拆除, 然后插入新的



## 一. patch函数

- 仅仅生成虚拟DOM是无法被页面显示的, 挂载虚拟节点(上树), 使其能被页面显示



```js
import { init } from 'snabbdom/init'
import { classModule } from 'snabbdom/modules/class'
import { propsModule } from 'snabbdom/modules/props'
import { styleModule } from 'snabbdom/modules/style'
import { eventListenerModule } from 'snabbdom/modules/eventlisteners'

const patch = init([classModule,propsModule,styleModule, eventListenerModule])

// 让虚拟节点上树
const container = document.getElementById('container')
// container会被删除, 而不是作为容器
patch(container, myVnode)
```



### 1. createElm(转换虚拟节点为真实DOM)

- 嵌套

```js
// patch函数中的createElement函数(负责将虚拟节点转换为真实的dom)
export default function createElm(vnode) {
    let domNode = document.createElement(vnode.sel)
    // 判断vnode内部是text还是children
    if(vnode.text !== '' && (vnode.children === undefined || vnode.children.length === 0)) {
        // text情况
        domNode.innerText = vnode.text
    } else if(Array.isArray(vnode.children) && vnode.children.length > 0) {
        // 内部是子节点
        for(let i=0; i< vnode.children.length; i++) {
            const el = createElm(vnode.children[i])
            domNode.appendChild(el)
        }
    }
    // 外面的vnode也会改变, 传入的对象是引用赋值
	vnode.elm = domNode   
    return vnode.elm
}
```



- 旧节点和新节点不同
  - `oldVNode.elm.parentNode.insertBefore(newVnode)`先将元素插入到旧节点之前, 再删除旧节点

- **DOM插入的两种方法:**
  - `insertBefore`
  - `appendChild`



### 2.  进行最小化更新

- **先判断key是否相同**, 若key都不相同, 直接**暴力拆除**

![Snipaste_2023-02-02_11-25-19](.\图片\Snipaste_2023-02-02_11-25-19.png)

```js
// patch函数
export default patch(oldVnode,newVnode) {
    // 1. 判断oldVnode是否是Vnode, 如果不是则转换真实DOM为虚拟dom的格式
    
    // 2. 判断oldVnode和newVnode是否相同(key,sel)
    if(key, sel相同) {
        // 待解决
    } else {
        // 暴力拆除
        // 1. 创建新的真实DOM(通过newVnode)
        let newVnode = createElement(newVnode)
        if(oldVnode.elm.parentNode && newVnodeElm) {
            oldVnode.elm.parentNode.insertBefore(newVnode,oldVnode.elm)
        }
    }
}
```

### 1. 判断newVnode是否有text属性

```js
if(newVnode.text !=== undefine && newVnode.children.length === 0) {
    // 1. newVnode含有text属性, 即没有children
    if(newVnode.text !== oldVnode.text) {
		// newVnode的text直接覆盖原DOM元素(oldVnode.elm)的innerText
    	oldVnode.elm.innerText = newVnode.text
    }
} else {
    // 2. newVnode没有text属性, 则有children属性
    
    // 3. 判断oldVnode是否有children属性
}
```

### 2. 判断oldVnode是否有children属性

```js
if(oldVnode.children !== undefined && oldVnode.children.length > 0) {
    // 最复杂的情况
} else {
    // oldVnode不含children属性, 原本为text
    
    // 1. 先清空, dom元素无法覆盖text, text可以覆盖dom元素
    oldVnode.elm.innerText = ''
    // 2. 再将children挂载到oldVnode.elm上
    for(let i=0; i<newVnode.length; i++) {
        const el = createElm(newVnode.children[i])
        oldVnode.elm.appendChid(el)
    }
}
```

### 3. oldVnode,newVnode都具有children属性(最复杂)

#### 3.1 错误的做法(比较复杂, 需要多种情况杂糅)

- **需要将新节点`newVnode.children[i].elm`插入到所有未处理的节点`oldVnode.children[un].elm`之前, 而不是已处理的节点之后, 否则后面插入的会被之前已经插入的节点影响(oldVnode中的索引会随着之前插入的节点发生变化)**

```js
// 错误的做法, 下面的代码仅仅是处理新增节点
// 所有未处理的节点开头
let un = 0
for(let i = 0; i< newVnode.children.length; i++) {
    let el = newVnode.children[i]
    let isExit = false
    for(let j=0; j < oldVnode.children.length; j++) {
        if(el.key === oldVnode.children[j].key) {
            isExit = true
        }
    }
    if(!isExit) {
        const dom = createElm(el)
        if(un <= oldVnode.children.length){
             oldVnode.elm.insertBefore(dom, oldVnode.children[i].elm)
        } else {
             oldVnode.elm.appendChild(dom)
        }
    } else {
        un++
        // 判断位置是否相同
    }
}
```



#### 3.2 优化算法(重要)

- 源码在`init.ts`里, `updateChildren()`

- 四种命中查找

  - 新前: 新的虚拟节点中所有**没有处理的节点中开头的节点**

  - 新后: 新的虚拟节点中所有**没有处理的节点中最后的节点**

    1. 新前与旧前

    2. 新后与旧后

    3. 新后与旧前: ==当这个命中时, 需要移动**新后(旧前)指向的节点**到**旧后之后**==

    4. 新前与旧后: ==当这个命中时, 需要移动**新前(旧后)指向的节点**到**旧前之前**==

- **一旦命中,就不继续向下查找**, 开始移动指针, **前往下移, 后往上移**

- **如果四种命中查找都无法命中**, 就会使用**循环语句查找新前所指的节点**, 将查找的节点打上标记`undefined`(**没有查找到的话, 就直接插入**), 并将这个节点的真实DOM插入到**旧前之前**
- **如何移动节点:** 如果你使用`insertBefore`或`appendChild`插入的是一个已经存在的DOM树上的节点, 那么它就会被移动

![Snipaste_2023-02-02_14-09-16](.\图片\Snipaste_2023-02-02_14-09-16.png)

- 循环条件`whild(新前<=新后 && 旧前<=旧后)`
  - 如果旧的节点先循环完毕, 说明新节点中有要插入的节点
  - 如果新的节点先循环完毕, 说明旧节点中需要删除