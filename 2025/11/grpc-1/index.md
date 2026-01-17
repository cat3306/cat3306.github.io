# grpc


<!--more-->
 ## 记一次线上的错误日志
 ### 问题描述
 在周六的下午高峰期，因为有个接口要加白不需要token也能访问。当时在数据库改了接口的配置。就重启业务网关的服务apigateway。
 飞书的报警群就弹出了grpc服务超时的调用。
 `rpc error: code = DeadlineExceeded desc = received context error while waiting for new LB policy update: context deadline exceeded`
  
还有一条更加明确
 `rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial tcp 10.16.253.251:8081: i/o timeout"`
第一反应是重启导致的

为啥会有这两种错误呢，一是因为k8s中DNS的更新是需要时间的，二是原来的client的参数中没有keepalive
两个问题
- 网关的服务已经重启，但是作为下游的服务还是持有旧的tcp连接，导致超时。
- 网关重启后，DNS解析还未更新

### 解决
`grpc client` 的配置
``` go 
err := dispatcher.NewClient("dns:///apigateway:8081",
		ggrpc.WithKeepaliveParams(keepalive.ClientParameters{
			Time:                10 * time.Second,
			Timeout:             2 * time.Second,
			PermitWithoutStream: true,
		}),
		ggrpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`),
	)
	if err != nil {
		panic(err)
	}
```

前缀 dns:/// 明确告诉 gRPC：使用 gRPC 内置的 DNS resolver周期性刷新 DNS（默认每 30 秒）可以获取 Service 下最新的 Endpoint 列表避免 client 缓存旧 IP 导致连接不可用

设置keepalive 可以断掉已经失效的连接，避免出现连接超时

设置默认的负载均衡策略为 round_robin，避免每次请求都打到同一个服务节点

然后就是deployment 的配置 设置lifecycle

Pod 被删除或重启时，Kubernetes 会先发送 SIGTERM 给容器。如果容器直接退出，客户端（比如 gRPC client）可能还在使用旧连接 → i/o timeout、请求失败。preStop 可以在容器收到 SIGTERM 后延迟执行操作，比如：sleep 3 给客户端时间感知 Pod 下线关闭 HTTP/gRPC server，先拒绝新请求，再处理现有请求防止服务被立即杀掉，提高请求的稳定性
``` yaml 
lifecycle:
    preStop:
      exec:
        command:
          - sleep
          - '3'
```

``` yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    analysis.crane.io/resource-recommendation: |
      containers:
      - containerName: account
        target:
          cpu: 125m
          memory: 125Mi
    app.gitlab.com/app: booster-server-raven
    app.gitlab.com/env: dev
    prometheus.io/port: '9090'
    prometheus.io/scrape: 'true'
  labels:
    app: account
    ref: dev
  name: account
  namespace: raven
  resourceVersion: '6066716413'
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: account
      ref: dev
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      annotations:
        kubectl.kubernetes.io/restartedAt: ''
        qcloud-redeploy-timestamp: ''
      creationTimestamp: null
      labels:
        app: account
        pod-template-hash: 6f79b965c6
        ref: dev
    spec:
      containers:
        - env:
            - name: PATH
              value: '/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin'
            - name: APOLLO_META
              value: 'http://cfg-apollo-configservice.apollo:8080'
            - name: APOLLO_ACCESS_KEY
              value: **************************
          image: '******************************'
          imagePullPolicy: Always
          lifecycle:
            preStop:
              exec:
                command:
                  - sleep
                  - '3'
          livenessProbe:
            failureThreshold: 3
            grpc:
              port: 8000
              service: ''
            periodSeconds: 10
            successThreshold: 1
            timeoutSeconds: 5
          name: account
          ports:
            - containerPort: 8000
              name: grpc
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            grpc:
              port: 8000
              service: ''
            periodSeconds: 20
            successThreshold: 1
            timeoutSeconds: 5
          resources: {}
          securityContext:
            privileged: false
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /etc/localtime
              name: localtime
              readOnly: true
      dnsPolicy: ClusterFirst
      imagePullSecrets:
        - name: **********
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - hostPath:
            path: /etc/localtime
            type: File
          name: localtime
---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    app: account
    ref: dev
  name: account
  namespace: raven
spec:
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - port: 8000
      protocol: TCP
      targetPort: 8000
  selector:
    app: account
    ref: dev
  sessionAffinity: None
  type: ClusterIP
  loadBalancer: {}
```
