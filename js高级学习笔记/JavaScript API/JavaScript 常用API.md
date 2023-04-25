[toc]

# JavaScript 常用API

## 一. TextDecoder(解码)

- `TextDecoder`: 将字节数组(ArrayBuffer)(默认是UTF-8)转换为字符串
  - `decode`

```js
const encodedData = new Uint8Array([72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]); // "Hello World" 的字节数组
const decoder = new TextDecoder();
const decodedData = decoder.decode(encodedData); // 将字节数组解码为字符串
console.log(decodedData); // "Hello World"
```



## 二. TextEncoder(编码)

- `TextEncoder`: 可以将字符串转换编码为字节数组(ArrayBuffer)
  - `encode`

```js
const str = "Hello World"; // 要编码的字符串
const encoder = new TextEncoder();
const encodedData = encoder.encode(str); // 将字符串编码为字节数组
console.log(encodedData); // Uint8Array [72, 101, 108, 108, 111, 32, 87, 111, 114, 108, 100]
```





## 三. AbortController

- `AbortController`: 用于取消网络请求或其他异步操作
  - `abort`: 发出一个信号(signal)来取消操作
  - `signal`: 将signal与controller和fetch请求相关联
    - `aborted`: boolean, controller是否进行了abort

```js
const controller = new AbortController();
const signal = controller.signal;

fetch('/api/data', { signal })
  .then(response => {
    // handle response
  })
  .catch(error => {
    if (signal.aborted) {
      console.log('Fetch aborted');
    } else {
      console.error('Fetch error:', error);
    }
  });

// later, 上面的请求将会返回一个被reject的Promise
controller.abort();

```



## 四. Stream API

[使用可读流 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API/Using_readable_streams)

### 1. 简单介绍

- `Request.body`和`Response.body`属性会将主体能容暴露为一个可读流(ReadableStream)的getter, 只需要简单访问响应的`body`就行

```js
// Fetch the original image
fetch('./tortoise.png')
  // Retrieve its body as ReadableStream
  .then((response) => response.body)

```

- 获取`ReadableStream`后, 需要为它附着一个`reader`, 用来处理这个流
  - `reader`会锁定这个流, 避免这个流同时被其他的`reader`使用
  - 可以使用`ReadableStream.tee()`创建两个分支, 这样就能使两个`reader`处理这个流了
  - 释放流需要使用`ReadableStreamDefaultReader.releaseLock()`

```js
// Fetch the original image
fetch('./tortoise.png')
  // Retrieve its body as ReadableStream
  .then((response) => {
    const reader = response.body.getReader();
    // …
  });

```

- 获取`reader`之后可以使用`ReadableStreamDefaultReader.read()`方法从流中读取数据分块

```js
// Fetch the original image
fetch('./tortoise.png')
  // Retrieve its body as ReadableStream
  .then((response) => {
    const reader = response.body.getReader();
    return new ReadableStream({
      start(controller) {
        return pump();
        function pump() {
          return reader.read().then(({ done, value }) => {
            // When no more data needs to be consumed, close the stream
            if (done) {
              controller.close();  // 关闭自定义流
              return;
            }
            // Enqueue the next data chunk into our target stream
            controller.enqueue(value);
            return pump();
          });
        }
      }
    })
  })
  // Create a new response out of the stream
  .then((stream) => new Response(stream))
  // Create an object URL for the response
  .then((response) => response.blob())
  .then((blob) => URL.createObjectURL(blob))
  // Update image
  .then((url) => console.log(image.src = url))
  .catch((err) => console.error(err));

```



### 2. 创建自定义的可读流

- 通过创建自定义可读流, 可以处理从可读流中读取的数据

```js
const stream = new ReadableStream({
  start(controller) {

  },
  pull(controller) {

  },
  cancel() {

  },
  type,
  autoAllocateChunkSize,
}, {
  highWaterMark: 3,
  size: () => 1,
});
```

- `start(controller)`:  在`ReadableStream`构建后, 立即被调用. 应该包含设置流功能的代码

- `pull()`: 当被包含时, 它会被重复的调用直到填满流的内置队列
- `cancel`: 当应用发出流被取消的信号, 它将被调用
- `type`和`autoAllocateChunkSize`: 当它们被包含时, 会被用来表示流将是一个字节流



### 3. 一个自定义例子

- 创建一个自定义ReadableStream(stream), 并建立起与其有关的响应(res)
- 实现的效果, 当stream往下游传入一个字符, 会马上被下游的响应res接收到, 并在while循环中进行处理
- 这样当上游传输字符, 下游能进行一个一个流式处理

```js
const stream = new ReadableStream({
    start(controller) {
      interval = setInterval(() => {
        let string = randomChars();

        // Add the string to the stream
        // 将字符添加进流, 使下游能接收到
        controller.enqueue(string);

        // show it on the screen
        let listItem = document.createElement('li');
        listItem.textContent = string;
        list1.appendChild(listItem);
      }, 1000);

      button.addEventListener('click', function() {
        clearInterval(interval);
        teeStream();
        controller.close();
      })
    },
    pull(controller) {
      // We don't really need a pull in this example
    },
    cancel() {
      // This is called if the reader cancels,
      // so we should stop generating strings
      clearInterval(interval);
    }
  });

 async function requestStream() {
    // 建立联系
    const res = new Response(stream)
    const reader = res.body.getReader()
    // 程序停留在这, 不断从流中读取数据
    while(true) {
      const content = await reader.read()
      if(!content || content.value === "DONE") {
        console.log("DONE")
        break
      }
      appendList(content.value)
    }
  }
  requestStream()
```





## 五. EventSource

[使用服务器发送事件 - Web API 接口参考 | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/API/Server-sent_events/Using_server-sent_events)

- `EventSource`: 是服务器推送的一个网络事件接口. 一个EventSource实例会对HTTP服务开启一个持久化连接. 一旦连接开启, 来自服务端传入的消息会以**事件**的形式分发至你的代码中
  - 如果接收消息中有一个事件字段, 触发的事件和事件字段的值相同. 如果没有事件字段存在, 则会触发通用事件
- 与`webSockets`不同, 服务端推送是单向的. 数据信息被单向从服务端到客户端分发
  - 当不需要以消息形式将数据从客户端发送到服务器时, 这使他们称为绝佳的选择
    - 如处理社交媒体状态更新
    - 或将数据传递到客户端存储机制(如: indexedDB或Web存储之类的)

### 1. 从服务器接受事件

1. 指定接受事件的URI

```js
const eventSource = new EventSource("ssedemo.php")
```

2. 初始化事件源之后, 对`message`事件添加一个处理函数, 开始监听从服务器发出的消息
   1. `message`事件是对从服务器发送来的没有指定事件类型的信息(没有`event`字段)的消息处理

```js
evtSource.onmessage = function(event) {
  const newElement = document.createElement("li");
  const eventList = document.getElementById("list");

  newElement.innerHTML = "message: " + event.data;
  eventList.appendChild(newElement);
}
```

3. 也可以使用`addEventListener()`方法监听其他类型的事件
   1. `event === "ping"`

```js
evtSource.addEventListener("ping", function(event) {
  const newElement = document.createElement("li");
  const time = JSON.parse(event.data).time;
  newElement.innerHTML = "ping at " + time;
  eventList.appendChild(newElement);
});
```



### 2. 服务器端如何发送事件流

- 服务器端发送的响应内容应该使用值为`text/event-stream`的MIME类型, 每个通知以文本块的形式发送, 并以换行符结尾
- 下面的代码会让服务器每隔一段时间生成一个事件流并返回, 其中每条消息的事件类型为"ping"

```php
// php
date_default_timezone_set("America/New_York");
header("Cache-Control: no-cache");
header("Content-Type: text/event-stream");

$counter = rand(1, 10);
while (true) {
  // Every second, send a "ping" event.

  echo "event: ping\n";
  $curDate = date(DATE_ISO8601);
  echo 'data: {"time": "' . $curDate . '"}';
  echo "\n\n";

  // Send a simple message at random intervals.

  $counter--;

  if (!$counter) {
    echo 'data: This is a message at time ' . $curDate . "\n\n";
    $counter = rand(1, 10);
  }

  ob_end_flush();
  flush();
  sleep(1);
}

```



### 3. 事件流格式

- 字段:
  - `event`
  - `data`
  - `id`
  - `retry`



### 4. SSE和ReadableStream的关系

- 可以在客户端接收SSE, 并对其进行处理, 写入一个ReadableStream中
  - 如: openai的API返回的就是一个SSE, 通过使用eventsource-parser来进行解析

```js
const stream = new ReadableStream({
    async start(controller) {
      function onParse(event: any) {
        if (event.type === "event") {
          const data = event.data;
          // 当服务器传输结束, 最后一个chunk的数据
          if (data === "[DONE]") {
            controller.close();
            return;
          }
          try {
            const json = JSON.parse(data);
            const text = json.choices[0].delta.content;
            // 将字符转换为Unit8Array, 在流中传输Uint8Array类型的数据, 以减少数				据传输的大小
            const queue = encoder.encode(text);
            // 将解析后的数据写入ReadableStream中, 传递给下游
            controller.enqueue(queue);
          } catch (e) {
            controller.error(e);
          }
        }
      }

      const parser = createParser(onParse);
      // 从res.body这个ReadableStream中读取数据, 获得chunk
      for await (const chunk of res.body as any) {
        // 通过parser.feed()来调用上面定义的onParser()函数, 来解析SSE流
        // 之所以要先解码, 是需要通过解码后的字符串判断这个数据是否完整, 通过"\n"来			判断event是否结束
        parser.feed(decoder.decode(chunk, { stream: true }));
      }
    },
  });
```



## 六. Navigator

- `navigator`是一个表示浏览器本身的全局对象, 它提供了许多关于浏览器, 设备, 网络等信息的属性和方法

### 1. navigator常用属性

- `navigator.language`为只读属性, 返回一个表示用户偏好语言的的字符串
- `navigator.geolocation`: 提供一个异步的Geolocation API, 允许Web应用程序访问用户的位置信息

```js
if (navigator.geolocation) {
  // 支持 geolocation API
   navigator.geolocation.getCurrentPosition(successCallback, 		 errorCallback);

} else {
  // 不支持 geolocation API
}
function successCallback(position) {
  const latitude = position.coords.latitude; // 纬度
  const longitude = position.coords.longitude; // 经度
  const accuracy = position.coords.accuracy; // 精度，以米为单位
  // do something with the position information
}

```

- `navigator.platform`: 返回正在运行的操作系统平台, 如"win32"
- `navigator.onLine`: 是否连接到互联网
- `navigator.cookieEnabled`: 是否启用了cookie
- `navigator.serviceWorker`: 提供一个API, 允许Web应用程序注册和控制Service Worker, 以便在脱机是仍能访问应用程序
- `navigator.clipboard`: 提供一个异步的API, 允许Web应用程序读取和修改剪切板中的内容
- `navigator.mediaDevices`: 提供一个异步的API, 允许web应用程序访问用户的媒体设备, 如摄像头和麦克风





## 七. Navigation

- `Navigation`和`Navigator`是两个不同的JavaScript对象

  - `Navigation`是`window`对象的属性之一, 用于控制浏览器的历史记录和页面导航
    - `back`: 浏览器回退
    - `forward`: 浏览器前进
    - `go`: 跳转到指定页面

  - `navigator`则是关注与浏览器本身的信息和功能





## 八. location

- `location`对象表示当前窗口中加载文档的URL信息. 提供访问和操作当前URL的接口

### 1. 常用属性和方法

- 下面的属性和方法有时以`location.xxx`的形式(这里为了方便省略)

- `href`: 获取或设置当前页面的完整URL
- `protocol`: 获取或设置当前页面的协议, 如"http"或"https"
- `host`: 获取或设置当前页面的主机名和端口号
- `hostname`: 获取或设置当前页面的主机名
- `port`: 获取或设置当前页面的端口号
- `pathname`: 获取或设置当前页面的路径部分
- `search`: 获取或设置当前页面的查询字符串部分
- `hash`: 获取或设置当前页面的hash锚点部分



- `assign(url)`: 将当前页面导航到指点的URL, 会在浏览器历史中创建一个新的条目, 使用户能通过浏览器的后退按钮返回到之前的页面
- `reload()`: 重新加载当前页面
- `replace(url)`: 用指定的URL替换当前页面, 会在浏览器历史记录中替换当前的条目, 



## 九.  Service Worker





## 十. Window

```js
window === document.defaultView // 两者相等
```

### 1. getComputedStyle

- `window.getComputedStyle`方法返回一个包含所有计算样式属性和对应值的对象(只读)

```js
<div id="box" style="width: 200px; height: 100px; background-color: red;"></div>
```

```js
const box = document.getElementById("box");
const styles = window.getComputedStyle(box);

console.log(styles.width);  // "200px"
console.log(styles.height);  // "100px"
console.log(styles.backgroundColor);  // "rgb(255, 0, 0)"

```

- element
- pseudoElt: 指定一个要匹配的伪元素的字符串
- style: 是一个是实时的`CSSStyleDeclaration`对象, 当元素的样式更改时, 它会自动更新本身



### 2. confirm

`window.confirm: (content: string) => boolean`

- 弹出一条系统消息, 根据用户点击的`是/否`按钮确定返回的结果(boolean)
- 会阻塞之后的代码执行, 因此必要时再使用

```js
clearAllData() {
    if (confirm(Locale.Store.ConfirmClearAll)) {
      localStorage.clear();
      location.reload();
    }
},
```



### 3. Prompt

- `window.prompt`: 弹出一个输入框, 要求用户输入, 并将结果返回

```js
result = window.prompt(text, value); // 会弹出一个显示框
```





## 十一. DOM

### 1. getBoundingClientRect

- 可以获取指定元素相对于视口的位置和大小信息. 它返回一个'DOMRect'对象
- 常用属性
  - `x`: 元素左边缘相对于视口左侧的距离
  - `y`: 元素上边缘相对于视口顶部的距离
  - `width`: 元素的宽度
  - `height`: 元素的高度
  - `top`: 元素上边缘相对于视口左侧的距离



## 十二. KeyboardEvent

### 1. altkey

- `KeyboardEvent.altKey`: boolean, 只读, 表示事件触发的时候, 键盘上的`alt`是否是按下的, 按下为true, 没按下为false