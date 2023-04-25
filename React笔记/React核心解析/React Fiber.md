[toc]

# Fiber

[Inside Fiber](https://indepth.dev/posts/1008/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react)

## 一. 介绍Fiber

### 1. 定义和作用

- Fiber可以理解为一种React新的反映协调算法(reconciliation algorithm), 同时它也是一种数据结构
- 它能提供许多有趣的特性, 如进行非阻塞渲染, 实现基于优先级的渲染和在后台进行预渲染
- 将原本的虚拟树遍历从递归(foreach), 无法拆分, 无法暂停的困境中解放了出来, 改为了`while`循环, 将其拆分为了**递增的基本单位**, 使其堆栈始终保持为2层, 而不是随着递归的调用无限增长,  同时将每个节点都可以看为顶节点, 做到任意暂停, 配合`requestIdleCallback`这个API使用, 实现非阻塞渲染等一系列特性
- Fiber节点也会保持组件的状态和DOM, 方便进行对比, 同时避免重复渲染



```txt
// FIber
// 链接节点主要属性
{
    stateNode,
        child,
        return,
        sibling
}

// 更新中的节点主要属性
{
    stateNode: new ClickCounter,   // 类组件中指向实例, React Element
    type: ClickCounter,
    alternate: null,
    key: null,
    updateQueue: null,
    memoizedState: {count: 0},
    pendingProps: {},
    memoizedProps: {},
    tag: 1,
    effectTag: 0,
    nextEffect: null
}
```

### 2. 常用属性

- `stateNode`
  - 保存对实例, DOM节点或者其他React Element关联的Fiber节点, 总的来说这个属性用来将本地状态(local state)与Fiber节点联系起来
  - 函数式组件没有实例, 因此函数式组件构成的节点这个值为null
- `type`
  - 节点的类型, class component是constructor函数的名字
- `tag`
  - 定义Fiber的类型, 用来区分协调算法(reconciliation algorithm)不同的处理
- `updateQueue`
  - 包含state 更新, callbacks和DOM更新的队列
- `memoizedState`
  - 包含当前这次渲染保存的state值
- `memoizedProps`
  - 包含当前这次渲染保存的props值
- `pendingProps`
  - 已经根据新的data更新过的props中需要应用于子组件或DOM元素的props
- `alternate`
  - 保存上一次更新的状态(旧节点的状态), 与`memoziedState`等状态进行比较
- `effectTap`
  - 二进制, 标识出这个Fiber节点需要进行的操作, 如`Update`, `Ref`, `Deletion`, `ContentReset`等





## 二. 两个阶段:

Fiber架构下有两个主要的阶段

- 阶段一: render/reconciliation  可以被打断, 可以是异步的, 因为这个阶段的操作是不可见的
  - 更新state和props
  - 调用lifecycle hooks
  - 检索children
  - 比较新旧children
  - 弄清楚需要更新的DOM

- 阶段二: commit  不能被打断, 只能是同步的, React必须一次性更新完所有的DOM, 因为这个阶段的操作是可见的
- 一些`lifecycle method`会在render阶段被调用, 一些则在commit阶段被调用



## 三. 旧架构存在的问题

- 旧协调器, 也可以称为堆栈协调器(stack reconciler)
  - 如果有大量的渲染工作需要**一次性**去做, 即同步的遍历整个tree,等待组件执行它们的逻辑,  那么可能就会导致动画丢帧
  - 它的优点是非常直观, 但它的局限性是我们无法将整个工作**拆分为递增的基本单位**, 因此我们就无法在特定的组件上将其暂停, 并在之后再重启它, 递归(recourse)必须一次性调用完, 一直迭代, 直到堆栈为空


<img src="./assets/image-20230322155357358.png" alt="image-20230322155357358" style="zoom: 33%;" />

```js
walk(a1);

function walk(instance) {
    doWork(instance);
    const children = instance.render();
    children.forEach(walk);
}

function doWork(o) {
    console.log(o.name);
}

// 结果 a1, b1, b2, c1, d1, d2, b3, c2
// 堆栈在运行时会一直增长, 不会返回
```

<img src="./assets/image-20230325211332951.png" alt="image-20230325211332951" style="zoom:50%;" />



## 四. requestIdleCallback 

`requestIdleCallback`: 

- 这个global API(window), 可以将所有的函数传入队列, 并在浏览器闲置下来的时期再依次调用队列中的函数

- 由于这个API具有一定的限制, react团队基于这个global API实现了自己的`requestIdleCallback`,  对于不支持的浏览器, react会polyfill它
- main thread(主线程)在收到请求后会安排一些时间, 然后空闲时开始构建`work-in-progress tree `
- 如果在一次分配的时间内没有处理完, 就会再次调用`requestIdleCallback`(请求空闲回调), 等待主线程再次返回分配时间

```jsx
requestIdleCallback((deadline)=>{
    console.log(deadline.timeRemaining(), deadline.didTimeout)
});
```

```jsx
// 改写之后
requestIdleCallback((deadline) => {
    // while we have time, perform work for a part of the components tree
    while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
        nextComponent = performWork(nextComponent);
    }
});
```

### 1. 使用这个API的条件

- 通过分配时间, 你不能再像之前的算法一样同步的处理整个tree, 所以为了使用这个API, **你需要将整个渲染过程拆分为可递增的基本单位**
- walking the tree **from the synchronous recursive (递归) model that relied on the <u>built-in stack</u> to an asynchronous model with <u>linked list and pointers</u>**
- 如果你依赖的是`[build-in] call stack`, 它会一直运行直到这个stack为空. 使用Fiber的目的是我们可以**人为的**中断并操纵这个stack. Fiber是堆栈stack的重新实现, 专门用于React组件, 可以将Fiber看为单独的**虚拟栈帧**

- **可以用堆栈的角度看这个问题**
  - 每次的递归(recourse)可以看为同步的添加一个栈帧



![image-20230322164349460](./assets/image-20230322164349460.png)

![image-20230322164423931](./assets/image-20230322164423931.png)



### 2. cooperative scheduling(协调工作)

- 通过将大的工作分解为更小的工作单元, 因此可以暂停完成需要的其他任务, 即可以调用`requestIdleCallback`, 每次一个小单元执行完成, 就会返回观察是否有优先级更高的任务(while循环执行完了一次loop)

![image-20230322173926854](./assets/image-20230322173926854.png)





## 五. Link list traversal(链表遍历)

- 为了实现能够在运行过程中暂停并运行别的任务, 然后在返回回来继续, 我们需要一种数据结构, 包含下面三个字段
  - `child`: 指向当前节点的第一个子节点
  - `sibling`: 指向当前节点的第一个同级节点
  - `return`: 指向父节点
- 因此包含这三个字段的节点就叫为**Fiber**

```js
class Node {
    constructor(instance) {
        this.instance = instance;
        this.child = null;
        this.sibling = null;
        this.return = null;
    }
}
```



<img src="./assets/image-20230325203403256.png" alt="image-20230325203403256" style="zoom: 67%;" />

### 1. 链接

- `link`链接函数在父节点和所有子节点间建立联系, 父节点指向第一个子节点, 子节点的返回指针都指向父节点, 并且子节点间也建立sibling指针

```js
function link(parent, elements) {
    if (elements === null) elements = [];

    parent.child = elements.reduceRight((previous, current) => {
        const node = new Node(current);
        node.return = parent;
        node.sibling = previous;
        return node;
    }, null);

    return parent.child;
}
```

### 2. 运行

- 工作函数

```js
function doWork(node) {
    console.log(node.instance.name);
    const children = node.instance.render();
    return link(node, children);
}
```

### 3. 代码介绍

- 主要代码

```js
function walk(o) {
    let root = o;
    let current = o;

    while (true) {
        // perform work for a node, retrieve & link the children
        // 1. 进行工作(对比)
        let child = doWork(current);

        // if there's a child, set it as the current active node
        // 2. 如果有子节点, 继续向下遍历
        if (child) {
            current = child;
            continue;
        }

        // if we've returned to the top, exit the function
        // 3. 返回到了顶点, 退出遍历
        if (current === root) {
            return;
        }

        // keep going up until we find the sibling
        // 4. 上面条件都不满足时, 且当前节点没有兄弟节点, 则向上返回到父节点
        while (!current.sibling) {

            // if we've returned to the top, exit the function
            // 4.1 一路返回到了顶点则退出
            if (!current.return || current.return === root) {
                return;
            }

            // set the parent as the current active node
            // 4.2 返回到上一层的父节点
            current = current.return;
        }

        // if found, set the sibling as the current active node
        // 5. 没有子节点, 但有兄弟节点的情况下, 将当前节点设为兄弟节点
        current = current.sibling;
    }
}
```

- 在这种算法下, 堆栈不会在我们沿着树往下运行时无限增长, 这样我们就能有效的用我们对堆栈的实现替换浏览器对堆栈的实现

### 4. 与旧遍历算法的对比

- 例如在上面的例子中, 因为`while`每次都直接完成了函数调用,  **所以整个系统堆栈只有两层**;  而旧的算法, 是通过`foreach`来进行遍历的, 所以随着遍历次数增加, 堆栈会越来越大
- 在旧算法中, 我们从顶点开始寻找`children`调用然后`foreach`, 最后一层一层往下调用, 只能一路向下, 无法停止. 
- 但在新算法中, 现在我们通过对每个节点的引用的控制, 使其表现的像顶点, 因此就实现了将大任务拆分为单独的**递增工作单元**, 通过这样我们就能在任何时刻停止这次的遍历, 然后再之后再重启它, 所以就满足了使用`requestIdleCallback`这个API的条件
- 主要区别可以看为`while`循环和`foreach`迭代之间的区别



## 六. work loop

![image-20230322164710993](./assets/image-20230322164710993.png)

### 1. 介绍

```jsx
// if...else区分出两种渲染模式
// shouldYield的值取决于requestIdleCallback中的deadline的值, 它是会一直改变的
function workLoop(isYieldy) {
    if (!isYieldy) {
        // Flush work without yielding
        while (nextUnitOfWork !== null) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    } else {
        // Flush asynchronous work until the deadline runs out of time.
        while (nextUnitOfWork !== null && !shouldYield()) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    }
}
```

`work loop`: 

- 给予Fiber在主线程允许的反应时间中消失, 允许React向下做一些计算(优先级更高), 然后返回让主线程去做别的需要做的事, 为了实现这个, 需要跟踪两件事

  1. 下一个需要分配时间的工作
     1. 当前任务结束会`return`下一个需要工作的任务
     2. 如果当前节点没有child或者sibling, 则会调用complete告诉它的父节点

  2. 主线程分配给当前任务的剩余时间

- 如果当前任务在分配的时间中没有完成任务, 则会再调用`requestIdleCallback`等待主线程再次分配时间



### 2. 作用

- 这种算法能异步的遍历组件树, 并为每个节点(node)都调用这个工作(nextUnitOfWork), 这种并发渲染模式可以用于一些交互式更新, 如: 点击, 输入等



### 3. 主要函数

#### 3.1 render阶段

- [performUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1056)
- [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489)
- [completeUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L879)
- [completeWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532)
- begin视为"步入一个组件", complete视为"走出"一个组件
- 只有子节点和所有分支都完成工作, 父节点才能完成工作和回溯
- 主要的工作集中在`beginWork`和`completeWork`中; `completeUnitWork`主要负责迭代工作

![tmp2](./assets/tmp2.gif)

```jsx
// recieve node from workInProgress tree
function performUnitOfWork(workInProgress) {
    // begin to work
    let next = beginWork(workInProgress);
    if (next === null) {
        next = completeUnitOfWork(workInProgress);
    }
    return next;
}

// point to the next children node to process
function beginWork(workInProgress) {
    console.log('work performed for ' + workInProgress.name);
    return workInProgress.child;
}
```

```js
// 结束当前单元的工作
function completeUnitOfWork(workInProgress) {
    while (true) {
        let returnFiber = workInProgress.return;
        let siblingFiber = workInProgress.sibling;

        nextUnitOfWork = completeWork(workInProgress);

        if (siblingFiber !== null) {
            // If there is a sibling, return it
            // to perform work for this sibling
            return siblingFiber;
        } else if (returnFiber !== null) {
            // If there's no more work in this returnFiber,
            // continue the loop to complete the parent.
            workInProgress = returnFiber;
            continue;
        } else {
            // We've reached the root.
            return null;
        }
    }
}

function completeWork(workInProgress) {
    console.log('work completed for ' + workInProgress.name);
    return null;
}
```

#### 3.2 commit阶段

-  [completeRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306): 开始函数, 在这个函数中生成DOM, 并调用改变后的生命周期函数





### 4. 工作流程

#### 4.1 shouldComponentUpdate

- 在每个节点都检测是否需要更新, 如果state和props没有改变, 则保留原来的DOM(将当前节点指向原本tree上的节点)

<img src="./assets/image-20230322165746913.png" alt="image-20230322165746913" style="zoom: 50%;" />

#### 4.2 effect list

- 如果节点需要更新, 则将其打上标签传入到一个`effect list`队列中, 这个线性列表能进行快速迭代, 且不用在没有副作用的节点浪费时间

  1. `effect list`的作用就是指明哪些节点需要被插入, 更新或者删除, 或者哪些`lifecycle hook`需要被调用, 它是在`commit`阶段被迭代的节点的集合
  2. 注意, 如果使用了`refs`, 即使没有发生变化也会被添加到`effect list`中, `commit`阶段处理时会先将`refs`从这分离出去, 然后在之后再重新附加上
  3. `effect list`中的节点是通过`nextEffect`这个属性连接起来, 而不是`child, sibling`

  ```json
  {
      stateNode: new ClickCounter,
      type: ClickCounter,
      alternate: null,
      key: null,
      updateQueue: null,
      memoizedState: {count: 0},
      pendingProps: {},
      memoizedProps: {},
      tag: 1,
      effectTag: 0,
      nextEffect: null
  }
  ```

  

  ![image-20230326173854894](./assets/image-20230326173854894.png)

<img src="./assets/image-20230322170541458.png" alt="image-20230322170541458" style="zoom:50%;" />



3. 如果当前节点没有子节点, 且不含兄弟节点, 则根据`return`指针向上返回

### 4. 优点

- 这样会让React更具响应性, 能对浏览器事件作出更快的响应, 有助于实现非阻塞渲染





## 七. commit阶段

<img src="./assets/image-20230322170749678.png" alt="image-20230322170749678" style="zoom: 33%;" />



-  开始根据之前构建的`effect list`进行提交更新, 并调用所有的生命周期钩子

   -  Calls the `getSnapshotBeforeUpdate` lifecycle method on nodes tagged with the `Snapshot` effect
   -  Calls the `componentWillUnmount` lifecycle method on nodes tagged with the `Deletion` effect
   -  Performs all the DOM insertions, updates and deletions
      -  当`workInProgress tree`构建完成, 就转换指针`current`指向`workInProgress tree`
   -  Sets the `finishedWork` tree as current
   -  Calls `componentDidMount` lifecycle method on nodes tagged with the `Placement` effect
   -  Calls `componentDidUpdate` lifecycle method on nodes tagged with the `Update` effect

   ```js
   // 大概就是这种顺序
   function commitRoot(root, finishedWork) {
       commitBeforeMutationLifecycles()
       commitAllHostEffects();
       root.current = finishedWork;
       commitAllLifeCycles();
   }
   // 上面的所有函数都会遍历effect list, 检查effect的类型, 执行那些在自己范围内的作用
   ```

### 1. commitBeforeMutationLifecycle

-  检查effect tree中的节点是否含有`Snapshot`effect

   ```js
   function commitBeforeMutationLifecycles() {
       while (nextEffect !== null) {
           const effectTag = nextEffect.effectTag;
           if (effectTag & Snapshot) {
               const current = nextEffect.alternate;
               commitBeforeMutationLifeCycles(current, nextEffect);
           }
           nextEffect = nextEffect.nextEffect;
       }
   }
   // 对于类组件相当于调用getSnapshotBeforeUpdate method
   ```

   

### 2. commitAllHostEffect

- 这个function执行dom update

```js
function commitAllHostEffects() {
    switch (primaryEffectTag) {
        case Placement: {
            commitPlacement(nextEffect);
            ...
        }
        case PlacementAndUpdate: {
            commitPlacement(nextEffect);
            commitWork(current, nextEffect);
            ...
        }
        case Update: {
            commitWork(current, nextEffect);
            ...
        }
        case Deletion: {
            commitDeletion(nextEffect);
            ...
        }
    }
}
```

- It’s interesting that React calls the `componentWillUnmount` method as part of the deletion process in the `commitDeletion` function



### 3. commitAllLifecycle

- 调用剩余的所有生命周期`lifecycle function`, 如`compoenntDidUpdate`和`componentDidMount`





## 八. lifecycle

<img src="./assets/image-20230322173508643.png" alt="image-20230322173508643" style="zoom: 50%;" />

- 因为render阶段是可以异步执行的, 因此有些`lifecycle method`会对这个过程造成影响, 而被弃用
- 以下是常见在同步运行的commit阶段可以调用的lifecycle method
  - `getSnapshotBeforeUpdate`
  - `componentDidMount`
  - `componentDidUpdate`
  - `componentWillUnmount`

- 在一些情况下被终止重启时会重新开始, 因此也会重新调用`lifecycle Hook`, 有些情况下也会从停止的地方开始

<img src="./assets/image-20230322173542606.png" alt="image-20230322173542606" style="zoom: 33%;" />



## 九. side-effect

- `data fetching, subscriptions(订阅), manually changing the DOM(人为的改变DOM)`以上的这种操作都被称为`side-effect`(副作用), 因为上面的这些操作能影响到别的组件, 不符合纯函数的定义, 因此不能让他们发生在rendering期间
- class component一般将其放到各种`lifecycle Hook`中, function component一般将其放到`useEffect`component中
- Fiber node是一种非常方便的机制去追踪`effect`, 每个FIber节点都能与`effects`产生联系, Fiber节点中包含一个称为`effectTag`的字段
- Fiber中基本定义了在更新实例后需要做的`effect`操作, 在更新完成后便会执行这些操作,  如ref, 具体见[这里](https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js)





## 十. priority优先级

- 在fiber reconciler中, 每个update都得被分配一个优先级

  - `Synchronous`: 同步, 就像堆栈协调器一样工作


  - `Task`: 需要在`event loop`的下一个`tick`(nextTick)前被处理


  - `Animation`: 利用`requestAnimationFrame`来进行安排, 所以能在下一帧看到它


- 下面三个使用`requestIdleCallback`

  - `High`: 需要尽快反应, 适合反应灵敏的低优先级工作

  - `Low`

  - `Offscreen`

<img src="./assets/image-20230322172535810.png" alt="image-20230322172535810" style="zoom: 33%;" />



![image-20230322173217292](./assets/image-20230322173217292.png)







## 十一. FIber的优点

### 1. 流式加载渲染

在之间的架构中, React为了渲染出完整的树, 必须等待完整的文件, 然后加载它们, 再开始构建完整的tree

Fiber可以做到`streaming rendering`, 不需要等待完整的文件, 因为协作调度的存在

![image-20230322174705831](./assets/image-20230322174705831.png)

### 2. 并行处理工作

- 将完整的tree根据sibling分成多个分支, 每个分支并行处理, 

![image-20230322174835937](./assets/image-20230322174835937.png)





- 双缓冲技术(待查阅)
