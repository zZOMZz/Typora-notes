[toc]



# Storage存储

- localStorage: 本地存储，提供的是一种永久性的存储方式
- sessionStorage: 会话存储， 提供的是本次会话的存储， 在关掉会话时， 存储的内容会被清除

### 一. localStorage

注意点:

- 存储和拿取的都是字符串形式因此如果需要存取或者拿到别的类型的数据需要调用函数
  - 存取: JSON.stringify(..)
  - 拿取: JSON.parse(...)

```js
// 1.通过key获取内容
localStorage.getItem(key)
// 2.设置一个属性
localStorage.setItem(key,value)
```



## 二. sessionStorage

- 关闭网页后再打开,localStorage会保留,而sessionStorage会删除
- 页面内跳转时都会保留
- 页面外跳转,打开了一个新的网页,localStorage会保留,而sessionStorage不会保留

```js
// 与localStorage相似
sessionStorage.setItem(...)
```



## 三. Cookie

- Cookies, 一种**"小型文本文件"**, 某些网站为了辨别用户身份存储在用户本地终端(Client Side) 上的数据
  - 浏览器会在特定的情况下携带上cookie来发送请求, 我们也可以通过cookie来获取一些信息
- cookie保存在客户端中,按存储位置分为:
  - 内存cookie: 由浏览器维护,保存在内存中,浏览器关闭时cookie就会消失,存在时间短暂
  - 硬盘cookie保存在硬盘中, 存在一个**过期时间**,用户手动清理或者过期时间到时, 才会被清理

- 详细的知识在node-js笔记中记录
- 现在越来越多的公司利用Token来取代cookie
