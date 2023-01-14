[TOC]

# 浏览器和JavaScript的运行原理

## 一. 浏览器的运行原理

### 1. 浏览器内核

事实上，我们经常说的浏览器内核指的是浏览器的**排版引擎**:

- 排版引擎(layout engine), 也称为浏览器引擎(browser engine) , 页面渲染引擎(rendering engine) 或样版引擎

### 2. 浏览器渲染原理

#### 1. 总体渲染过程

![image-20221207012117765](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207012117765.png)

- DOM Tree经过渲染生成Render Tree,两者并不相等,(如:有些DOM元素可能并不显示)
- Render Tree 并不包含节点的大小和位置信息, 因此需要Layout(排版)来呈现所有节点的宽度等位置信息
- 最上面的DOM为我们js代码进行的一些DOM操作
- Painting是将元素的可见部分进行绘制

#### 2. 另一种渲染思路

![image-20221207013135140](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221207013135140.png)

#### 3. 回流(重排)

- 在第一次布局后, 对节点的大小,位置进行重新计算布局(layout)称为回流
- **回流是一件非常消耗性能的事,因此要尽量避免**
  - 修改样式要一次性修改, 如用class
  - 避免频繁操作DOM
  - 对某些元素使用position的absolute或者fix
    - 引起相对开销较小的回流,不对其他元素造成影响

触发回流的操作:

1. 改变DOM树结构(添加或移除节点)
2. 改变布局(width, mrgin)
3. 修改window窗口尺寸
4. 调用getComputedStyle方法获取尺寸,位置等信息



#### 4. 重绘

- 第一次绘制渲染后,再一次进行绘制渲染

触发重绘的操作:

1. 修改背景颜色
2. 文字颜色
3. 边框颜色
4. 样式



#### 5. composite合成(性能优化)

- 绘制过程中, 可以将布局后的元素绘制到多个合成图层中(一种浏览器的优化手段)
- 默认情况下,标准流中的内容都是被绘制在同一个图层中
- 一些特殊的属性, 会创建一个新的合成层, 并且新的图层可以利用GPU来加速绘制
  - **每个图层都是单独渲染的**, 因此单独更改某一个图层时,效率更高
- 能生成新的图层层的属性如下:
  - 3D transform
  - video, canvas, iframe
  - opacity 动画转换时
  - position: fixed 注: absolute不会生成新的图层
  - will-change
  - animation 或 transition 设置了 opacity, transform: translateX(Y)
- **分层可以提高绘制性能,但是是以牺牲内存管理为代价, 因此不必作为web性能优化的手段过分使用**



### 3. 浏览器页面解析过程

- 常规的脚本会阻塞HTML的解析, 且在加载完后会立即执行
- async属性会在解析HTML的同时加载script脚本,当脚本加载完成后,会执行script脚本
- defer属性也会在解析HTML的同时加载script脚本,但它是等到HTML解析完成后再执行script脚本

![31010916590848302](C:\Users\zZOMZz\Desktop\js课程截图\31010916590848302.png)

defer和async的使用场景

- defer通常用于需要在文档中解析后操作DOM的Javascript代码, 并且对多个script文件有顺序要求
- async通常用于独立的脚本, 对其他脚本, 甚至DOM都没有依赖



## 二. Javascript的运行原理

### 1. V8引擎

两款JavaScript引擎:

- WebKit
  - WebCore: 负责HTML解析,布局,渲染等工作
  - JavascriptCore: 解析,执行javaScript代码(小程序的js代码就是有这个引擎解析的)
- V8

#### 1.1 V8引擎解析过程总览

[🚀⚙️ JavaScript Visualized: the JavaScript Engine - DEV Community 👩‍💻👨‍💻](https://dev.to/lydiahallie/javascript-visualized-the-javascript-engine-4cdf)

<img src="C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221209194437914.png" alt="image-20221209194437914" style="zoom:200%;" />

![Snipaste_2022-12-09_20-34-52](.\图片\Snipaste_2022-12-09_20-34-52.png)

步骤:

1. 通过Parse模块将源代码解析成AST抽象语法树
2. AST抽象语法树通过Ignition解释器模块, 将AST转换成字节码
3. 一些字节码可以直接交付cpu进行翻译执行
4. 对于常用的字节码,会通过TurboFan模块,生成优化的机器码,直接交付运行,提高效率
5. 对于已经生成的机器码进行修改时,会通过Deoptimization(去优化),重新还原成字节码



#### 1.2 三大模块介绍

##### 1.Parse模块

[官方文档](https://v8.dev/blog/scanner)

AST抽象语法树: 源代码的抽象语法结构的树状表现形式，这里特指编程语言的源代码。树上的每个节点都表示源代码中的一种结构. ( **代表程序结构的一组对象** )

![image-20221209200108661](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221209200108661.png)

![pv4y4w0doztvmp8ei0ki](.\图片\pv4y4w0doztvmp8ei0ki.gif)

![bic727jhzu0i8uep8v0k](.\图片\bic727jhzu0i8uep8v0k.gif)

![sgr7ih6t7zm2ek28rtg6](.\图片\sgr7ih6t7zm2ek28rtg6.gif)

解析过程

1. Blink将源码交给V8引擎, Stream进行编码转换, 将编码转换为字符流
2. Scanner会进行词法分析(lexical analysis)
   1. 拿到从UTF-16流中解码出的Unicode characters(Unicode 字符)
   2. 将由一个或多个具有单一语义含义的字符组成的块(称为标记--Token)交付给Parse和PreParse
3. 在Parse和PreParse中将拿到的Token转换成AST
   1. Prase是直接转换
   2. PreParse称为预解析:
      1. 不是所有的代码马上就能被用上
      2. 实现Lazy Parsing(延迟解析), 将不必要的函数进行预解析
      3. 比如当在函数内部定义另一个函数

##### 2. Ignition(解释器)

[官方文档](https://v8.dev/blog/ignition-interpreter)

![i5f0vmcjnkhireehicyn](.\图片\i5f0vmcjnkhireehicyn.gif)

- 将解析生成的AST解析树解释成字节码
  - 对于常用的code, 会进行优化(optimized)成机器码


![image-20221209202203313](C:\Users\zZOMZz\AppData\Roaming\Typora\typora-user-images\image-20221209202203313.png)

##### 3. TurboFan编译器

[官方文档](https://v8.dev/blog/turbofan-jit)

![ongt4qftovd82sp2vihk](.\图片\ongt4qftovd82sp2vihk.gif)

- 可以将字节码编译为cpu能直接运行的机器码
- 将多次运行的函数标记为**热点函数**, 将其的字节码解析为机器码, 提高代码执行效率
- 当字节码发生改变(如: 变量类型变化), 如果发生这种情况，机器代码将被取消优化，引擎将回退到解释生成的字节代码, 以字节代码的方式运行



