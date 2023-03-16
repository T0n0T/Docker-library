### docker compose 使用

##### **docker-compose服务运维**
- `docker-compose` 

```shell
  -f，–file FILE # 指定Compose模板文件，默认为docker-compose.yml，可以多次指定。
  -p，–project-name # NAME指定项目名称，默认将使用所在目录名称作为项目名。
  -x-network-driver # 使用Docker的可拔插网络后端特性（需要Docker 1.9+版本）
  -x-network-driver # DRIVER指定网络后端的驱动，默认为bridge（需要Docker 1.9+版本）
  -verbose # 输出更多调试信息
  -v，–version # 打印版本并退出
```

- + `up`  
```shell
  -d # 在后台运行服务容器
  –no-color # 不使用颜色来区分不同的服务的控制输出
  –no-deps # 不启动服务所链接的容器
  –force-recreate # 强制重新创建容器，不能与–no-recreate同时使用
  –no-recreate # 如果容器已经存在，则不重新创建，不能与–force-recreate同时使用
  –no-build # 不自动构建缺失的服务镜像
  –build # 在启动容器前构建服务镜像
  –abort-on-container-exit # 停止所有容器，如果任何一个容器被停止，不能与-d同时使用
  -t, –timeout TIMEOUT # 停止容器时候的超时（默认为10秒）
  –remove-orphans # 删除服务中没有在compose文件中定义的容器
  –scale SERVICE=NUM # 设置服务运行容器的个数，将覆盖在compose中通过scale指定的参数

```

- +`down`
```shell
  –rmi type # 删除镜像，类型必须是：all，删除compose文件中定义的所有镜像；local，删除镜像名为空的镜像
  -v, –volumes # 删除已经在compose文件中定义的和匿名的附在容器上的数据卷
  –remove-orphans # 删除服务中没有在compose中定义的容器
```

##### **docker-compose开发调试**
- + `build`
```shell
  –compress # 通过gzip压缩构建上下环境
  –force-rm # 删除构建过程中的临时容器
  –no-cache # 构建镜像过程中不使用缓存
  –pull # 始终尝试通过拉取操作来获取更新版本的镜像
  -m, –memory # MEM为构建的容器设置内存大小
  –build-arg key=val # 为服务设置build-time变量
```

- + `logs`
```shell
docker-compose logs [options] [SERVICE...] # 查看服务容器的输出
```

- + `pause`
```shell
docker-compose pause [SERVICE...] # 暂停一个容器
docker-compose unpause [SERVICE...] #恢复容器
```

- + `port`
```shell
docker-compose port [options] SERVICE PRIVATE_PORT # 显示某个容器端口所映射的公共端口
   –protocol=proto，# 指定端口协议，TCP（默认值）或者UDP
   –index=index，# 如果同意服务存在多个容器，指定命令对象容器的序号（默认为1）
```

- + `exec`
```shell
docker-compose exec [options] SERVICE COMMAND [ARGS...]
  -d 分离模式，后台运行命令。
  –privileged 获取特权。
  –user USER 指定运行的用户。
  -T 禁用分配TTY，默认docker-compose exec分配TTY。
  –index=index，当一个服务拥有多个容器时，可通过该参数登陆到该服务下的任何服务，例如：docker-compose exec –index=1 web /bin/bash ，web服务中包含多个容器
```


---

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

##### <font color="#da73ff">services</font> ：
- `image` 指定服务的镜像名称或镜像`id`，如果镜像在本地存在，就会使用本地的镜像，如果不存在，`compose`就会自动去尝试拉取这个镜像
- `build` 基于 `dockerfile`，在使用 `up` 启动的时候会自动执行构建任务，可以指定 `dockerfile` 所在的文件夹的路径，`compose` 会将他自动构建这个镜像，让后使用这个镜像启动服务容器
- `context` 可以是 `dockerfile` 的文件路径，也可以是到链接到 `git` 仓库的 `url`，当提供的值是相对路径时，被解析为相对于撰写文件的路径
- `dockefile` 使用 `dockerfile` 文件来构建，必须使用 `context` 指定构建路径
- `command` 使用command可以覆盖容器启动后默认执行的命令
- `container_name` <项目名称><服务名称><序号>自定义项目名称、服务名称
- `depends_on` 确保该容器依赖的前置条件，如另一个数据库容器已经打开
- `volumes` 使用 `HOST:CONTAINER` 格式挂载宿主机文件，可以使用 `HOST:CONTAINER:ro` 限定为只读
- `volumes_from` 从另一个容器或服务处挂载数据
- `entrypoint` 指定接入点，可以覆盖 `dockerfile` 中的定义
- `env_file` 设置环境变量的配置文件
- `environment` 在文件中指定环境变量，会覆盖 `env_file`
- `privileged` 使容器内`root`拥有宿主机`root`的权利
- `cap_add` 增加容器内核能力，这通常可以给容器又有宿主机的socket等实体外设操作能力
- `cap_drop` 去掉某种内核能力
- `devices` 使用 `HOST:CONTAINER` 格式映射宿主机设备

##### <font color="#da73ff">network</font> ：
- `hostname` 设置主机`name`
- `ports` 使用 `HOST:CONTAINER` 格式或者只是指定容器的端口，宿主机会随机映射端口
- `dns` 指定服务器
- `network_mode` 设置网络模式
- `network` 指定自定义网络
- `extends` 基于其他配置文件，可以形成嵌套
- `links` 链接到其它服务中的容器。使用服务名称（同时作为别名），或者“服务名称:服务别名”

```c
services:
	web:
		container_name: xxxxx
	    build:
		    context: ./dir
		    dockerfile: DockerfileXXXX
		ports:
		 - "3000"  //随机映射
		 - "8080:8080"
		 - "127.0.0.1:8888:8888"
		net: "bridge"
		//net: "none"
		//net: "host"
		volumes:  
		  - /var/lib/mysql  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）  
		  - /opt/data:/var/lib/mysql // 使用绝对路径挂载数据卷  
		  - ./cache:/tmp/cache // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器
		volumes_from:
			- service_name    
			- container_name
		dns：
			- 8.8.8.8   
			- 9.9.9.9
		entrypoint: /code/entrypoint.sh
		env_file:
			- ./common.env
			- ./apps/web.env
			- /opt/secrets.env
		cap_add:
		    - ALL
		devices:
		    - "/dev/ttyUSB1:/dev/ttyUSB0"

	depends_on:
	    - mysql
	    - redis

	redis:
	    image: "redis:latest"
	    extends:
	        file: xxxxx.yml //extends不会继承links和volumes_from中定义的容器和数据卷资源
	        service: xxxxx  //在extend的配置中，尽量只定义共享的镜像和环境变量

    mysql:
        network_mode: "bridge"
        environment:
            MYSQL_ROOT_PASSWORD: "111111"
            MYSQL_USER: 'test'
            MYSQL_PASS: '111111'
        image: "mysql:latest"
        restart: always
        volumes:
            - "./db:/var/lib/mysql"              //自定义的配置文件的载入
            - "./conf/my.cnf:/etc/my.cnf"
            - "./init:/docker-entrypoint-initdb.d/"
        ports:
            - "33060:3306"

	web2：
		links:	
		    - mysql	
		    - mysql:database	
		    - redis
```
