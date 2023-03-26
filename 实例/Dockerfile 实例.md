#### 多阶段构建

层的共享机制可以节约大量的磁盘空间和传输带宽，在同一层中进行多次构建，每一次构建都需要一个基础镜像，使用`FROM`连接；
每一条 `FROM` 指令都是一个构建阶段，多条 `FROM` 就是多阶段构建，最后生成的镜像是最后一个阶段的结果，**能够将前置阶段中的文件拷贝到后边的阶段中，这就是多阶段构建的最大意义。**


```shell
COPY --from=0  第一阶段作为来源 第二阶段的目标文件 # --from 无指定名字会从数字表示的层获取
==============================================
FROM alpine:latest AS first
...
COPY --from=first 第一阶段作为来源 第二阶段的目标文件 # --from 指定名字会获取名字处的层
```

#### 精简版镜像执行可执行文件

精简版`alpine`镜像一般使用的命令行是`sh`，但是`alpine`使用的`c`库使用`mini`版的`musl libc`与其他`Linux`发行版使用的`gnu libc`不一样。虽说号称兼容，但也只是部分兼容了，实际应用中，可能会因为这个库问题导致诸如一下的问题：

```shell
# 实际上文件目录中存在./server
/bin/sh: ./server: not found
```

**解决方法：**
- 通过一个软链接来关联：
>  首先使用`file`查看执行文件的架构模式，如
>    ![[Pasted image 20230316175002.png]]
> 可以注意到`/lib/ld-linux-armhf.so.3` ，则需要在精简镜像基础上，对此动态库链接


```shell
# amd64
mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2 
# armv7
ln -s /lib/libc.musl-armv7.so.1 /lib/ld-linux-armhf.so.3
```

- 通过补充`libc`库
```shell
apk add libc6-compat
```
