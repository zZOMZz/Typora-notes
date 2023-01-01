[toc]

# React的过渡动画

[React Transition Group](http://reactcommunity.org/react-transition-group/)

## 一. react-transition-group介绍

社区推出的第三方库

给一个组件的显示和消失添加某种过渡动画, 增加用户体验

```shell
npm install react-transition-group 
```



## 二. CSSTransition

- CSSTransition是基于Transition组件构建的, 主要用于==**进入和退出**==动画
- CSSTransition执行过程中, 有三个状态: appear, enter, exit
- 三个状态, 需要定义对应的CSS样式
  - 开始状态: `-appear, -enter, -exit`
  - 执行动画: `-appear-active, -enter-active, -exit-active`
  - 执行结束: `-appear-done, -enter-done, -exit-done`



```jsx
render() {
    return (
        // in判断进入或者隐藏
        // unmountExit当in为false时, 将元素卸载
    	<CSSTransition in={isShow} className="zzt" timeout={2000} unmountExit={true}>
        	<h1>HHHHH</h1>
        </CSSTransition>
    )
}
```

```css
// 进入动画, 由in属性判断
.zzt-enter {
    opacity: 0;
}

.zzt-enter-active {
    opacity: 1;
    transition: opacity 2s ease;
}

// 离开动画
.zzt-exit {
    opacity: 1;
}

.zzt-exit-active {
    opacity: 0;
    transition: opacity 2s ease;
}
```

- in: 触发进入(enter)和退出(exit)状态
  - 如果添加了`unmountOnExit={true}`, 那么组件会在执行推出动画结束后被从DOM上移除
- timeout: 过渡动画的设置事件, 必须设置
- appera: 第一次刷新时,是否添加动画

```css
.zzt-appear {
    opacity: 0
}
.zzt-appear-active {
    opacity: 1;
    transition: 2s opacity ease;
}
```

钩子函数:

- onEnter: 开始执行进入动画
- onEnering: 正在执行进入动画
- onEntered: 执行进入结束
- ...(appear和exit与enter相似)



- nodeRef: 直接拿到内部整体组件的ref进行调用, 避免使用一些过老的API



## 三. SwitchTransition

- SwitchTransition可以完成两个组件之间==**切换**==的酷炫动画
  - 比如先退出再进入, 先隐藏再显露

```jsx
// 先进入再离开
<SwitchTransition mode="out-in">
    // 进入和退出动画再设置到CSSTransition里
    <CSSTransition key={isLogin ? "login" : "exit"} classNames="login" timeout={1000}>
        <h2>HHHHH</h2>
    </CSSTransition>
</SwitchTransition>
```



## 四. TransitionGroup

- 在给**列表**数据添加动画时用
  - 增删数据时, 不能报key设置为index, 得保证key是唯一的

```jsx
 <TransitionGroup className="todo-list">
          {items.map(({ id, text, nodeRef }) => (
            <CSSTransition
              key={id}
              nodeRef={nodeRef}
              timeout={500}
              classNames="item"
            >
              <ListGroup.Item ref={nodeRef}>
                <Button
                  className="remove-btn"
                  variant="danger"
                  size="sm"
                  onClick={() =>
                    setItems((items) =>
                      items.filter((item) => item.id !== id)
                    )
                  }
                >
                  &times;
                </Button>
                {text}
              </ListGroup.Item>
            </CSSTransition>
          ))}
</TransitionGroup>
```

