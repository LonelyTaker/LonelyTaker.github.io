---
title: docker-compose
date: 2024-01-02 14:48:58
categories: 其他
---

# nginx

```yaml
services:
  nginx:
    image: nginx:1.24.0-alpine-slim
    container_name: nginx
    network_mode: bridge
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /data/container/nginx/config:/etc/nginx
      - /data/container/nginx/data:/usr/share/nginx/html
      - /data/container/nginx/logs:/var/log/nginx
    ports:
      - 80:80
    restart: always
```

<br />

# mysql

```yaml
version: '3'
services:
  mysql:
    image: mysql:8.0.20
    container_name: mysql
    volumes:
      - /data/container/mysql/config:/etc/mysql
      - /data/container/mysql/data:/var/lib/mysql
      - /data/container/mysql/logs:/var/log/mysql
    environment:
      - "MYSQL_ROOT_PASSWORD=xxxxxx"
      - "TZ=Asia/Shanghai"
    ports:
      - 3306:3306
    restart: always
```

<br />

# redis

```yaml
services:
  redis:
    image: redis:7.4.0
    container_name: redis
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /data/container/redis/config:/etc/redis
      - /data/container/redis/data:/data
    ports:
      - 6379:6379
    restart: always
    command: ["redis-server","/etc/redis/redis.conf"]
```

>需要提前创建配置文件，配置文件可从官网下载
>
>https://redis.io/docs/latest/operate/oss_and_stack/management/config/
