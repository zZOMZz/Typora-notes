[toc]

# 自定义Hook

- 自定义Hook只是一种函数逻辑代码的抽取, 避免代码重复, 提高重用性

- 一定要以`use`开头
- 可以在在这个函数内调用其他的Hooks



```jsx
function useLogLife() {
    useEffect(() => {
        coonsole.log("zzzz")
        // 别的一些操作
    },[])
}
```

```jsx
import { UserContext } from "../context"

export function useUserToken() {
	const token = useContext(UserCOntext)

	return [token]
}
```

