# React setState其中的细节

## 一. 安排更新

- 每一个React组件都有个与之相关的`updater`, 充当组件和React核心的桥梁, 同时也允许`setState`在ReactDOM, React Native, server side rendering中实现方式不同
- 对于一个类组件, 它的`updater`是`classComponentUpdater`, 他负责检索Fiber, 将更新排队, 安排工作

```jsx
// 一次setState后的Fiber结构
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    updateQueue: {
         baseState: {count: 0}
         firstUpdate: {
             next: {
                 // 在render阶段需要进行处理的函数
                 payload: (state) => { return {count: state.count + 1} }  
             }
         },
         ...
     },
     ...
}
```



## 二. beginWork function

- React会先调用"预突变"生命周期方法`pre-mutation lifecycle methods`, 来先更新state和定义标记好相关的副作用`side-effects`

- `beginWork`function 通过Fiber 节点的`type`确定如何对其进行处理, 并返回这次渲染形成更新后的FIber节点

```js
function beginWork(current$$1, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        ...
        case FunctionalComponent: {...}
        case ClassComponent:
        {
            ...
            return updateClassComponent(current$$1, workInProgress, ...);
        }
        case HostComponent: {...}
        case ...
}
```

### 1. updateClassComponent

```js
// 返回新的Fiber节点
function updateClassComponent(current, workInProgress, Component, ...) {
    ...
    // 如果是第一次创建,则stateNode为null; 通过stateNode拿到当前节点的实例instance(类组件)
    const instance = workInProgress.stateNode;
    let shouldUpdate;
    if (instance === null) {
        ...
        // In the initial pass we might need to construct the instance.
        // 创建实例
        constructClassInstance(workInProgress, Component, ...);
		// 挂载实例, 触发更新
        mountClassInstance(workInProgress, Component, ...);
        shouldUpdate = true;
    } else if (current === null) {
        // In a resume, we'll already have an instance we can reuse.
        // 有可以重用的实例
        shouldUpdate = resumeMountClassInstance(workInProgress, Component, ...);
    } else {
        // 更新当前节点的状态, 具体见下
        shouldUpdate = updateClassInstance(current, workInProgress, ...);
    }
    return finishClassComponent(current, workInProgress, Component, shouldUpdate, ...);
}
```

### 2. updateClassInstance

- 为更新后的Fiber节点(Fiber数据结构中的属性修改完成)进行更新处理
  - 处理`updateQueue`中的updates, 更新实例中的state和props
  - 同时调用一些这个阶段的`lifecycle method`
- 因此改变state和prop后的节点就形成了

```js
function updateClassInstance(current, workInProgress, ctor, newProps, ...) {
    const instance = workInProgress.stateNode;

    const oldProps = workInProgress.memoizedProps;
    instance.props = oldProps;
    if (oldProps !== newProps) {
        callComponentWillReceiveProps(workInProgress, instance, newProps, ...);
    }

    let updateQueue = workInProgress.updateQueue;
    if (updateQueue !== null) {
        processUpdateQueue(workInProgress, updateQueue, ...);
        newState = workInProgress.memoizedState;
    }

    applyDerivedStateFromProps(workInProgress, ...);
    newState = workInProgress.memoizedState;

    const shouldUpdate = checkShouldComponentUpdate(workInProgress, ctor, ...);
    if (shouldUpdate) {
        instance.componentWillUpdate(newProps, newState, nextContext);
        workInProgress.effectTag |= Update;
        workInProgress.effectTag |= Snapshot;
    }

    instance.props = newProps;
    instance.state = newState;

    return shouldUpdate;
}
```



### 3. finishClassComponent

- 调用组件实例(instance)上的render方法, 通过`alternote`属性中保存的instance



## 三. 协调子项

- 通过`finishClassComponent`返回`React element`与当前的Fiber进行比较

```
// Fiber before
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: {children: 0},
    pendingProps: {children: 0},
    ...
}
```

```
// React element
{
    $$typeof: Symbol(react.element)
    key: "2"
    props: {children: 1}
    ref: null
    type: "span"
}
```

```
// Fiber after
{
    stateNode: new HTMLSpanElement,
    type: "span",
    key: "2",
    memoizedProps: {children: 0},
    pendingProps: {children: 1},
    ...
}
```





## 四. 之后的工作

- 至此更新后的Fiber节点彻底形成, 这个节点`{type: ..., effectTag: ...}`会被传入`effect list`中, 在`work loop`全部工作完成后, 形成`workInProgress tree`再`commit`, 调用一些生命周期, 遍历完`effect list` , 改变`current tree`这个指针的指向
- 详细见`React Fiber`这篇笔记
