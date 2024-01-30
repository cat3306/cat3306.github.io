# mysql install


## docker compose

``` yaml
services:
  mysql:
    image: mysql
    container_name: mysql
    hostname: mysqldb
    ports:
      - 3306:3306
    restart: always
    volumes:
      - mysqldata:/var/lib/mysql    
    environment:
      - MYSQL_ROOT_PASSWORD=12345678
      - TZ=Asia/Shanghai
    networks:
      - "mysqlnet"
volumes:
  mysqldata:
networks:
  mysqlnet:
```

``` bash
    docker run --name mysql -v ~/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -d -p 3307:3306 --restart=always  mysql
```

