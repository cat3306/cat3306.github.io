# minio command


<!--more-->

# 官网

[minio](https://min.io/) 



# minio server

```shell
#单点部署。--console-address 指定固定端口，~/Desktop/testminio 挂载点
$ minio server ~/Desktop/testminio --console-address ":9001"
```

````bash
sudo docker run \
   -p 9000:9000 \
   -p 7070:9090 \
   --name minio \
   --restart=always \
   -v /Users/joker/docker/minio/data:/data \
   -e "MINIO_ROOT_USER=cat13" \
   -e "MINIO_ROOT_PASSWORD=cat12345" \
   -e "TZ=Asia/Shanghai" \
   -d \
   quay.io/minio/minio server /data --console-address ":9090"
````




