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
