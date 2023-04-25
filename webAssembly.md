[toc]

# webAssembly

- JavaScript执行过程

![image-20230406133757324](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230406133757324.png)

- webAssembly执行过程

![image-20230406133836275](C:/Users/zZOMZz/AppData/Roaming/Typora/typora-user-images/image-20230406133836275.png)



- inerpreter(解释器)
  - 即时编译, 对代码的翻译几乎是逐行进行的
  - 能得到快速响应, 得到即时反馈循环
- compiler(编译器)
  - 在翻译之前需要时间处理



JIT(即使编译)

- 代码开始在解释器中运行(inerpreter), 监视器(monitor)收集相关的信息, 然后取决于这段代码的运行频率, 决定是否将这段代码发送到编译器(compiler)

WebAssembly

- 让库(library)作者和应用程序开发人员能够使用更一致的语言进行编码, 并且该代码能像JavaScript一样在web上运行, 并与现有的JavaScript集成