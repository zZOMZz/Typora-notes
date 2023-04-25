[toc]

# React学习时零碎的知识

## 一. onClick和onClickCapture

- 两者都是react中的事件处理函数属性. 区别在于冒泡机制的不同
  - `onClick`: 使用默认的冒泡机制: 从子元素向父元素传递, 直到根元素, 如果多个元素上都注册了onClick函数, 则会从最底层开始一次触发
  - `onClickCapture`: 使用事件捕获机制, 即从根元素开始, 向下传递到子元素. 如果多个元素都注册了相同事件类型的`onClickCapture`事件处理程序, 那么事件会从根元素开始捕获