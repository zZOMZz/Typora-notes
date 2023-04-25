[toc]

# Relay

![image-20230324190044432](./assets/image-20230324190044432.png)



- 因为relay为我们掌管所有的data fetch, 所以它静态的直到需要什么数据, 他对整个系统的数据需求有全局的了解,  所以当服务器收到客户端的请求时, 就立马请求数据, 并与代码并行的下载它

![image-20230324190102820](./assets/image-20230324190102820.png)



## HTML flushing(HTML 刷新)

- 当浏览器请求一个HTML文档时, 它会以网络数据包(packages)的形式接收响应, 然后会逐步解析HTML, 这会在整个HTML被完全下载下来之前就开始运行

- 因此可以将一些CSS和js放到HTML的head靠前的位置, 这样它们就会先下载, 后续



![image-20230324190950591](./assets/image-20230324190950591.png)





## Dynamic Import on render

![image-20230324191312469](./assets/image-20230324191312469.png)



### 将代码分为三个阶段

![image-20230324191344385](./assets/image-20230324191344385.png)



![image-20230324191414018](./assets/image-20230324191414018.png)



![image-20230324191441014](./assets/image-20230324191441014.png)



![image-20230324191511542](./assets/image-20230324191511542.png)





## streamed Data Fetch

![image-20230324191754443](./assets/image-20230324191754443.png)