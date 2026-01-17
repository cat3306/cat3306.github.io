# K8s 之Ingress-nginx 读取客户端原始ip


<!--more-->
## 问题描述
在集群中部署了Ingress-nginx-controller 。整条链路 clb ->nginx-controller -> backend-service 数据流动是正常，但是在后端服务中发现，客户端的原始ip地址并没有正确透传到后端服务中，透传的是一个内网地址类似 10.8.64.38

## 尝试解决
首先看nginx-controller 的配置。配置了use-forwarded-headers: true，不应该啊
{{< image src="/images/k8s/2.jpg" width="100%" caption="图片描述" >}}

然后一通排查后最后要改service 的配置
k8s文档![https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/create-external-load-balancer/]

{{< image src="/images/k8s/3.jpg" width="100%" caption="图片描述" >}}

把这个改成 `externalTrafficPolicy: Local` 后就可以了

```yaml
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.cloud.tencent.com/client-token: *
    service.kubernetes.io/loadbalance-id: *
    service.kubernetes.io/service.extensiveParameters: >-
      {"AddressIPVersion":"IPV4","InternetAccessible":{"InternetChargeType":"TRAFFIC_POSTPAID_BY_HOUR","InternetMaxBandwidthOut":523}}
  finalizers:
    - service.k8s.tencent/resources
  labels:
    k8s-app: systemsrv-ingress-nginx-controller
    qcloud-app: systemsrv-ingress-nginx-controller
  name: systemsrv-ingress-nginx-controller
  namespace: raven
  resourceVersion: '61130952591'
spec:
  allocateLoadBalancerNodePorts: true
  clusterIP: 10.16.254.223
  clusterIPs:
    - 10.16.254.223
  externalTrafficPolicy: Local
  healthCheckNodePort: 32746
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - name: http
      nodePort: 31328
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      nodePort: 31112
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    k8s-app: systemsrv-ingress-nginx-controller
    qcloud-app: systemsrv-ingress-nginx-controller
  sessionAffinity: None
  type: LoadBalancer


```
