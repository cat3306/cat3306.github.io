# docker Compose


<!--more-->

# 简介



# 部署redis

单节点 不要配置文件

ymal文件

```yaml
version: '3.8'
services:
  cache:
    image: redis
    restart: always
    container_name: myredis
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning --requirepass 12345
    volumes:
      - cache:/data
volumes:
  cache:
    driver: local
```

自定义配置文件

```yaml
version: '3.8'
services:
  cache:
    image: redis
    restart: always
    container_name: myredis
    ports:
      - '6379:6379'
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - cache:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
volumes:
  cache:
    driver: local
```





```bash
$ docker-compose -f docker-compose-redis.yml up -d
```




