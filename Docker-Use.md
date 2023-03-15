> 首先，由于`docker daemon`需要绑定到主机的`Unix socket`而不是普通的TCP端口，而`Unix socket`的属主为`root`用户，所以其他用户只有在命令前添加`sudo`选项才能执行相关操作；可以为`docker`增加组，使用组策略为当前用户赋权。也可以直接给sock文件赋权。
> 
```shell
# 方法一，有可能单客户端生效
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
# 方法二
sudo chmod 666 /var/run/docker.sock
sudo systemctl restart docker
```

---

#### docker镜像操作

```shell
# 搜索镜像，获取网上镜像目录
docker search xxxx
# 获取镜像，若不使用tag会默认最新
docker pull [options] NAME[:TAG]
# 列出当前本地镜像，参数不选则列出全部
docker images [options] [REPOSITORY[:TAG]]
# 运行镜像，本地没有镜像则会拉取
docker run [options] IMAGE[:TAG] [COMMAND] [ARG..]
# 运行镜像后，在容器内部执行某种命令
docker exec [options] CONTAINER [COMMAND] [ARG..]
# 特别的，如下命令可以进入容器并开启特定的终端
docker exec -it CONTAINER bash
# 删除容器，-f表强制
docker rm [-f] CONTAINER
# 删除所有容器
docker rm -f `docker ps -aq`
# 删除镜像
docker image rm [image]
# 保存镜像相当于快照
docker save -o NAME CONTAINER
# 加载保存的镜像
docker load NAME

# 复制文件
docker cp filename CONTAINER:/filename
docker cp CONTAINER:/filename filename 
# 复制大量文件
docker cp srcbolder/. container_id:/dstbolder
```

> 在容器内执行命令是开启终端进入容器时，`-i` 保证我们的输入有效,即使在没有 `detach` 的情况下也能运行。 `-t` 表示将分配给我们一个伪终端.我们将在伪终端输入我们的内容。

都可以使用 `--help` 查看手册

---

#### docker网络

如下图：
1. docker在<font color="#da73ff">默认</font>情况下，一般会分配一个独立的`network-namespace`，也就是网络类型中的Bridge模式，docker可以指定你想把容器内的某一个端口可以在容器所在主机上的某一个端口它俩之间做一个映射
2. `Host`模式,如果在启动容器的时候指定使用Host模式，那么这个容器将不会获得一个独立的`network` `namespace`，而是和主机共同使用一个，这个时候容器将不会虚拟出自己的网卡,配置出自己的`ip`，而是使用宿主机上的`ip`和端口，包括`socket`
3. 还有一种网络类型是`None`也就是没有网络
![[Pasted image 20230314165426.png]]

---

#### dockerfile 

Dockerfile 指令选项:

```shell
Dockerfile 指令选项:

FROM                  #基础镜像 。 （centos）
RUN                   #镜像构建的时候需要执行的命令。
CMD                   #类似于 RUN 指令，用于运行程序（只有最后一个会生效，可被替代）
EXPOSE                #对外开放的端口。
ENV                   #设置环境变量，定义了环境变量，在后续的指令中，可以使用这个环境变量。
ADD                   # 步骤：tomcat镜像，这个tomcat压缩包。添加内容。
COPY                  #复制指令，将文件拷贝到镜像中，COPY可以使用相对路径和绝对路径，相对路径是相对于dockerfile的。
VOLUME                #设置卷，挂载的主机目录。
USER                  #用于指定执行后续命令的用户和用户组，
                       这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。
WORKDIR               #工作目录（类似CD命令）。
ENTRYPOINT            #类似于 CMD 指令，但其不会被docker run的命令行参数指定的指令所覆盖，会追加命令。
ONBUILD               #当构建一个被继承Dokcerfile，就会运行ONBUILD的指令。出发执行。


注意：CMD类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
CMD 在docker run 时运行。
RUN 是在 docker build。
作用：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。
CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。
如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。

LABEL
LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：
LABEL <key>=<value> <key>=<value> <key>=<value> ...
比如我们可以添加镜像的作者：
LABEL org.opencontainers.image.authors="runoob"
```

---

#### 制作镜像

在某些镜像的基础上，制作自己的镜像，最基础的需要需要：
- 使用 `RUN` ，`ENV` 在 `dockerfile` 中增加自己需要 `docker内系统` 执行的命令，`环境变量` 
- 并将本地需要投入容器中的文件使用 `COPY` 复制到其中，包括资源文件，配置文件 
- `FROM` 指定基础镜像
随后使用如下命令构建新镜像

```shell
# 在当前目录构建镜像
docker build .
# 重命名镜像为 test:latest
docker build -t test:latest .
# 或者使用commit提交为新副本
docker commit -m="描述信息" -a="作者" container_id name:[TAG]
# 重命名镜像
docker tag container_id name:[TAG]
```