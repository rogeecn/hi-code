---
title: 'Docker --link 报错 Error response from daemon: Cannot link to /xxx, as it does not belong to the default network.'
date: 2018-03-02 00:00:00
tags: ["Docker"]
abbrlink: docker-link-error-does-not-belong-to-the-default-network
img: ""
comments: false
---

我先使用docker-compose创建了一套服务
```
version: '2'
services:
  db:
    image: mongo:3
  mq:
    image: rabbitmq:3
  api:
    build: .
    image: my_app/api
    ports:
      - "3000:3000"
    links:
      - db
      - mq
    environment:
      - NODE_ENV=development
```
当尝试使用命令连接容器的时候
```
docker run --link my_app_mq_1:mq --link my_app_db_1:db -it worker 
```
报了下面的错：
> docker: Error response from daemon: Cannot link to /my_app_mq_1, as it does not belong to the default network.




原因是docker-compose在启动窗口的时候会创建一个默认的网络，需要将你新启动容器的网络与连接容器的网络放在一个网络下即可。

查看指定窗口的网络：
```
docker inspect -f '{{json .NetworkSettings.Networks.myapp_default.Aliases}}' my_app_db_1
```

添加网络连接参数再运行即可：

```
docker run -it --net myapp_default worker
```
