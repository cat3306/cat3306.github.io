# docker guide


<!--more-->

本篇是按照[官方文档](https://docs.docker.com/)走一遍

# 概述

`docker`是一个用于开发，部署，运行的开放平台，可以为开发者虚拟出独立的沙箱环境用于程序运行。[^1]

[^1]:https://docs.docker.com/get-started/overview/#the-docker-platform

## Docker的架构

`docker`是典型的的C/S架构

{{% figure class="center" src="/img/docker/1.png" alt="img" %}}

## Docker daemon

`docker`后台进程(`dockerd`)，接收docker命令，管理`images`，`containers`,`networks`,`volumes` 。dockerd和dockerd之间也可有通信。

## Docker client

docker的客户端，是用户与`dockerd`的交互方式。例如` docker run -d -p 3001:3001 getting-started`，客户端会将这些命令发送到`dockerd`，由`dockerd`执行。`docker` 客户端可以与多个`dockerd`通信

## Docker Desktop

集成了`dockerd`,`docker`,`docker compose`,`docker content trust`,`kubernetes`

## Docker registries

存储大量`image`的地方

## Images

docker镜像

## Containers

容器，是镜像的实例

# CLI

## `docker version`

| 参数         | 例如           | 描述               |
| ------------ | -------------- | ------------------ |
| --format ,-f | docker version | 格式化输出版本信息 |
|              |                |                    |

例如

- `docker version`

```

Client:
 Cloud integration: v1.0.23
 Version:           20.10.14
 API version:       1.41
 Go version:        go1.16.15
 Git commit:        a224086
 Built:             Thu Mar 24 01:49:20 2022
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true

Server: Docker Desktop 4.7.1 (77678)
 Engine:
  Version:          20.10.14
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.15
  Git commit:       87a90dc
  Built:            Thu Mar 24 01:46:14 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.5.11
  GitCommit:        3df54a852345ae127d1fa3092b95168e4a88e2f8
 runc:
  Version:          1.0.3
  GitCommit:        v1.0.3-0-gf46b6ba
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

- `docker version -f '{{json .}}'`

```js
{
  "Client": {
    "Platform": {
      "Name": ""
    },
    "CloudIntegration": "v1.0.23",
    "Version": "20.10.14",
    "ApiVersion": "1.41",
    "DefaultAPIVersion": "1.41",
    "GitCommit": "a224086",
    "GoVersion": "go1.16.15",
    "Os": "darwin",
    "Arch": "amd64",
    "BuildTime": "Thu Mar 24 01:49:20 2022",
    "Context": "default",
    "Experimental": true
  },
  "Server": {
    "Platform": {
      "Name": "Docker Desktop 4.7.1 (77678)"
    },
    "Components": [
      {
        "Name": "Engine",
        "Version": "20.10.14",
        "Details": {
          "ApiVersion": "1.41",
          "Arch": "amd64",
          "BuildTime": "Thu Mar 24 01:46:14 2022",
          "Experimental": "false",
          "GitCommit": "87a90dc",
          "GoVersion": "go1.16.15",
          "KernelVersion": "5.10.104-linuxkit",
          "MinAPIVersion": "1.12",
          "Os": "linux"
        }
      },
      {
        "Name": "containerd",
        "Version": "1.5.11",
        "Details": {
          "GitCommit": "3df54a852345ae127d1fa3092b95168e4a88e2f8"
        }
      },
      {
        "Name": "runc",
        "Version": "1.0.3",
        "Details": {
          "GitCommit": "v1.0.3-0-gf46b6ba"
        }
      },
      {
        "Name": "docker-init",
        "Version": "0.19.0",
        "Details": {
          "GitCommit": "de40ad0"
        }
      }
    ],
    "Version": "20.10.14",
    "ApiVersion": "1.41",
    "MinAPIVersion": "1.12",
    "GitCommit": "87a90dc",
    "GoVersion": "go1.16.15",
    "Os": "linux",
    "Arch": "amd64",
    "KernelVersion": "5.10.104-linuxkit",
    "BuildTime": "2022-03-24T01:46:14.000000000+00:00"
  }
}
```

- `docker version --format '{{.Server.Version}}'`

```
20.10.14
```

## `docker run`

例子

### `docker run --name test -it debian`

```bash
$ docker run --name test -it debian
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
6aefca2dc61d: Already exists
Digest: sha256:6846593d7d8613e5dcc68c8f7d8b8e3179c7f3397b84a47c5b2ce989ef1075a0
Status: Downloaded newer image for debian:latest

exit 13
echo $?
13
$ docker ps -a | grep test
a30b8390fce8   debian                   "bash"                   2 minutes ago       Exited (13) About a minute ago
```

如果本地没有`debian`镜像，则去拉取最新的`debian:latest` ,-i 是交互模式，-t后台运行容器，--name取名字，`echo $?`：打印出刚才退出去的进程退出码

### `docker run --cidfile /tmp/docker_test.cid ubuntu echo "test"`

```bash
$ docker run --cidfile /tmp/docker_test.cid ubuntu echo "test"
test
```

创建一个容器并打印`test`到控制台。`cidfile` 尝试创建一个新文件并将容器 ID 写入其中，如果文件已经存在返回错误，命令结束时关闭此文件。

在宿主机上

```bash
$ cat /tmp/docker_test.cid
759876e15d829ae04bd91a81410959be8630efcbb68a29dba344d105e87522d3
```

### --privileged

赋予最高权限

```bash
$ docker run -t -i --rm ubuntu bash
root@e700a5cb0af4:/#
root@e700a5cb0af4:/# mount -t tmpfs none /mnt
mount: /mnt: permission denied.
```

```bash
$ docker run -t -i --privileged ubuntu bash
root@f66cf697d0d1:/#
root@f66cf697d0d1:/# mount -t tmpfs none /mnt
root@f66cf697d0d1:/#
```

参数--privileged，可以越过很多限制，意味着容器几乎可以做主机可以做的所有事情。该参数用于特殊用例，例如在 Docker 中运行 Docker

### 设置工作目录（-w）

```bash
$ docker  run -w /path/to/dir/ -i -t  ubuntu pwd
/path/to/dir
```

`-w`让命令在给定的目录中执行， 这里是`/path/to/dir/`。如果路径不存在，则在容器内创建。

```bash
$ docker  run -w /path/to/dir/ -i -t  ubuntu
root@83958f63bad9:/path/to/dir#
```



### 挂载 tmpfs (--tmpfs) 

```bash
$ docker run -d --tmpfs /run:rw,noexec,nosuid,size=65536k my_image
```

### 挂载卷（-v，--read-only）



```bash
docker run \
   -p 9000:9000 \
   -p 9090:9090 \
   --name minio \
   -v ~/Desktop/testminio:/data \
   -e "MINIO_ROOT_USER=ROOTNAME" \
   -e "MINIO_ROOT_PASSWORD=CHANGEME123" \
   -d \
   minio/minio server /data --console-address ":9090"
```

-p ：端口映射，host_port:local_port

--name：容器的名字

-v：挂载

-d：后台运行

-e：设置环境变量，`MINIO_ROOT_USER`和`MINIO_ROOT_PASSWORD`。



```bash
$ docker  run  -v `pwd`:`pwd` -w `pwd` -i -t  ubuntu pwd
```

`-v`参数将当前工作目录挂载到容器中。`-w` 让命令在当前工作目录中执行，最后在容器中执行`pwd`



```bash
$ docker  run  -v `pwd`:`pwd` -w `pwd` -i -t  ubuntu
root@be74d2cdbc61:/Users/joker#
root@be74d2cdbc61:/Users/joker# ls
1.mpeg   Documents  Library  Music     Postman  blog  java_error_in_goland_52033.log  software_pkg
Desktop  Downloads  Movies   Pictures  Public   code  software
```

很明显，进入容器后，访问的是是宿主机的家目录。

```bash
$ docker run --read-only -v /icanwrite busybox touch /icanwrite/here
```

### --mount

```bash
$ docker run -t -i --mount type=bind,src=/data,dst=/data busybox sh
```

# docker-compose

Compose is a tool for defining and running multi-container Docker applications

```bash
$ docker-compose -h

Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [--profile <name>...] [options] [--] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file
                              (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name
                              (default: directory name)
  --profile NAME              Specify a profile to enable
  -c, --context NAME          Specify a context name
  --verbose                   Show more output
  --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  --ansi (never|always|auto)  Control when to print ANSI control characters
  --no-ansi                   Do not print ANSI control characters (DEPRECATED)
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the
                              name specified in the client certificate
  --project-directory PATH    Specify an alternate working directory
                              (default: the path of the Compose file)
  --compatibility             If set, Compose will attempt to convert keys
                              in v3 files to their non-Swarm equivalent (DEPRECATED)
  --env-file PATH             Specify an alternate environment file

Commands:
  build              Build or rebuild services
  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove resources
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show version information and quit
```

```bash
# 后台运行
$ docker-compose up -d
```

## docker-compose.yml

`docker-compose` 读取这个配置文件来运行`containers`

例如`minio`

```yaml
version: '3.7'

# Settings and configurations that are common for all containers
x-minio-common: &minio-common
  image: quay.io/minio/minio:RELEASE.2022-06-03T01-40-53Z
  command: server --console-address ":9001" http://minio{1...4}/data{1...2}
  expose:
    - "9000"
    - "9001"
  # environment:
    # MINIO_ROOT_USER: minioadmin
    # MINIO_ROOT_PASSWORD: minioadmin
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
    interval: 30s
    timeout: 20s
    retries: 3

# starts 4 docker containers running minio server instances.
# using nginx reverse proxy, load balancing, you can access
# it through port 9000.
services:
  minio1:
    <<: *minio-common
    hostname: minio1
    volumes:
      - data1-1:/data1
      - data1-2:/data2

  minio2:
    <<: *minio-common
    hostname: minio2
    volumes:
      - data2-1:/data1
      - data2-2:/data2

  minio3:
    <<: *minio-common
    hostname: minio3
    volumes:
      - data3-1:/data1
      - data3-2:/data2

  minio4:
    <<: *minio-common
    hostname: minio4
    volumes:
      - data4-1:/data1
      - data4-2:/data2

  nginx:
    image: nginx:1.19.2-alpine
    hostname: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "9000:9000"
      - "9001:9001"
    depends_on:
      - minio1
      - minio2
      - minio3
      - minio4

## By default this config uses default local driver,
## For custom volumes replace with volume driver configuration.
volumes:
  data1-1:
  data1-2:
  data2-1:
  data2-2:
  data3-1:
  data3-2:
  data4-1:
  data4-2:
```


