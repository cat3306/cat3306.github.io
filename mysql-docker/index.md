# mysql docker


## run

```bash
$ docker run --name mysql -v ~/docker/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=12345678 -d -p 3307:3306 --restart=always  mysql
```


