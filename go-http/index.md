# go http


<!--more-->

# http 协议

`http`协议全称`Hypertext Transfer Protocol`，位于整个网络层的应用层，也不难发现很多语言关于`http`的封装都是第三方框架实现的如`java` 的`spring`，也有的是标准库实现的如`go` 的`http`包，本篇的重点探究`go`的标准库`http`。



## 底层原理

作为应用层协议需要依赖传输层来进行。传输层用的最广泛的是`tcp`和`udp`，`HTTP /1.1`和 `HTTP /2`采用的`tcp`作为底层传输协议，而`HTTP /3`采用的是`QUIC`作为底层传输协议，如图[^1]

{{% figure class="center" src="/img/golang/1.png" alt="img" %}}

[^1]:https://draveness.me/golang/docs/part4-advanced/ch09-stdlib/golang-net-http/#921-%e8%ae%be%e8%ae%a1%e5%8e%9f%e7%90%86


