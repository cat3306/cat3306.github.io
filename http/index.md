# http 协议


<!--more-->

# HTTP 

`HTTP` 全称`HyperText Transfer Protocol`，是一种基于`TCP`的超文本传输协议。所谓的协议通俗的讲就是制定一系列规则，要想沟通就必须遵守这些规则。稍微熟悉网络协议栈的朋友都知道`HTTP`协议位于应用层

{{% figure class="center" src="/img/net/49.png" alt="img" %}}

一个完整的`HTTP`报文长这个样子，请求的数据格式和响应的数据格式都是这样。

{{% figure class="center" src="/img/net/34.png" alt="img" %}}

{{< admonition tip >}}

`TCP` 协议并没有数据包的概念，传输的二进制流（binary stream），所以基于`TCP`的应用层协议都需要自己处理数据边界。HTTP数据格式的那些分隔符就是为了确定信息边界的。例如一个简易的游戏服务，数据包是基于`TCP`设计的，那么它的消息体有可能长这个样子

{{% figure class="center" src="/img/net/50.png" alt="img" %}}

{{< /admonition >}}

## HTTP 状态码

状态码比较多，如果忘记可随时[查阅](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#information_responses)

### 100 - 199



### 200` – `299

- `200 OK`

### 300` – `399

### 400` – `499

- `401 Unauthorized`

- `403 Forbidden`
- `404 Not Found`
- `405 Method Not Allowed`

### 500` – `599

- `500 Internal Server Error`
- `502 Bad Gateway`

## HTTP 请求方法

### GET

get 请求并不建议带有`body` 

- 幂等

### POST

### HEAD

### PUT

### DELETE

### CONNECT



### OPTIONS

### TRACE

### PATCH



## HTTP 数据报文

http协议主要分为三个部分

1. 起始行(start line)
2. 头部字段(header) key-value 结构
3. 消息主体(entity) 实际传输的的数据，也称为body。可能是纯文本，图片，二进制流，视频等



## HTTP 请求过程



## HTTPS

由于http协议都是明文传输，并不安全，容易遭到劫持和修改。所以为数据安全用 http+SSL(安全套接层)和TLS(安全传输协议) 就变成https。协议层次图

![](/img/net/5.png "工作流程")}

### 加密方式

- 对称加密

加密端和解密端持有同一份秘钥
![](/img/net/6.png "工作流程")}

- 非对称加密

非对称算法需要两个秘钥，公钥(public key)，私钥(private key)。公钥和私钥是成对的。用公钥加密的密文只有对应的私钥能解开，私钥加密对应的公钥才能解开。非对称加密解决了对称加密的一个问题就是，秘钥传输本身不是安全的。
![](/img/net/7.png "工作流程")}

在https 这两种加密方式都采用了，为什么呢，http的报文可能很大，如果用非对称加密，性能就很差。所以http报文采用对称加密来加密的。
![](/img/net/8.png "工作流程")}

### 数字证书认证

由于公钥的下发是不安全的，可能会被劫持篡改，所以需要三方介入，这个三方就是CA。CA是一个数字证书认证机构，一个客户端和服务器都信任的机构。该机构负责数字签名服务器的上报的公钥。
数字签名：一种认证机制。
HTTPS 数字认证的流程如图
![](/img/net/9.png "工作流程")}

### 握手过程

![](/img/net/10.png "工作流程")}

- Client Hello:握手第一步是客户端向服务端发送 Client Hello 消息，这个消息里包含了一个客户端生成的随机数 Random1、客户端支持的加密套件（Support Ciphers）和 SSL Version 等信息
- Server Hello:第二步是服务端向客户端发送 Server Hello 消息，这个消息会从 Client Hello 传过来的 Support Ciphers 里确定一份加密套件，这个套件决定了后续加密和生成摘要时具体使用哪些算法，另外还会生成一份随机数 Random2。注意，至此客户端和服务端都拥有了两个随机数（Random1+ Random2），这两个随机数会在后续生成对称秘钥时用到。
- Certificate:这一步是服务端将自己的证书下发给客户端，让客户端验证自己的身份，客户端验证通过后取出证书中的公钥
- Server Hello Done:通知客户端 Server Hello 过程结束。
- Certificate Verify:客户端收到服务端传来的证书后，先从 CA 验证该证书的合法性，验证通过后取出证书中的服务端公钥，再生成一个随机数 Random3，再用服务端公钥非对称加密 Random3生成 PreMaster Key
- Client Key Exchange:上面客户端根据服务器传来的公钥生成了 PreMaster Key，Client Key Exchange 就是将这个 key 传给服务端，服务端再用自己的私钥解出这个 PreMaster Key 得到客户端生成的 Random3。至此，客户端和服务端都拥有 Random1 + Random2 + Random3，两边再根据同样的算法就可以生成一份秘钥，握手结束后的应用层数据都是使用这个秘钥进行对称加密。为什么要使用三个随机数呢？这是因为 SSL/TLS 握手过程的数据都是明文传输的，并且多个随机数种子来生成秘钥不容易被暴力破解出来。
- Change Cipher Spec(Client):这一步是客户端通知服务端后面再发送的消息都会使用前面协商出来的秘钥加密了，是一条事件消息
- Encrypted Handshake Message(Client):这一步对应的是 Client Finish 消息，客户端将前面的握手消息生成摘要再用协商好的秘钥加密，这是客户端发出的第一条加密消息。服务端接收后会用秘钥解密，能解出来说明前面协商出来的秘钥是一致的
- Change Cipher Spec(Server):这一步是服务端通知客户端后面再发送的消息都会使用加密，也是一条事件消息
- Encrypted Handshake Message(Server):这一步对应的是 Server Finish 消息，服务端也会将握手过程的消息生成摘要再用秘钥加密，这是服务端发出的第一条加密消息。客户端接收后会用秘钥解密，能解出来说明协商的秘钥是一致的。
- Application Data:到这里，双方已安全地协商出了同一份秘钥，所有的应用层数据都会用这个秘钥加密后再通过 TCP 进行可靠传输



 




