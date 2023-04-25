[toc]

# Server Component

- server-component只在服务器端运行, 它们永远不会被发布到客户端
  - 可以用来解决瀑布流问题(请求一个接着一个, 而不是一起异步发送)
  - 还可以使各个组件的数据独立分割起来, 避免互相影响

![image-20230324220135522](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230324220135522.png)

- 有些只在server端渲染时才用的库, 就不用再在客户端下进行下载了



## Server Component与SSR的区别

SSR是在获取请求后在服务器端渲染组件, 生成HTML发送到客户端, 这样在客户端下载JavaScript代码时, 能够显示页面.

**但是Sever Component与SSR不同**, 客户端state永远不会消失, 它会始终保持,即使重新获取Server Component tree 因为Server component不会将脚本渲染成HTML, 它们会渲染成一种特殊的格式 这种格式能够以流的形式下发给客户端



## Server Component的作用

- Server Compoennt have zero effect on the bundle size, 服务端组件对捆绑包的大小影响是0
- 服务端组件能让你直接访问后端的资源
- React IO libraries
  - react-fs: filesystem
  - react-pg: PostgreSQL
  - react-fetch: fetch
- Server Component 让你只下载必要的代码, 自动实现客户端代码分割

- Server Component让你可以决定具体用例的取舍
  - 需要执行数据获取和预处理的代码, 可以将它们放到Server端
  - 如果你有一些代码需要快速交互来立即响应用户输入, 可以将它们放到客户端



将客户端组件与服务端组件混合使用, 可以看为服务端组件在以属性的形式向客户端组件传递数据

![image-20230324234335018](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230324234335018.png)