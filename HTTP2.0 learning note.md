# HTTP2.0 learning note

1. [中文](http://io.upyun.com/2015/05/13/http2/)
2. [Wiki pedia](https://en.wikipedia.org/wiki/HTTP/2)

##对比HTTP1.1
相比 HTTP/1.x，HTTP/2 在底层传输做了很大的改动和优化：

1. HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式。二进制格式在协议的解析和优化扩展上带来更多的优势和可能。
2. HTTP/2 对消息头采用 HPACK 进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。头压缩能够很好的解决该问题。
3. 多路复用，直白的说就是所有的请求都是通过一个 TCP 连接并发完成。HTTP/1.x 虽然通过 pipeline 也能并发请求，但是多个请求之间的响应会被阻塞的，所以 pipeline 至今也没有被普及应用，而 HTTP/2 做到了真正的并发请求。同时，流还支持优先级和流量控制。
4. Server Push：服务端能够更快的把资源推送给客户端。例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 再发送这些请求。当客户端需要的时候，它已经在客户端了。

##详解
1. 采用二进制格式进行传输：Frame 是 HTTP/2 二进制格式的基础，基本可以把它理解为它 TCP 里面的数据包一样。HTTP/2 之所以能够有如此多的新特性，正是因为底层数据格式的改变。<br/>
2. 压缩头部Header：HTTP1.1 Header是文本，每次客户端请求服务端的时候都将带上很多文本信息，这浪费了很多服务端带宽；如果约定将常用的请求比如 `GET /index.html` 用一个 `1` 来表示，`POST /index.html` 用 `2` 来表示。那么将可以节省很多字节。
	* 为 HTTP/2 的专门量身打造的 HPACK 便是类似这样的思路延伸。它使用一份索引表来定义常用的 HTTP Header。把常用的 HTTP Header 存放在表里，请求的时候便只需要发送在表里的索引位置即可；
	* HPACK 不仅仅通过索引键值对来降低数据量，同时还会将字符串进行霍夫曼编码来压缩字符串大小；这将对如`User agent`一类的用户自定义的字符串Header进行大幅压缩，减少Header长度；
3. Multipexing 多路复用：每个`Frame Header` 都有一个 `Stream ID` 就是被用于实现该特性。每次请求/响应使用不同的 `Stream ID`。就像同一个 TCP 链接上的数据包通过 `IP:PORT`来区分出数据包去往哪里一样。通过 `Stream ID` 标识，所有的请求和响应都可以欢快的同时跑在一条 TCP 链接上了。 下图是 http 和 spdy(http2 的模型和 spdy 是类似的) 的并发模型对比：<br/>
![](http://onepiece.b0.upaiyun.com/assets/http2request.png)
4. Server Push：当服务端需要主动推送某个资源时，便会发送一个 `Frame Type` 为 `PUSH_PROMISE` 的 Frame，里面带了 PUSH 需要新建的 `Stream ID`。意思是告诉客户端：接下来我要用这个 ID 向你发送东西，客户端准备好接着。客户端解析 Frame 时，发现它是一个 `PUSH_PROMISE` 类型，便会准备接收服务端要推送的流。