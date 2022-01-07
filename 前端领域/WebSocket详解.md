# WebSocket 讲解


## 如何及时获得服务器更新


服务器端的资源经常在更新，客户端需要尽量及时地知道这些更新从而展示给用户。

HTTP 1.1的做法如下图所示，通过 Ajax 轮询来获取服务器端资源的变化。


![image](https://csdn.52wike.com/wike_blog/2022-01-07/a60a2a34-d1c1-4d89-96d9-f2b81d7fa1fc.png)


轮询是在特定的的时间间隔（如每1秒），由浏览器对服务器发出HTTP请求，然后由服务器返回最新的数据给客户端的浏览器。这种传统的模式带来很明显的缺点，即浏览器需要不断的向服务器发出请求，然而HTTP请求可能包含较长的头部，其中真正有效的数据可能只是很小的一部分，显然这样会浪费很多的带宽等资源。

## 早期服务器和客户端之间的双工通信技术


-  轮询（Polling）

-  长轮询（Long Polling）



### 轮询（Polling）
轮询可以定义为一种方法，无论传输中存在哪些数据，它都执行周期性请求。定期请求以同步方式发送。客户端在指定的时间间隔内向服务器发出定期请求。服务器的响应包括可用数据或其中的一些警告消息。

这种方式下，是不适合获取实时信息的，每隔一段时间就询问一次。客户端会轮询，有没有新消息。这种方式连接数会很多，一个接受，一个发送。而且每次发送请求都会有 HTTP 的 Header，会很耗流量，也会消耗 CPU 的利用率。

### 长轮询（Long Polling）

长轮询类似轮询的技术。客户端和服务器保持连接处于活动状态，直到获取某些数据或发生超时。如果由于某些原因导致连接丢失，则客户端可以重新开始并执行顺序请求。

长轮询是对轮询的改进版，客户端发送 HTTP 给服务器之后，有没有新消息，如果没有新消息，就一直等待。直到有消息或者超时了，才会返回给客户端。消息返回后，客户端再次建立连接，如此反复。这种做法在某种程度上减小了网络带宽和 CPU 利用率等问题。

## WebSocket解决的问题


WebSocket 解决的第一个问题是，通过第一个 HTTP request 建立了 TCP 连接之后，之后的交换数据都不需要再发 HTTP request了，使得这个长连接变成了一个真.长连接。但是不需要发送 HTTP header就能交换数据显然和原有的 HTTP 协议是有区别的，所以它需要对服务器和客户端都进行升级才能实现。在此基础上 WebSocket 还是一个双通道的连接，在同一个 TCP 连接上既可以发也可以收信息。此外还有 multiplexing 功能，几个不同的 URI 可以复用同一个 WebSocket 连接。这些都是原来的 HTTP 不能做到的。


## WebSocket 浅析

1. WebSocket 本质上跟 HTTP 完全不一样，只不过为了兼容性，WebSocket 的握手是以 HTTP 的形式发起的，如果服务器或者代理不支持 WebSocket，它们会把这当做一个不认识的 HTTP 请求从而优雅地拒绝掉。

2. WebSocket是一种双向通信协议。在建立连接后，WebSocket服务器端和客户端都能主动向对方发送或接收数据，就像Socket一样WebSocket需要像TCP一样，先建立连接，连接成功后才能相互通信。

## WebSocket模式客户端与服务器请求响应模式

![image](https://csdn.52wike.com/wike_blog/2022-01-07/5aa0cbb9-df65-41c4-8cbd-86b6476d749f.png)

## 升级过程

客户端请求：
```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Origin: http://example.com
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

客户端发起的WebSocket连接报文类似传统HTTP报文，Upgrade：websocket参数值表明这是WebSocket类型请求，Sec-WebSocket-Key是WebSocket客户端发送的一个 base64编码的密文，要求服务端必须返回一个对应加密的Sec-WebSocket-Accept应答，否则客户端会抛出Error during WebSocket handshake错误，并关闭连接。

服务器回应：

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat
```

Sec-WebSocket-Accept的值是服务端采用与客户端一致的密钥计算出来后返回客户端的，HTTP/1.1 101 Switching Protocols表示服务端接受WebSocket协议的客户端连接，经过这样的请求-响应处理后，两端的WebSocket连接握手成功, 后续就可以进行TCP通讯了。用户可以查阅WebSocket协议栈了解WebSocket客户端和服务端更详细的交互数据格式。


## 为什么需要心跳包

因为网络世界的一些缺陷性设计。WebSocket虽然解决了服务器和客户端两边的问题，但坑爹的是网络应用除了服务器和客户端之外，另一个巨大的存在是中间的网络链路。一个 HTTP/WebSocket 连接往往要经过无数的路由，防火墙。你以为你的数据是在一个“连接”中发送的，实际上它要跨越千山万水，经过无数次转发，过滤，才能最终抵达终点。在这过程中，中间节点的处理方法很可能会让你意想不到。所以你需要保持连接存活，一般数据长时间不发送默认有一个超时时间，中间节点可能会断开，所以我们通过心跳包来保持链路的激活状态，一般60秒以内，具体可以根据业务定，但不要超过60秒

## 客户端数据类型

服务器数据可能是文本，也可能是二进制数据（blob对象或Arraybuffer对象）。

```

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
除了动态判断收到的数据类型，也可以使用binaryType属性，显式指定收到的二进制数据类型。
```
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
## 客户端的简单示例

```
// Create WebSocket connection.
const socket = new WebSocket('ws://localhost:8080');

// Connection opened
socket.addEventListener('open', function (event) {
    socket.send('Hello Server!');
});

// Listen for messages
socket.addEventListener('message', function (event) {
    console.log('Message from server ', event.data);
});
```
