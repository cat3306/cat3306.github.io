# 一个诡谲的线上问题


<!--more-->
## 1
一个线上问题从去年一直困扰到了今年，直到上个星期才接近真相。事情是什么呢，我和客户端约定了一个接口名为心跳接口。当时设计的时候，是希望通过心跳接口，可以做一些通知的逻辑(经典的短连接当成长连接用)，并且在返回参数中有个`interval` 字段可以实时调整心跳的间隔，如果服务器的负载很高可以把`interval` 调高一点来降低负载，就是这么一个机制出现很奇怪的现象。
## 2
随着业务的增长和用户的增多，服务器明显有点吃不消，出现了第一次事故，负责跑`Ingress nginx`的机器挂了1个，一共有3个节点。但是这两个节点不足以支撑当晚的流量，造成了连锁反应导致全站都挂了，因为网关挂掉了。流量全打在`clb`上没往下走了。后来复盘这个问题，我觉得心跳`10s`太重了，要调整间隔，之前预留的`interval` 字段就可以用起来了

## 3
去年的时候，当我把`interval` 改成 `15s`后，部署上线，诡谲的事情发生了，`Ingress nginx`的节点CPU的利用率以肉眼的速度迅速飙升，好家伙，把心跳间隔提高CPU不降反而上升了，这不对吧。我觉得是客户度逻辑问题，让客户端看看，客户端看了表示没啥问题。但是这个现象解释不通，最后也没有确切的解释，就没下文了

## 4
今年，又出现这个问题，又是一次线上事故，原因和去年一样，`Ingress nginx` 节点挂了，然后雪崩波及全站。不得不说`Ingress nginx` 节点的配置有点垃圾，4c4g，最要命的是，这个节点池的节点没办法单独升配置。也就是说抗不住了，直接横向扩节点。由之前的3个基点升到现在的5个，分析下来我认为还是心跳接口的问题，`10s`一次，如果十万人同时在线，就这一个接口qps就1万了。而且今年年初的时候我写长连接服务，可以做到实时消息通知，心跳接口是可以废掉了，但是心跳还绑定了其他的业务还动不了。于是我有想起了`interval` 这个字段，于是我改成了30s 上线了。在上线后的当天晚上，收到了报警，好家伙5个`Ingress nginx` 节点的cpu全打满了，再不干涉服务器又要大面积瘫痪了。我想到了什么把心跳间隔从30s改成10s，cpu的利用率立马降低了。真是奇怪。
## 5
最后没办法我吧客户端的代码看了一遍，没看出问题，但是我怀疑里面有递归调用。我这边`interval`不论改成什么参数。都会导致cpu升高，最后抓包才发现端倪，原来我一改时间间隔，客户端每次心跳调用都需要三次tcp握手和tls握手。在线上就是，每个人都在重连。人一多，那就很消耗CPU。
最后定位到客户端有一处`idleTimeout`写死了10s
``` dart
late final _dio = Dio()
    ..options = BaseOptions(
      baseUrl: _global.release ? ApiConfig.domain : ApiConfig.domainDev,
      responseType: ResponseType.bytes,
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 10),
      sendTimeout: const Duration(seconds: 10),
    )
    ..interceptors.addAll([
      BizInterceptor(),
      SignInterceptor(),
      LogInterceptor(
        requestBody: true,
        responseBody: false,
      )
    ])
    ..httpClientAdapter = Http2Adapter(
      ConnectionManager(
        idleTimeout: const Duration(seconds: 10),
        onClientCreate: (_, config) => config.onBadCertificate = (_) => true,
      ),
    );

```
``` go 
	return &account.PcHeartbeatResponse{
		Timestamp:    time.Now().Unix(),
		PauseState:   int64(pauseState),
		Notification: not,
		Interval:     10,
	}, nil
```
好吧原本想通过这个参数控制客户端的间隔，却被反向限制了。
