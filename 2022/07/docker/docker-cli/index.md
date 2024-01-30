# docker cli


<!--more-->

# [doc](https://docs.docker.com/reference/)

# docker attach

`docker attach`容器的 ID 或名称将终端的标准输入、输出和错误（或三者的任意组合）附加到正在运行的容器

```bash
$ docker run -d --name topdemo ubuntu /usr/bin/top -t

$ docker attach topdemo
top - 13:13:27 up 8 days,  5:16,  0 users,  load average: 0.04, 0.07, 0.03
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7838.6 total,   2394.0 free,   1615.3 used,   3829.4 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5758.8 avail Mem


```

# docker build



# docker run 



运行容器的命令

## -d, --detach

`detach`直译为脱离，分离模式启动容器，后台运行

下面例子只关心 `-d`这个参数

```bash
$ docker run --name some-nginx -d -p 8080:80 nginx
```

## -a

可以指定三种标准输出 `STDIN`, `STDOUT` ,`STDERR`

```bash
$ docker run -a stdin -a stdout -i -t ubuntu /bin/bash
```

## -i -t

--interactive Keep STDIN open even if not attached

--tty Allocate a pseudo-TTY

交互模式，可以进入容器内部

```bash
$ docker run  -i -t ubuntu /bin/bash
```

```bash
$ echo test | docker run -i ubuntu cat
```

## --name 

给容器取名字，下面例子中`some-nginx`是这个容器的名字

```bash
$ docker run --name some-nginx -d -p 8080:80 nginx

$ docker ps -f 'name=some-nginx'

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                   NAMES
028c1ac0cf08   nginx     "/docker-entrypoint.…"   17 minutes ago   Up 17 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   some-nginx
```

唯一确定一个容器，有三种方式



| **Identifier type**   | **Example value**                                            |
| --------------------- | ------------------------------------------------------------ |
| UUID short identifier | f78375b1c487                                                 |
| UUID long identifier  | f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778 |
| Name                  | myredis                                                      |

## --cidfile string

docker 将容器 ID 写入指定的文件中。这类似于某些程序将其进程 ID (`PID`)写入文件

## --restart

重启策略

- **no** :当容器退出后，不自动重启（默认）
- **on-failure**：[max-retries]:只有容器以非0的状态退出(异常退出)才重启，（可选）限制 Docker 守护程序尝试重新启动的次数
- **always**：无论退出状态如何，始终重新启动容器。`docker`无限制地重启容器，容器也将始终在守护程序启动时启动
- **unless-stopped**：无论退出状态如何，始终重新启动容器，包括docker的守护进程重启，除非容器自己处于stopped state



````bash
$ docker run --name mysql -v ~/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -d -p 3307:3306 --restart=always  mysql
````



```bash
# 获取重启次数
$ docker inspect -f "{{ .RestartCount }}" my-container


# 获取重启时间
$ docker inspect -f "{{ .State.StartedAt }}" my-container
```

## --network

网络模式



## -- link

容器互联

````
docker run --rm   --name gameserver --link mysql-mysql-1:db --net mysql_default cat19/gameserver
````

参数`--link name:alias` name 是连接的容器名称，alias为别名

{{< admonition  >}}
如果连接的容器是`docker-compose`启动的，需要指定`--net mysql_default `，否则会抛出错误

docker: Error response from daemon: Cannot link to /mysql-mysql-1, as it does not belong to the default network.

{{< /admonition >}}

## --add-host

向容器的`/etc/hosts` 添加映射

```bash
1221$ docker run --rm -it --add-host db-static:86.75.30.9 ubuntu cat /etc/hosts
```







# docker ps

```bash
$ docker ps --help
usage:  docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes
```

### docker ps -a 

退出的`container`也会展示

```
$ docker ps -a
```

### docker ps -f  

```bash
$ docker ps -f 'name=redis'

CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS             PORTS                                       NAMES
ab0141a96725   redis     "docker-entrypoint.s…"   About an hour ago   Up About an hour   0.0.0.0:6379->6379/tcp, :::6379->6379/tcp   myredis
```

# docker container

# docker logs

### -f  

```bash
$ docker logs  -f 028c1ac0cf08

172.17.0.1 - - [10/Jul/2022:13:34:59 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0" "-"
192.168.0.103 - - [10/Jul/2022:13:35:41 +0000] "GET / HTTP/1.1" 200 615 "-" "Mozilla/5.0 
```



# docker search

### --filter , -f

根据提供的条件过滤输出

|     格式     |         描述          |
| :----------: | :-------------------: |
|    .Name     |       镜像名字        |
| .Description |       镜像描述        |
|  .StarCount  |     镜像的点赞数      |
| .IsOfficial  |    OK:镜像是官方的    |
| .IsAutomated | OK:镜像构建是自动化的 |



```bash
$ docker search --format "{{.Name}}: {{.StarCount}}" nginx
nginx: 17077
linuxserver/nginx: 169
bitnami/nginx: 135
ubuntu/nginx: 52
bitnami/nginx-ingress-controller: 19
rancher/nginx-ingress-controller: 10
clearlinux/nginx: 4
ibmcom/nginx-ingress-controller: 4
bitnami/nginx-ldap-auth-daemon: 3
bitnami/nginx-exporter: 2
rapidfort/nginx: 2
rancher/nginx: 2
rancher/nginx-ingress-controller-defaultbackend: 2
vmware/nginx: 2
```

# docker compose

新版的Compose v2的命令已经 嵌入`docker cli `，可以用`docker compse` 来代替 `docker compse`

## -p 

指定项目的名字

例如有以下docker-compose.yml

````yaml
services:
  MYSQLDB:
    image: mysql
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    container_name: MYSQLDB
    command: [ 'mysqld', '--character-set-server=utf8mb4', '--collation-server=utf8mb4_unicode_ci','--default-time-zone=+08:00' ]
    ports:
      - 3309:3306
    environment:
      - MYSQL_ROOT_PASSWORD=12345678
      - TZ=Asia/Shanghai
  gameserver:
    image: cat19/gameserver
    ports:
      - 8840:8840
    restart: always
    container_name: GameServer
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    links:
      - MYSQLDB:MysqlDb
volumes:
  db_data
````



```bash
$ docker compose -p gameserver up -d
```

# docker volume

备份

```
```






