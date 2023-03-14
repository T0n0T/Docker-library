### docker compose 使用

```shell
docker-compose up    
# 若是要后台运行： $ docker-compose up -d
# 若不使用默认的docker-compose.yml 文件名：    
docker-compose -f server.yml up -d

```


### docker compose 语法

docker-compose标准的配置文件应该包含`version`、`services`、`networks`
- `version` ：定义容器版本
- `services` ：定义容器服务
- `network` ：定义容器使用的网络

#### 概览

```shell
version: '3'
services:
  httpd:
    image: dockercloud/hello-world
    ports:
      - 8080
    networks:
      - front-tier
      - back-tier
 
  redis:
    image: redis
    links:
      - web
    networks:
      - back-tier
 
  lb:
    image: dockercloud/haproxy
    ports:
      - 80:80
    links:
      - web
    networks:
      - front-tier
      - back-tier
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock 
 
networks:
  front-tier:
    driver: bridge
  back-tier:
	driver: bridge


```

在概览中，出现了几种变量，在这里解释一个 `docker-compose.yml` 可以拥有的变量：
- `image` 指定服务的镜像名称或镜像id，如果镜像在本地存在，就会使用本地的镜像，如果不存在，compose就会自动去尝试拉取这个镜像
- `build` 基于dockerfile,在使用up启动的时候会自动执行构建任务，可以指定dockerfile所在的文件夹的路径，compose会将他自动构建这个镜像，让后使用这个镜像启动服务容器
