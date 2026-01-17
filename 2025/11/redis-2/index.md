# redis search


<!--more-->

## k8s部署
``` yaml 
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  annotations:
    analysis.crane.io/resource-recommendation: |
      containers:
      - containerName: redisearch
        target:
          cpu: 125m
          memory: 125Mi
  labels:
    k8s-app: redisearch
  name: redisearch
  namespace: cache
  resourceVersion: '61066461593'
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Retain
  podManagementPolicy: OrderedReady
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: redisearch
  serviceName: ''
  template:
    metadata:
      annotations:
        kubectl.kubernetes.io/restartedAt: '2025-11-13T13:14:22+08:00'
      creationTimestamp: null
      labels:
        k8s-app: redisearch
    spec:
      containers:
        - args:
            - /usr/local/etc/redis/redis.conf
          command:
            - redis-stack-server
          image: 'redis/redis-stack-server:latest'
          imagePullPolicy: Always
          name: redisearch
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          volumeMounts:
            - mountPath: /usr/local/etc
              name: cache
            - mountPath: /var/lib/redis-stack
              name: redisearch-data
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
        - configMap:
            defaultMode: 420
            items:
              - key: redis.conf
                path: redis/redis.conf
            name: redisearch-config
          name: cache
        - name: redisearch-data
          persistentVolumeClaim:
            claimName: redisearch-pvc
  updateStrategy:
    rollingUpdate:
      partition: 0
    type: RollingUpdate

---
apiVersion: v1
kind: Service
metadata:
  annotations: {}
  labels:
    k8s-app: redisearch
  name: redisearch
  namespace: cache
  resourceVersion: '61066444775'
spec:
  clusterIP: 10.16.254.133
  clusterIPs:
    - 10.16.254.133
  internalTrafficPolicy: Cluster
  ipFamilies:
    - IPv4
  ipFamilyPolicy: SingleStack
  ports:
    - port: 6379
      protocol: TCP
      targetPort: 6379
  selector:
    k8s-app: redisearch
  sessionAffinity: None
  type: ClusterIP

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    k8s.kuboard.cn/pvcType: Dynamic
    pv.kubernetes.io/bind-completed: 'yes'
    pv.kubernetes.io/bound-by-controller: 'yes'
    volume.beta.kubernetes.io/storage-provisioner: com.tencent.cloud.csi.cbs
    volume.kubernetes.io/storage-provisioner: com.tencent.cloud.csi.cbs
  finalizers:
    - kubernetes.io/pvc-protection
  name: redisearch-pvc
  namespace: cache
  resourceVersion: '61066163021'
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
  storageClassName: cbs
  volumeMode: Filesystem
  volumeName: pvc-24fbe547-cb2a-4ca1-9d82-d8937602f7dc
status:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 50Gi
  phase: Bound



---
apiVersion: v1
data:
  redis.conf: |
    bind 0.0.0.0
    protected-mode yes
    port 6379
    requirepass 1234567
    appendonly yes
    dir /data
kind: ConfigMap
metadata:
  name: redisearch-config
  namespace: cache
  resourceVersion: '61066351780'


```


## 语法
### 建立索引
``` bash
FT.CREATE idx:games ON HASH PREFIX 1 game: SCHEMA body TEXT
```

### 插入数据
``` bash
HSET game:1 body "hpjygf 和平精英 hepingjingying 新世纪福音战士 xinshijifuyinzhanshi hpjy battlegrounds 和平精英国服 hepingjingyingguofu hpjygf"
```

### 查询数据
``` bash 
FT.SEARCH idx:games "*和*"
```
