# redis


<!--more-->



 [官网](https://redis.io/download)

[仓库](https://github.com/redis/redis)

[在线体验](http://try.redis.io/)

[命令参考](http://doc.redisfans.com/)

# 安装

`make` 后 `src` 下会出现 `redis-server`和 `redis-cli`

```bash
$ wget https://download.redis.io/releases/redis-6.2.6.tar.gz
$ tar xzf redis-6.2.6.tar.gz
$ cd redis-6.2.6
$ make
$ cd src
$ ./redis-server ../redis.conf
$ ./redis-cli
```

### 配置

1. 直接改配置文件 `vim redis.conf`

   配置密码

   ```
   requirepass 123456
   ```

   配置守护进程

   ```
   daemonize yes
   ```

   配置外部访问

   ```
   注释 bind 127.0.0.1 -::1
   # bind 127.0.0.1 -::1
   ```

   修改port

   ```
   port 6333
   ```

2. 配置命令

   - 获取配置

     ```bash
     #获取配置信息
     127.0.0.1:6333> CONFIG GET CONFIG_SETTING_NAME
     #列如
     127.0.0.1:6333> config get loglevel
     1) "loglevel"
     2) "notice"
     127.0.0.1:6333>
     #获取全部
     127.0.0.1:6333> config get *
       1) "rdbchecksum"
       2) "yes"
       3) "daemonize"
       4) "yes"
       5) "io-threads-do-reads"
       6) "no"
       7) "lua-replicate-commands"
       8) "yes"
       9) "always-show-logo"
      10) "no"
      11) "protected-mode"
      12) "no"
      13) "rdbcompression"
      14) "yes"
     ....................
     ```

   - 修改配置

     ```bash
     127.0.0.1:6333> config set loglevel "notice"
     OK
     ```

注册系统服务

1. 创建文件，复制配置

- ```bash
  $ mkdir /etc/redis
  ```

- ```bash
  $ cp ~/software/redis-6.2.1/redis.conf /etc/redis
  ```

- ```bash
  $ cp /etc/redis/redis.conf /etc/redis/6333.conf
  ```

3. 复制redis-server  redis-cli

```bash
$ cp ~/software/redis-6.2.1/redis.conf/src/redis-server /usr/local/bin/redis-server
$ cp ~/software/redis-6.2.1/redis.conf/src/redis-cli /usr/local/bin/redis-server
```

4. 拷贝脚本

```bash
$ cat software/redis-6.2.1/utils/redis_init_script >> /etc/ini.d/redis
#如果修改了port 需要更改脚本文件中的port
$ chmod 755 /etc/ini.d/redis
```

5. 注册开启服务

```bash
$ update-rc.d redis defaults
```

6. 开启服务

```bash
$ service redis start
```

## 


