# WebSocket

`web`领域的实时推送技术，也被称作`RealTime`技术，这种技术要到达的目的是`让用户不需要刷新浏览器就可以获得实时更新`。它有着更广泛的应用场景。

`webSocket protocol`是`Html 5`一种新协议。它实现了浏览器与服务器的全双工通信，一开始的握手需要借助`Http`请求来完成





## 一、传统的Socket技术

长连接

好处：可以实现客户端和服务端的双向通信

缺点：如果连接数过多，那么将一直耗费服务器资源



## 二、webSocket的实现方式

它是一种长连接，只能通过一次请求来初始化连接，然后所有的请求和响应都是通过这个`TCP`链接进行通信，这意味着它是一种基于事件驱动的，异步的消息机制

详细的通信过程

1、`Http`请求头中

`upgrade:websocket`：代表客户端准备使用`websocket`协议进行通信，服务端如果支持的话，咱们就交换`websocket`协议吧

`Sec-WebSocket-Key`：是一种验证服务端是否支持`webSocket`的验证算法，与响应中的`sec-websocket-accept`是对应的

`Sec-WebSocket-Version`：是指浏览器支持的`webSocket`的版本号，注意不会出现9-12的版本号，因为`webSocket`协议规定9-12是保留字段

2、`Http`响应的头中

`Sec-WebSocket-Accept`：

3、传输数据（双向）