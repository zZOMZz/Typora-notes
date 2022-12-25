[TOC]

## vue相关知识



### 1. vue-router

[API 参考 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/api/index.html#createwebhistory)

#### 1. vue-router基础知识

```js
// 1.需要先从vue-router中导入需要的函数和类型
import { createRouter, createWebHashHistory, RouteRecordRaw } from 'vue-router'
// 2.创建routes数组
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    redirect: '/main'
  },
  {
    path: '/main',
    name: 'main',
    component: () => import('@/views/main/main.vue')
  },
  {
    path: '/:pathMatch(.*)*',
    component: () => import('@/views/not-found/not-found.vue')
  }
]
// name的作用
// <router-link :to="{ name: 'user', params: { userId: '123' }}">User</router-link>
// <router-link :to="'/home'">Home</router-link>

// 3.创建router
const router = createRouter({
  // history是有回溯历史,createWebHashHistory产生的url中会带#, localhost:8080/#/login
  history: createWebHashHistory(process.env.BASE_URL),
  routes
})

// 4.导出router
export default router
```

#### 2. 添加动态路由

```ts
// addRoute的定义
addRoute(parentName: RouteRecordName, route: RouteRecordRaw): () => void;
addRoute(route: RouteRecordRaw): () => void;

// addRoute的使用
routes.forEach((route) => {
    // 这里是添加children
    router.addRoute('main', route)
})

```





### 2. vuex使用

[Vuex](https://vuex.vuejs.org/zh/guide/)

```js
// 拿到子模块的state类型
import { ILoginState } from './login/types'
// 定义根目录的state类型
export interface IRootState {
  name: string
  age: number
  entireDepartments: any[]
  entireRole: any[]
}
// 定义要放入主模块的子模块
// 目的是为了能在模块中直接利用 const userCount = store.state.system.userCount, 不主要
// 不这么做也能使用state.dispatch('...')
interface IRootWithModule {
  login: ILoginState
}
// 将这个对象导出
export type IStoreType = IRootState & IRootWithModule
```



```js
import { createStore, Store, useStore as useVueStore } from 'vuex'
import { IRootState, IStoreType } from './types'
// 主模块
export const store = createStore<IRootState>({
  state: {
    name: 'zzt',
    age: 15,
    entireDepartments: [],
    entireRole: []
  },
  // 子模块定义
  modules: {
    login,
    system
  }
})

//子模块
import { Module } from 'vuex'
import { IRootState } from '@/store/types'
import { ISystemState, pagePayload } from './types'

// ISystemstate是对自身state的定义,IRootState是对根目录的定义
const systemModule: Module<ISystemState, IRootState> = {
  // namespace在调用子模块的actions需要为dispatch['moduleName/actionName']
  namespaced: true,
  state: {
  },
  getters:{
    // 第一个参数为state,一般通过getters拿到state里的数据
    pageListData(state) {
      return (pageName: string) => {
        return (state as any)[`${pageName}List`]
      }
    },
  },
  mutations:{
    // 第一个参数为state
    changeUsersList(state, userList: any[]) {
      state.usersList = userList
    },
  },
  actions:{
    // 在actions里执行复杂操作,可以异步,第一个参数是个对象,可以通过解构直接拿到相应函数
    // commit对应mutation, dispatch对应action
    async getPageListAction({ commit, dispatch, state }, payload: pagePayload) {
    	await ....
    }
  }
}
```

1.actions 第一个参数context:

```js
{
  state,      // 等同于 `store.state`，若在模块中则为局部状态
  rootState,  // 等同于 `store.state`，只存在于模块中
  commit,     // 等同于 `store.commit`
  dispatch,   // 等同于 `store.dispatch`
  getters,    // 等同于 `store.getters`
  rootGetters // 等同于 `store.getters`，只存在于模块中
}
```

2.getters 接受的参数

```js
state,       // 如果在模块中定义则为模块的局部状态
getters,     // 当前模块的局部 getters
rootState,   // 全局 state
rootGetters  // 所有 getters
```



### 3.父组件调用子组件方法

**子组件:**

```vue
<script>
import { defineExpose } from 'vue'
const callback = () => {
    console.log('子组件方法')
}
// 将子组件的方法或者属性暴露出去
defineExpose({
    callback
})
</script>
```

**父组件:**

```vue
<script setup lang='ts'>
import { ref, InstanceType } from 'vue'
import subElement from './subElement.vue'
// 拿到子组件的DOM元素
const subElementRef = ref<InstanceType<typeof subElement>>()
// 这样就能直接调用子组件暴露出来的属性和方法了
subElementRef.value?.callback()
</script>
<template>
	<subElement ref='subElementRef' />
</template>
```

**子组件中的内容是对组件的描述(类比于'类 class', 但又不同于类,它有一个具体的值, 因此需要InstanceType<typeof subComponent> 拿到拥有构造函数的实例), 父组件中利用子组件是根据子组件的对象创建一个真正的组件实例(Instance)**

[InstanceType文档](https://www.typescriptlang.org/docs/handbook/utility-types.html#instancetypetype)

```ts
class C {
    x = 0;
    y = 0;
}

type T0 = InstanceType<typeof C>
type T0 = C
```



### 4. 响应式相关知识

#### 1. 基础理解(不涉及到源码)

1. 当我们们更改了响应式对象后,DOM元素会相应的发生变化,但DOM元素并不是实时发生变化的,它会等到'下一个时机',以便我们确保无论你进行了多少次的状态更改,每个组件都只改变一次,可以使用nextTick()等待一次改变完成后的DOM元素

```js
import { nextTick } from 'vue'
function increment() {
  state.count++
  nextTick(() => {
    // 访问更新后的 DOM
  })
}
```

2. Vue中, 状态都是默认深层响应式的,即在深层的改变也能被检测到

```js
import { reactive } from 'vue'

const obj = reactive({
  nested: { count: 0 },
  arr: ['foo', 'bar']
})

function mutateDeeply() {
  // 以下都会按照期望工作
  obj.nested.count++
  obj.arr.push('baz')
}
```

3. 依靠深层响应性,响应式嵌套对象依旧是代理

```js
const proxy = reactive({})

const raw = {}
proxy.nested = raw
// 这里proxy.nested具有响应性
console.log(proxy.nested === raw) // false
```

4. 一个响应式的ref对象可以被整个替换

```js
const objectRef = ref({ count: 0 })

// 这是响应式的替换
objectRef.value = { count: 1 }
// 对count进行改变依旧能被追踪到
```

5. ref被传递给函数,或者被结构都不会失去响应性

```js
const obj = {
  foo: ref(1),
  bar: ref(2)
}

// 该函数接收一个 ref
// 需要通过 .value 取值
// 但它会保持响应性
callSomeFunction(obj.foo)

// 仍然是响应式的
const { foo, bar } = obj
```

6. 当ref被嵌套在响应式对象reactive中时会被自动解包, 将一个新的ref替换掉旧的ref时,旧的ref的连接就会断开

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0
// 同时保留响应性
state.count = 1
console.log(count.value) // 1

const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
// 原始 ref 现在已经和 state.count 失去联系
console.log(count.value) // 1
```



#### 2. 失去响应性的情况

1. Vue的响应式系统是通过==属性==访问进行追踪的, 因此我们必须保持对该响应式对象的相同引用,不能随意'替换'一个响应式对象, 不然就会导致原响应性丢失

```js
const state = reactive({ count: 0 })

// n 是一个局部变量，同 state.count
// 失去响应性连接
let n = state.count
// 不影响原始的 state
n++

// count 也和 state.count 失去了响应性连接
let { count } = state
// 不会影响原始的 state
count++

// 该函数接收一个普通数字，并且
// 将无法跟踪 state.count 的变化
callSomeFunction(state.count)

```



### 5.v-slot相关知识

#### 1. 具名插槽

**子组件:**

```vue
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

```

**父组件:**

```vue
<BaseLayout>
  <template v-slot:header>
    <!-- header 插槽的内容放这里 -->
  </template>
    <!-- 或者有另一种简便写法 -->
  <template #header>
  </template>
</BaseLayout>

```

**当一个组件同时接收默认插槽和具名插槽时，所有位于顶级的非 `<template>` 节点都被隐式地视为默认插槽的内容。所以上面也可以写成：**

```vue
<BaseLayout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <!-- 隐式的默认插槽 -->
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</BaseLayout>

```



#### 2.动态插槽名

**即插槽名会动态变化, 这个时候这个动态变化的名字要用"[ ]"括起来**

```vue
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>

  <!-- 缩写为 -->
  <template #[dynamicSlotName]>
    ...
  </template>
</base-layout>

```



#### 3. 作用域插槽

**使用作用域插槽的目的是: 令父组件在向子组件插入的时候,==能拿到子组件的状态和数据==**

1.子组件要自己将数据放入到能被父组件访问到的插槽上

```vue
<!-- <MyComponent> 的模板 -->
<div>
  <slot :text="greetingMessage" :count="1"></slot>
</div>
```

2.父组件使用"v-slot="指令, 拿到子组件slot上定义的props

````vue
<MyComponent v-slot="slotProps">
  {{ slotProps.text }} {{ slotProps.count }}
</MyComponent>
````

3.类似于下面的函数(伪函数)

```js
MyComponent({
  // 类比默认插槽，将其想成一个函数
  default: (slotProps) => {
    return `${slotProps.text} ${slotProps.count}`
  },
  // 别的名字的插槽
  [name]: ....
})

function MyComponent(slots) {
  const greetingMessage = 'hello'
  return `<div>${
    // 在插槽函数调用时传入 props
    slots.default({ text: greetingMessage, count: 1 })
  }</div>`
}
```

4.可以使用解构

```vue
<MyComponent v-slot="{ text, count }">
  {{ text }} {{ count }}
</MyComponent>
```

5.具名作用域插槽

```vue
<MyComponent>
  <!-- v-slot:name = 'props' 简写就是 #name='props' -->
  <template #header="headerProps">
    {{ headerProps }}
  </template>

  <template #default="defaultProps">
    {{ defaultProps }}
  </template>

  <template #footer="footerProps">
    {{ footerProps }}
  </template>
</MyComponent>

```

6.如果混用了具名插槽和作用域插槽,则需要为默认插槽安排一个显性的`<template>`标签. 给组件直接使用`v-slot`指令会导致编译错误, 避免造成因为默认插槽的props的作用域而困惑

```vue
<!-- 该模板无法编译 -->
<template>
	<!-- 组件内不应该使用v-slot命令
-->
  <MyComponent v-slot="{ message }">
    <p>{{ message }}</p>
    <template #footer>
      <!-- message 属于默认插槽，此处不可用 -->
      <p>{{ message }}</p>
    </template>
  </MyComponent>
</template>

<!-- 修改后 -->
<template>
  <MyComponent>
    <!-- 使用显式的默认插槽 -->
    <template #default="{ message }">
      <p>{{ message }}</p>
    </template>

    <template #footer>
      <p>Here's some contact info</p>
    </template>
  </MyComponent>
</template>

```

