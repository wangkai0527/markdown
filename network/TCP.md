[TOC]


# TCP

tcp: 传输控制协议,全拼:Transmission Control Protocol 它是一个面向连接，可靠的传输协议

udp: 用户数据报协议,全拼:User Datagram Protocol 它不是面向连接，不是可靠的传输协议, udp协议传输速度快

tcp和udp都是传输层的两个传输协议

tcp的特点:
4.1 面向连接，间接验证对方ip的有效性

4.2 可靠的传输

4.2.1 应答机制:发送数据包完成以后，对方收到数据底层会回复

4.2.2 超时重传:发送数据以后对方没有进行回复，会隔一段时间再次给对方发送数据，如果对方一直没有回复，那么会认为对方已经掉线了

4.2.3 错误校验:如果收到的数据和之前发送数据包的序号不一致，会自动根据需要进行排序，如果收到重复的数据包，会把重复的数据包删除

4.2.4 流量控制:使用tcp能保证接收数据的时候电脑不会卡死

5. tcp和udp的不同点对比
5.1 tcp 面向连接， udp不面向连接

5.2 tcp 能保证数据有效和有序的传输，udp保证不了

5.3 tcp 有超时重传，udp没有

5.4 tcp 有错误校验，如果出现数据包顺序不一致会自动排序，还有如果收到数据包重复会自动删除重复的数据包，udp没有

5.5 tcp 有流量控制 udp没有

5.6 tcp 需要建立连接然后需要资源开销要大， udp不需要建立连接资源开销小

— 扩展

5.7 tcp 适合发送大量的数据,tcp每次发送的数据包理论上没有上限控制，udp每次发送的数据包不能超过64k

5.8 tcp 应用场景: 文件上传和下载、浏览器上网，绝大多数应用程序都是用tcp协议，udp应用场景: 发送广播消息（飞秋上线），音视频传输（qq和微信），包括共屏软件

5.9 tcp 发送数据的时候需要建立连接，udp不需要建立连接，udp发送速度比tcp发送速度要快





## 微信

微信使用了TCP协议和HTTP协议，没有用到UDP协议。微信通讯中使用了HTTP短连接和TCP长连接，并没有bai用到UDP，其中登陆验证和头像身份信息及日志等功能采用的HTTP，文本消息、语音消息、视频消息、图片消息这些使用的是TCP长连接。通过心跳包来维护长连接状态，300S一个心跳。

TCP功能说明：

1.当应用层向TCP层发送用于网间传输的、用8位字节表示的数据流，TCP则把数据流分割成适当长度的报文段，最大传输段大小（MSS）通常受该计算机连接的网络的数据链路层的最大传送单元（MTU）限制。之后TCP把数据包传给IP层，由它来通过网络将包传送给接收端实体的TCP层。

2.TCP为了保证报文传输的可靠，就给每个包一个序号，同时序号也保证了传送到接收端实体的包的按序接收。然后接收端实体对已成功收到的字节发回一个相应的确认(ACK)；如果发送端实体在合理的往返时延(RTT)内未收到确认，那么对应的数据（假设丢失了）将会被重传。

3.在数据正确性与合法性上，TCP用一个校验和函数来检验数据是否有错误，在发送和接收时都要计算校验和；同时可以使用md5认证对数据进行加密。

4.在保证可靠性上，采用超时重传和捎带确认机制。在流量控制上，采用滑动窗口协议，协议中规定，对于窗口内未经确认的分组需要重传。



## QQ
QQ既有UDP也有TCP协议。不管UDP还是TCP，最终登陆成功之后，QQ都会有一个TCP连接来保持在线状态。这个TCP连接的远程端口一般是80，采用UDP方式登陆的时候，端口是8000。

UDP协议是无连接方式的协议，它的效率高，速度快，占资源少，但是其传输机制为不可靠传送，必须依靠辅助的算法来完成传输控制。

QQ采用的通信协议以UDP为主，辅以TCP协议。由于QQ的服务器设计容量是海量级的应用，一台服务器要同时容纳十几万的并发连接，因此服务器端只有采用UDP协议与客户端进行通讯才能保证这种超大规模的服务。

QQ客户端之间的消息传送也采用了UDP模式，因为国内的网络环境非常复杂，而且很多用户采用的方式是通过代理服务器共享一条线路上网的方式，在这些复杂的情况下，客户端之间能彼此建立起来TCP连接的概率较小，严重影响传送信息的效率。而UDP包能够穿透大部分的代理服务器，因此QQ选择了UDP作为客户之间的主要通信协议。

采用UDP协议，通过服务器中转方式。因此，现在的IP侦探在你仅仅跟对方发送聊天消息的时候是无法获取到IP的。大家都知道，UDP 协议是不可靠协议，它只管发送，不管对方是否收到的，但它的传输很高效。但是，作为聊天软件，怎么可以采用这样的不可靠方式来传输消息呢？于是，腾讯采用了上层协议来保证可靠传输；如果客户端使用UDP协议发出消息后，服务器收到该包，需要使用UDP协议发回一个应答包。如此来保证消息可以无遗漏传输。之所以会发生在客户端明明看到“消息发送失败”但对方又收到了这个消息的情况，就是因为客户端发出的消息服务器已经收到并转发成功，但客户端由于网络原因没有收到服务器的应答包引起的。


登陆采用TCP协议和HTTP协议，你和好友之间发送消息，主要采用UDP协议，内网传文件采用了P2P技术。

总来的说：

1.登陆过程，客户端client 采用TCP协议向服务器server发送信息，HTTP协议下载信息。登陆之后，会有一TCP连接来保持在线状态。

2.和好友发消息，客户端client采用UDP协议，但是需要通过服务器转发。腾讯为了确保传输消息的可靠，采用上层协议来保证可靠传输。如果消息发送失败，客户端会提示消息发送失败，并可重新发送。

3.如果是在内网里面的两个客户端传文件，QQ采用的是P2P技术，不需要服务器中转。




