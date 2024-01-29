# 浏览器输入网址回车后会发什么


<!--more-->

# http应用层

`http`协议是应用层协议。如果抛开细节不谈，一个简单的网页请求可以用下面这个图表示整个过程。

{{% figure class="center" src="/img/net/32.png" alt="img" %}}

## 解析url

浏览器的第一步工作就是解析`url`。访问网页的基本原理就是将服务器存放的`html`文件内容将以`http`协议发送给客户端。

{{% figure class="center" src="/img/net/33.png" alt="img" %}}

当没有路径名时，也就是`http://server-domain/`,会访问服务器根目录下的`/index.html`或者`/defalut.html`。

对`url`解析后，浏览器开始根据请求路径来封装`http`协议消息体，如图所示：
{{% figure class="center" src="/img/net/34.png" alt="img" %}}

`http`消息格式要发给服务器，需要`tcp`来承载，而`tcp`又需要`IP`来承载。所以浏览器的第二步需要知道服务器的`IP`是多少

## 域名解析

需要注意的是`url`上如果直接填写的`ip`+`port`的方式，不会有域名解析的步骤例如:

````bash
curl 'http://47.100.163.207:8888/api/index.html'
````

所谓的域名解析，实际上是域名和`IP`的映射关系。这个`IP`就是服务器的`IP`。存放映射关系的地方叫`DNS`(**D**omain **N**ame **S**ystem)。`DNS`中的域名以英文`.`分割，例如`www.baidu.com`，表示了不同层级，越靠右边层级越高。所以域名结构是一个棵树状结构

{{% figure class="center" src="/img/net/35.png" alt="img" %}}

客户端要想通过域名拿到`IP`,需要分几种情况讨论。

- 查看浏览器的域名缓存
- 查看本地`hosts` (`cat /etc/hosts`)
- 请求本地域名服务器
- 本地域名服务器请求根域名服务器
- 本地域名服务器请求顶级域名服务器比如.com
- ...........
- 最终查到`IP`

{{% figure class="center" src="/img/net/36.png" alt="img" %}}

拿到`IP`后开始封装各种底层协议

# 传输层tcp协议

`tcp`头部报文格式，如图所示：

{{% figure class="center" src="/img/net/37.png" alt="img" %}}

- source port：源端口，发起方端口
-  destination port：目的端口，接收方端口
- sequence number：序列号，保证数据包不乱序
- acknowledgment number：确认序列号，确认对方是否收到，如果没有收到就重发，保证不丢包
- header len：首部长度
- reserved：保留
- 状态位
  - FIN：结束连接请求
  - SYN：发起一个连接请求
  - RST：重新连接
  - PSH：?
  - ACK：回复
  - URG：?
- window size：窗口大小，流量控制，通信双方声明一个窗口，表示自己当前处理能力。
- Check sum：校验和
- urgent pointer：紧急指针
- option：选项
- data：数据，data最大`MSS`字节

## 三次握手

{{% figure class="center" src="/img/net/38.png" alt="img" %}}

- 客户端发出`SYN` 包后进入`SYS_SENT`状态
- 服务器收到`SYN` 包后，发出`SYN`&`ACK`包后，进入`SYN_RCVD`状态
- 客户端收到`SYN`&`ACK`包，发出`ACK`包后，进入``ESTABLISHED`状态
- 服务器收到`ACK`包后，进入``ESTABLISHED`状态

`tcp`的连接状态，可以通过`netstat -napt`命令查看

{{% figure class="center" src="/img/net/39.jpeg" alt="img" %}}

## tcp拆分数据

如果`http`请求的消息过长，超过了`MSS`长度。`tcp`协议会把数据拆成一个个的数据包分别发送

{{% figure class="center" src="/img/net/40.png" alt="img" %}}

- 一个网络包的最大长度，以太网中一般为 `1500` 字节。
- 除去 `IP` 和 `tcp` 头部之后，一个网络包所能容纳的 TCP 数据的最大长度。

## 构造tcp报文

`tcp`头部中，源端口是客户端随机生成的，目的端口是服务器的端口，`http`默认端口是80，`https`默认端口是443

`tcp`数据部分用来存放整个`http`报文，如图所示：

{{% figure class="center" src="/img/net/41.png" alt="img" %}}

# 网络层IP协议

传输层封装`tcp`报文接下来的工作就交给网络层，在这一层会封装`IP`数据包，`IP`数据包的头部如图所示

{{% figure class="center" src="/img/net/42.png" alt="img" %}}

字段简要解释[^1]

[^1]:http://www.tcpipguide.com/free/t_IPDatagramGeneralFormat.htm

- Version：IP的版本，ipv4，ipv6
- Internet Header Length (IHL)：IP 数据包的头部长度
- Type Of Service (TOS)：服务类型类型
- Total Length (TL)：IP数据报的总长度，最大为`65535`字节
- Identification：标识
- Flags：标志位
- Fragment Offset：当消息发生分片时，该字段指定该分片中的数据在整个消息中的偏移量或位置
- Time To Live (TTL): 指定数据报允许在网络上“生存”多长时间，以路由器跳数为单位。每个路由器在传输之前将 TTL 字段的值递减（减一）。如果 TTL 字段下降到零，则认为数据报经过了太长的路由并被丢弃
- Protocol：协议，表示，是tcp还是udp等等
- Header Checksum：在标头上计算的校验和，以提供基本保护以防止传输中的损坏
- Source Address：源ip地址
- Destination Address：目的ip地址
- Options：选项，在某些 IP 数据报的标准报头之后，可以包含几种类型的选项中的一种或多种
- Padding：填充，如果包含一个或多个选项，并且用于它们的位数不是 32 的倍数，则添加足够的零位以将标头“填充”为 32 位（4 字节）的倍数
- Data：数据，要在数据报中传输的数据，可以是整个高层消息，也可以是消息的片段。

`tcp`整个数据将填充于`IP`协议的`Data`中,如图所示
{{% figure class="center" src="/img/net/43.png" alt="img" %}}

# 以太网II

`Ethernet II`数据链路层

事实上只有`iP`是无法到达目的主机的，这是因为最终应用层交付的数据是由物理层面来接手。`ip`地址只是逻辑地址，真正地址是设备的`mac`地址，所谓的`mac`地址就是物理设备的"指纹"。整条链路下来如同套娃一样，一层层嵌套。以太网数据帧如图所示[^2]：

[^2]:https://en.wikipedia.org/wiki/Ethernet_frame

{{% figure class="center" src="/img/net/44.png" alt="img" %}}

{{% figure class="center" src="/img/net/45.png" alt="img" %}}

当`http`协议的数据被封装到链路层的数据包准备发送时，需要知道两个很重要的参数`Destination MAC Address` 和`Source MAC Address`。机器的操作系统一启动就会将网卡的`mac`地址写入内存，所以很容易知道自己的`mac`地址，而对方的`mac`地址则需要`ARP`协议来确定。

## ARP协议

全称:`Address Resolution Protocol`,是一种`ip`地址到`mac`地址的映射，已知目标`ip`查询所对应的`mac`地址。

如果目标主机的`ip`地址不在一个链路，那么会查找一下跳路由器的`mac`地址。如何知道对方的`mac`地址呢，很容易想到的方案就是广播，情况分为两种。

1. 两台主机在同一网段，`arp`将以广播的方式进行请求，如图所示：

{{% figure class="center" src="/img/net/46.png" alt="img" %}}

a想要发送数据包给b，比如`http`请求`tcp`建立连接的第一次握手`syn`包，a会检查自己的`arp`本地缓存(`arp -a`)，如果目的`ip`地址命中缓存就直接取出对应的`mac`地址，如果没有命中，那么a通过`arp`协议向局域网中广播`192.168.61.191`的`mac`地址是什么，局域网中的每个主机收到这个`arp`数据包后检查自己的`ip`地址和询问的`ip`地址是否匹配，如果匹配则返回自己的`mac`地址，如果不是则丢弃。这时候a就知道目的主机的`mac`地址了进而构造出L2[^4]的`header`,通过`switch`转发到目的主机。如果浏览器的网址是访问一个局域网中的服务器，那么到这里客户端的`syn`包已经发送到服务器端了，服务端接收到这个`L2`的数据包后，又层层解开封装，如同一层层剥开洋葱一样，直到`tcp`的包暴露出来，这时服务器才知道客户端想建立连接，然后就是三次握手，握手后客户端开始发送`http`协议的内容，服务端收到后将网页的内容返回回去，然后就是浏览器渲染`html`文本，进而用户看见了网页。

[^4]:L2 是`Layer 2`的缩写，指的是数据链路层，以此类推，L3是指网络层即ip层

2. 两台主机不在同一网段，此时就需要`Proxy ARP`[^5]。整个过程还需要路由器参与将数据包送到不同的网段。通过子网掩码可以判断两个`ip`是不是同一个网段。如图所示，每个路由器之间`mac`地址都会被剥离并且生成新的目的`mac`地址使其到达下一跳。第一个主机的生成的`ip`数据包只会被接收的主机剥离。因此`ip`数据包的头部是“端到端“的处理，而数据链路的数据包头部是”跳到跳“处理

   {{% figure class="center" src="/img/net/47.png" alt="img" %}}

[^5]:https://www.practicalnetworking.net/series/arp/proxy-arp/

# 总结

当浏览器发出页面请求的时候，就会层层封装消息，从应用层到物理层，到了服务器又会层层解封，最终解出http协议并将目录下的网页文件内容发送给客户端这个过程又涉及层层封装消息，客户端收到后层层解出得到网页的`html`文本，浏览器渲染，最终生成用户看见的页面。如动图所示[^6]：

{{% figure class="center" src="/img/net/48.gif" alt="img" %}}

[^6]:https://www.practicalnetworking.net/series/packet-traveling/osi-model/

