---
title: JS逆向之WebSocket协议
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-27 17:45:51
password:
summary:
tags: [JS, 逆向, WebSockets]
categories: [JS逆向]
---



<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/Hlp8XD0R5qo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



### 前言

本文的目的不是着重讲WebSockets，而是解决在JS逆向过程中遇到网站使用WebSockets协议的时候如何处理。



### WebSocket

#### 什么是WebSocket

WebSocket是一种网络传输协议，可在单个TCP连接上进行全双工通信，位于OSI模型的应用层。WebSocket使得客户端和服务器之间的数据交换变得更加简单，允许服务端主动向客户端推送数据。WebSocket的最大特点就是浏览器和服务器只需要完成一次握手，两者之间就可以创建持久性的连接，并进行双向数据传输。



WebSocket协议规范将`ws`（WebSocket）和`wss`（WebSocket Secure）定义为两个新的统一资源标识符(URI)方案，分别对应明文和加密连接。除了方案名称和片段ID（不支持`#`）之外，其余的URI组件都被定义为此URI的通用语法。



![img](https://img.heshipeng.com/202203272028182.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



WebSocket的其它特点：

1. 建立在 TCP 协议之上，服务器端的实现比较容易。
2. 与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。
3. 数据格式比较轻量，性能开销小，通信高效。
4. 可以发送文本(json/xml/纯文本)，也可以发送二进制数据(protobuf)。
5. 没有同源限制，客户端可以与任意服务器通信。
6. 协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。如ws(wss)://example.com:80/some/path。



![img](https://img.heshipeng.com/202203272034038.jpg?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 为什么需要WebSocket

我们已经有了 HTTP 协议，为什么还需要另一个协议？它能带来什么好处？答案很简单，因为 HTTP 协议有一个缺陷：通信只能由客户端发起。试想一下，现在有个场景，比如聊天室。如果A用户与B用户聊天，如果用HTTP协议，只能是A用户客户端向服务器发起一个HTTP请求，询问是否有B用户的消息，同样的对于B用户也一样。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。就只能用轮询的方式：每隔一段时间就发出一个询问，了解服务器有没有新的信息。



轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。



#### 握手协议

WebSocket 是独立的、创建在TCP上的协议。Websocket 通过HTTP

##### 协议说明

客户端请求：

 ```
 GET /chat HTTP/1.1
 Host: server.example.com
 Upgrade: websocket
 Connection: Upgrade
 Origin: http://example.com
 Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
 Sec-WebSocket-Protocol: chat, superchat
 Sec-WebSocket-Version: 13
 ```

服务器回应：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

字段说明：

* Connection必须设置Upgrade，表示客户端希望连接升级。
* Upgrade字段必须设置Websocket，表示希望升级到Websocket协议。
* Sec-WebSocket-Key是随机的字符串，服务器端会用这些数据来构造出一个SHA-1的信息摘要，然后进行Base64编码。
* Sec-WebSocket-Version 表示支持的Websocket版本。
* Origin字段是必须的。如果缺少origin字段，WebSocket服务器需要回复HTTP 403 状态码（禁止访问）。



#### WebSocket NodeJS版的客户端API

##### 简单示例

客户端代码：

```js
const WebSocket = require('ws')
const ws = new WebSocket('ws://localhost:3000')

// 接受
ws.on('message', (message) => {
    console.log(message.toString())

    // 当数字达到 10 时，断开连接
    if (Number.parseInt(message) === 10) {
        ws.send('close');
        ws.close()
    }
})
```



服务端代码：

```js
const WebSocket = require('ws')
const WebSocketServer = WebSocket.Server;

// 创建 websocket 服务器 监听在 3000 端口
const wss = new WebSocketServer({port: 3000})

// 服务器被客户端连接
wss.on('connection', (ws) => {
    // 通过 ws 对象，就可以获取到客户端发送过来的信息和主动推送信息给客户端

    let i = 0;
    setInterval(function f() {
        ws.send(i++) // 每隔 1 秒给连接方报一次数
    }, 1000)
})
```



##### 详细API介绍

1. WebSocket 构造函数

WebSocket 对象作为一个构造函数，用于新建 WebSocket 实例。

```js
var ws = new WebSocket('ws://localhost:8080');
```

执行上面语句之后，客户端就会与服务器进行连接。



2. webSocket.readyState

`readyState`属性返回实例对象的当前状态，共有四种。

- CONNECTING：值为0，表示正在连接。
- OPEN：值为1，表示连接成功，可以通信了。
- CLOSING：值为2，表示连接正在关闭。
- CLOSED：值为3，表示连接已经关闭，或者打开连接失败。

下面是一个示例：

```js
switch (ws.readyState) {
  case WebSocket.CONNECTING:
    // do something
    break;
  case WebSocket.OPEN:
    // do something
    break;
  case WebSocket.CLOSING:
    // do something
    break;
  case WebSocket.CLOSED:
    // do something
    break;
  default:
    // this never happens
    break;
}
```



3. webSocket.onopen

实例对象的`onopen`属性，用于指定连接成功后的回调函数。

```js
ws.onopen = function () {
  ws.send('Hello Server!');
}
```

如果要指定多个回调函数，可以使用`addEventListener`方法。

```js
ws.addEventListener('open', function (event) {
  ws.send('Hello Server!');
});
```



4. webSocket.onclose

实例对象的`onclose`属性，用于指定连接关闭后的回调函数。

```js
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};

ws.addEventListener("close", function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
});
```



5. webSocket.onmessage

实例对象的`onmessage`属性，用于指定收到服务器数据后的回调函数。

```js
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};

ws.addEventListener("message", function(event) {
  var data = event.data;
  // 处理数据
});
```

注意，服务器数据可能是文本，也可能是二进制数据（`blob`对象或`Arraybuffer`对象）。

```js
ws.onmessage = function(event){
  if(typeof event.data === String) {
    console.log("Received data string");
  }

  if(event.data instanceof ArrayBuffer){
    var buffer = event.data;
    console.log("Received arraybuffer");
  }
}
```

除了动态判断收到的数据类型，也可以使用`binaryType`属性，显式指定收到的二进制数据类型。

```js
// 收到的是 blob 数据
ws.binaryType = "blob";
ws.onmessage = function(e) {
  console.log(e.data.size);
};

// 收到的是 ArrayBuffer 数据
ws.binaryType = "arraybuffer";
ws.onmessage = function(e) {
  console.log(e.data.byteLength);
};
```



6. webSocket.send()

实例对象的`send()`方法用于向服务器发送数据。

发送文本的例子。

```js
ws.send('your message');
```

发送 Blob 对象的例子。

```js
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```

发送 ArrayBuffer 对象的例子。

```js
// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);

```



7. webSocket.bufferedAmount

实例对象的`bufferedAmount`属性，表示还有多少字节的二进制数据没有发送出去。它可以用来判断发送是否结束。

```js
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```



8. webSocket.onerror

实例对象的`onerror`属性，用于指定报错时的回调函数。

```js
socket.onerror = function(event) {
  // handle error event
};

socket.addEventListener("error", function(event) {
  // handle error event
});
```



#### 逆向案例介绍

##### 蝌蚪聊天室

网址：http://kedou.workerman.net/

按照JS逆向的步骤，即抓包，分析包信息，调试，本地运行。先进行抓包，网络面板过滤器选择WS，如下：

![image-20220327225044186](https://img.heshipeng.com/202203272250420.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点中这个WebSocket请求，然后切换到Message面板，如图：

![image-20220327225356124](https://img.heshipeng.com/202203272253279.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这个Message信息是实时的消息，因为WebSocket一经连接，除非断开就会一直存在，会实时发送消息。上边每条消息前面都有一个箭头，红色的表示服务器发给客户端的，绿色的箭头表示我们发送给服务器的。随便选中一条信息，可以看到它的详细信息，很显然这里的消息格式是使用JSON。

接下来就是断点分析，我们可以通过这个WebSocket请求的Initiator找到相应的代码：

![image-20220327230133082](https://img.heshipeng.com/202203272301258.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

也可以通过全局搜索关键字`.onopen`，如图：

![image-20220327230318560](https://img.heshipeng.com/202203272303646.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

不管用什么方法，都会定位到如下的代码段：

![image-20220327230426306](https://img.heshipeng.com/202203272304410.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

点进去`onMessage`方法， 然后打上断点，然后刷新网页，成功断上：

![image-20220327230536234](https://img.heshipeng.com/202203272305422.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这里的数据消息比较简单，明文JSON，没有任何的加密。



#### 参考链接

[WebSocket 教程](https://www.ruanyifeng.com/blog/2017/05/websocket.html)



### 总结

WebSocket与HTTP都是网络传输协议，它们的相同点是：

* 建立在TCP之上，通过TCP协议来传输数据
* 都是可靠性传输协议
* 都是应用层协议

它们的不同点：

* WebSocket是HTML5中的协议，支持持久连接，HTTP不支持持久连接
* HTTP是单向协议，只能由客户端发起，做不到服务器主动向客户端推送信息
