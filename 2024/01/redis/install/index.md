# redis install


### docker compose

``` yaml
version: '3.8'
services:
  cache:
    image: redis
    restart: always
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning --requirepass facai888
    volumes: 
      - cache:/data
volumes:
  cache:
    driver: local
```

