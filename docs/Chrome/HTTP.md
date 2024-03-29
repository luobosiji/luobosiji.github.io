# HTTP
> 超文本传输协议

## HTTP/0.9
- 只有一个请求行，并没有 HTTP 请求头和请求体
- 服务器也没有返回头信息
- 返回的文件内容是以 ASCII 字符流来传输（因为都是 HTML 格式的文件）

## HTTP/1.0
> 支持多种类型的文件下载是 HTTP/1.0 的一个核心诉求 \
> (浏览器中展示的不单是 HTML 文件了，还包括了 JavaScript、CSS、图片、音频、视频等不同类型的文件)

- 通过请求头和响应头来进行协商
  - 返回什么类型的文件、采取什么形式的压缩、提供什么语言的文件以及文件的具体编码。
  - 引入了状态码
  - 提供了 Cache 机制, 用来缓存已经下载过的数据。
  - 还加入了用户代理的字段(服务器需要统计客户端的基础信息)

## HTTP/1.1
- 增加了持久连接的方法
  - 是在一个 TCP 连接上可以传输多个 HTTP 请求，只要浏览器或者服务器没有明确断开连接，那么该 TCP 连接会一直保持。
  - 目前浏览器中对于同一个域名，默认允许同时建立 6 个 TCP 持久连接
- 不成熟的 HTTP 管线化
  - 这就是著名的**队头阻塞**的问题（需要等待前面的请求返回之后，才能进行下一次请求。）
  - 虽然可以**整批发送请求**，不过服务器依然需要根据请求顺序来回复浏览器的请求。
- 提供虚拟主机的支持
  - 实现在一台物理主机上绑定多个虚拟主机，每个虚拟主机都有自己的单独的域名，这些单独的域名都公用同一个 IP 地址。
  - 请求头中增加了 Host 字段
- 对动态生成的内容提供了完美支持
  - 引入 Chunk transfer 机制，
    - 服务器会将数据分割成若干个任意大小的数据块，每个数据块发送时会附上上个数据块的长度，最后使用一个零长度的块作为发送数据完成的标志。这样就提供了对动态内容的支持。
- 引入了客户端 Cookie 机制和安全机制

### 主要问题
- 对带宽的利用率却并不理想
  - 带宽是指每秒最大能发送或者接收的字节数
  - 原因：
    - TCP 的慢启动（慢启动是 TCP 为了减少网络拥塞的一种策略）
      - 可以把每个 TCP 发送数据的过程看成是一辆车的启动过程
    - 同时开启了多条 TCP 连接，那么这些连接会竞争固定的带宽
    - 队头阻塞的问题。
      - 虽然能公用一个 TCP 管道，但是在一个管道中同一时刻只能处理一个请求，在当前的请求没有结束之前，其他的请求只能处于阻塞状态。

## HTTP/2 的多路复用
- 一个域名只使用一个 TCP 长连接 （解决慢启动）
- 多路复用 （消除队头阻塞问题）
  - 个请求都有一个对应的 ID
  - 可以将请求分成一帧一帧的数据去传输，
    - 可以随意发送，是因为每份数据都有对应的 ID，浏览器接收到之后，会筛选出相同 ID 的内容，将其拼接为完整的 HTTP 响应数据。
    - 就是当收到一个优先级高的请求时，服务器可以暂停之前的请求来优先处理关键资源的请求
- 可以设置请求的优先级
- 服务器推送
- 头部压缩

### 多路复用的实现
- HTTP/2 在协议栈中 添加了一个二进制分帧层
  - 浏览器准备好请求数据
  - 数据经过二进制分帧层处理之后，会被转换为一个个带有请求 ID 编号的帧，通过协议栈将这些帧发送给服务器。
  - 服务器接收到所有帧之后，会将所有相同 ID 的帧合并为一条完整的请求信息。
  - 然后服务器处理该条请求，并将处理的响应行、响应头和响应体分别发送至二进制分帧层。
  - 二进制分帧层会将这些响应数据转换为一个个带有请求 ID 编号的帧，经过协议栈发送给浏览器。
  - 浏览器接收到响应帧之后，会根据 ID 编号将帧的数据提交给对应的请求。

### 主要问题
- TCP的队头阻塞
  - 如果在数据传输的过程中，有一个数据因为网络故障或者其他原因而丢包了，那么整个 TCP 的连接就会处于暂停状态，需要等待丢失的数据包被重新传输过来
- TCP 建立连接的延时
  - 建立连接 需要三次握手，消耗1.5个RTT
    - 网络延迟又称为 RTT（Round Trip Time）。我们把从浏览器发送一个数据包到服务器，再从服务器返回数据包到浏览器的整个往返时间称为 RTT
  - 进行 TLS 连接 ， 大致是需要 1～2 个 RTT
- TCP 协议僵化



## HTTP/3
- QUIC 协议 (基于 UDP 实现了类似于 TCP 的多路数据流、传输可靠性等功能，我们把这套功能称为 QUIC 协议)
  - 实现了类似 TCP 的流量控制、传输可靠性的功能
  - 集成了 TLS 加密功能。
  - 实现了 HTTP/2 中的多路复用功能
  - 实现了快速握手功能。

![HTTP/2 和 HTTP/3 协议栈](./res/HTTP%3A2%20%E5%92%8C%20HTTP%3A3%20%E5%8D%8F%E8%AE%AE%E6%A0%88.webp)

### 主要问题
- 服务器和浏览器端都没有对 HTTP/3 提供比较完整的支持
- 中间设备僵化的问题
  - 这些设备包括了路由器、防火墙、NAT、交换机等
  - 如果我们在客户端升级了 TCP 协议，但是当新协议的数据包经过这些中间设备时，它们可能不理解包的内容，于是这些数据就会被丢弃掉。