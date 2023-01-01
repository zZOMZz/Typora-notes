[toc]

# Redux拦截中间件

## 一. monkey patch

- 篡改现有的代码, 利用现有的代码对执行逻辑进行修改

```jsx
function log(store) {
    const next = store.dispatch
    
    function logAndDispatch(action) {
        console.log("action",action)
        next()
        console.log("当前的state",store.getState())
    }
    
    store.dispatch = logAndDispatch    
}
```



## 二. redux-thunk

```jsx
function thunk(store) {
    const next = store.dispatch
    function thunkDispatch(action) {
        if( typeof action === "function" ) {
            action(store.dispatch,store.getState)
        } else {
            next(action)
        }
    }
    
    store.dispatch = thunkDispatch
}
```

