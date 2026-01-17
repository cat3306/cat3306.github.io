# 由k8s调度pod的问题引发的事故


<!--more-->
## 问题
1. 起因是2025-11-08 下午/v2/account/mobile/pc_product/list 这个接口需要白名单访问，接口验token的配置在 gateway.http2rpc_meta 的need_auth_token 字段，改完配置需要重启apigateway 才能生效。然后重启了apigateway后线上开始卡顿现象，持续一个多小时。当时集群的负载并不高，apigateway ，业务服务account 和 booster都没有异常的日志。采取了降频心跳等措施，恢复正常，但是原因未知

2. 2025-11-12 上午调整apigateway 的deployment参数（腾讯提供建议），触发重启，又复现了2025-11-08 下午的问题，登录业务开始卡顿，一通排查后最终发现了超时日志，才锁定了最终的问题点
      grep '2025/11/12' /var/log/nginx/nginx_error.log | grep 'Operation timed out' | grep 'login' | grep '116.233.12.26'
   在nginx-Ingress-controller 的报错日志中，陆陆续续会有超时的记录。

   ``` text
   530#530: *87054449 connect() failed (110: Operation timed out) while connecting to upstream, client: 223.104.69.241, server: ****api.com, request: "POST /v2/account/get/user/state?sig=1b0fc3cc71e4bdf5d723cfd74d0d83b07d36a9d95dba7ac5a1d305e87b6c74ee&nonce=5fc68aea995546048c4b6db3aad19a9f&ts=1762998222&ver=2023-08-28 HTTP/1.1", upstream: "http://10.16.0.23:8080/v2/account/get/user/state?sig=1b0fc3cc71e4bdf5d723cfd74d0d83b07d36a9d95dba7ac5a1d305e87b6c74ee&nonce=5fc68aea995546048c4b6db3aad19a9f&ts=1762998222&ver=2023-08-28", host: "****api.com"
   ```
   下游10.16.1.209:8080 访问不通，导致客户端请求接口卡顿。然后发现apigateway 有一个pod 被调度到 nginx-ingress 的机器上去了。当请求打到这个调度错误的pod上就会超时

## 解决的办法
将原来的3个pod 调整成两个，强行让k8s重新调度pod到红色这三个节点的其中两个，只要pod调度进了 nginx-ingress-np 这个节点池（蓝色框起来的），就会有问题。

{{< image src="/images/k8s/1.png" width="100%" caption="图片描述" >}}
线上马上恢复正常了

## 后续
调研下k8s的调度机制，以及为啥有概率会被调度进nginx-ingress-np 节点池

强制调度策略
1. kubectl label node xyz.fun.prod1  dedicated=xyz-business 给节点打标签
2. Deployment 配置
  ```yaml
  template:
    metadata:
      labels:
        app: myapp
    spec:
      nodeSelector:
        dedicated: xyz-business
      containers:
        - name: myapp
          image: myapp:latest
  ```
