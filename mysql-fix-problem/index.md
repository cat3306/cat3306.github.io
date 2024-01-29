# docker mysql 数据恢复


## 缘由

本地的`docker`的 `k8s` 经常启动失败，谷歌检索问题后，`stack overflow` 上的帖子说到一种解决方案[docker-desktop-kubernetes-failed-to-start](https://stackoverflow.com/questions/63312861/docker-desktop-kubernetes-failed-to-start)，没错，这个解决方案，要把本地的所有镜像容器给干掉包括`docker volume`，我想着本地的容器也没啥，唯独mysql的数据是挂载外部的 `/Users/joker/docker/mysql/lib`，于是我初始化了docker 的所有镜像和容器。但是当我重新启动`mysql`容器，绑定这个路径发现容器根本起不起来，内部错误` Unknown redo log format`。完了，里面的数据读不出来。不过文件还在.ibd文件还在，应该可以恢复



## 恢复过程

1. 启动一个纯净mysql容器

docker-compose.yml

```yaml
version: '3.9'

services:
  mysql:
    image: mysql
    ports:
      - 3307:3306
    restart: always
    volumes:
      - mysqldata:/var/lib/mysql
      - /Users/joker/docker/mysql/lib/pwd/my_pwd.ibd:/my_pwd.ibd
    environment:
      - MYSQL_ROOT_PASSWORD=12345678
      - TZ=Asia/Shanghai
volumes:
  mysqldata:
```

把`/Users/joker/docker/mysql/lib/pwd/my_pwd.ibd`映射到容器里面

2. 把表结构同步到新的mysql中，并执行

   ```sql
   ALTER TABLE my_pwd DISCARD TABLESPACE; 
   ```

3. `/my_pwd.ibd` 将拷贝到`/var/lib/mysql/pwd`中，并且修改权限

   ```
   root@03dd4294e1cb:/var/lib/mysql/pwd# cp /my_pwd.ibd  ./
   root@03dd4294e1cb:/var/lib/mysql/pwd# chown mysql:mysql my_pwd.ibd
   ```

4. ```sql
   ALTER TABLE my_pwd IMPORT TABLESPACE;
   ```

结果一看数据果然恢复回来了




