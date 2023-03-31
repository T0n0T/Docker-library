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
docker save -o 打包名字.tar CONTAINER_ID
# 加载保存的镜像
docker load NAME
# 查看镜像信息
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
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

#### docker volumes

有三种基础模式：volumes，bind，tmpfs


---

#### docker drivers

有三种基础模式：volumes，bind，tmpfs
- volumes
- bind 即绑定装载，bind mount，<font color="#da73ff">容器目录本身如果有内容，建议不要使用bind</font> ，会被清空。现在已经被volumes全面替代
- tmpfs 可以避免数据永久存储在任何位置，并通过避免写入容器的可写层来提高容器的性能

具体的volumes：
顶级卷词条下的条目可以为空，在这种情况下，会使用引擎配置的默认驱动程序（在大多数情况下，这是本地驱动程序）

1. **机器间共享数据**  

当构建高可用应用程序，你需要配置多个相同的服务访问相同文件。  

![](https://docs.docker.com/storage/images/volumes-shared-storage.svg)

有几种方法可以达到这种效果。一种是在你的应用中添加对云存储文件的访问，如 Amazon S3。另一种是使用支持外服存储驱动（NFS，  Amazon S3 ）的volume。
Volume驱动允许你在应用中抽象下层的存储系统。例如，如果你的服务使用NFS驱动volume，你可以使用不同的驱动更新服务，就像存储在云中的数据，不需要修改应用逻辑。  

2. **使用volume驱动** 

当你使用docker volume create创建一个volume，或者当你启动一个带有没创建volume的容器，你可以指定volume驱动。下面例子使用 vieux/sshfs volume驱动 ，首先创建一个独立的volume，然后启动一个创建新volume的容器。  
**初始化设置**  
这个例子假设你有两个节点，第一个是Docker主机而且可以连接到第二个的ssh.  

在Docker主机中安装vieux/sshfs插件：
```shell
docker plugin install  --grant-all-permissions  vieux/sshfs
```
**使用volume驱动创建volume**  

这个样例指定一个SSH密码，但是如果两个主机共享keys配置，你可以省略密码。每个volume驱动可以没有或者更多配置选项，可以使用-o标识。 
```shell
docker volume create  --driver  vieux/sshfs  \
 -o   sshcmd = test @node2:/home/test  \
 -o   password = testpassword  \
  sshvolume
  # test @node2:/home/test 为远程主机挂载点 
``` 
相似的`docker-compose.yml`中
```shell
volumes:
  example:
    driver_opts:
      type: "nfs"
      o: "addr=10.40.0.199,nolock,soft,rw"
      device: ":/docker/example"
```

**启动一个带有使用volume驱动创建volume的容器**  

这个样例指定一个SSH密码，但是如果两个主机共享keys配置，你可以省略密码。每个volume驱动可以没有或者更多配置选项。如果volume驱动要穿可选参数，你必须使用—mount。
```shell
docker run  -d   \
 --name  sshfs-container  \
 --volume-driver  vieux/sshfs  \
 --mount   src = sshvolume,target = /app,volume-opt = sshcmd = test @node2:/home/test,volume-opt = password = testpassword  \
  nginx:latest
```

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

#### 深度清理


```shell
# 查看docker的空间占用情况
docker system df 
# 深度清理docker占用
docker system prune
```
