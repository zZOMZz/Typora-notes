# React笔记的一些总结

## 一. this绑定问题

### 1. 类组件

- 在`jsx语法`这一篇笔记中



## 二. 开发时所需的依赖及其作用

```shell
npm install React  # 核心代码
npm install React-dom  # [待补充]
npm install babel # 负责解析jsx
npm install create-react-app  # react脚手架
npm install react-router-dom   # react-router包含react-native的内容, 对于web开发来说是多余的
npm install react-transition-group  # 制作tansition过渡动画
npm install styled-components   # css in js
npm install classnames  # 利用boolean值判断是否添加某一个属性
npm install @reduxjs/toolkit # 一种简便的利用redux的方式
npm install react-redux  # 提供Provider, connect, useDispatch, useSelector等函数的将redux和react联系起来包
```



## 三. 提升性能的方法

### 1. setState设置为异步

- setState设计为异步, 可以显著的提升性能
  - 如果每次调用setState都进行一次更新, 那么render函数会被频繁调用, 界面重新渲染, 造成效率低下
  - 最好的方法是获取到多个更新后, 将多次更新放入到队列里, 获取时进行批量处理, 这样render函数只用执行一次, 避免大量回流和重绘
- 如果同步更新了state, 但是还没有执行render函数, 那么state和props不能保持同步
  - state和props不保持一致性, 同步改变了state但是不能马上执行render函数, 会在开发中产生很多问题
  - 设计成异步, 可以使state一改变就能马上执行render函数



### 2. 优化render函数

- 类组件: `PureComponennt`
- 函数组件: `memo(() => {})`



- 在父组件重新调用render函数重新渲染时, 子组件即使未发生改变也会被迫重新渲染
- 即使state的前后状态相同, 但因为调用了setState, 还是会进行重新render

解决办法:

- 使用`shouldComponentUpdate(nextProps, newSate)`生命周期钩子(SCU)
  - nextProps: 新的属性, 子组件可用
  - newState: 新的state参数, 可以用来比较



### 3. useCallback

- 定义一个传入子组件的函数时, 可以在组件重新渲染时, 返回的是一个相同对象(地址)的函数, 避免改变传入子组件的props从而引起不必要的渲染



### 4. useMemo

- 当依赖不变的情况下, 返回的是同一个值
- 定义的是一个传入子组件的对象时, 可以避免由于重复定义而传入的是不同的对象造成不必要的渲染



### 5. useSelector(shallowEqual)

- `useSelector`监听了整个state, 当state发生改变时, 即使变换的不是组件的依赖数据, 组件也会发生渲染
- 需要从`react-redux`中导入`shallowEqual`函数, 并将其传入到`useSelector`的第二个参数

```jsx
const { count } = useSelector(() => {}, shallowEqual)
```



### 6. useEffect第二个参数

- 定义useEffect只有在哪些参数改变后, 才重新运行, 如果传入的是一个空数组, 即不受任何参数的影响, 只会在初始渲染的时候执行
