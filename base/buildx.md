
### <font color="#da73ff">BuildKit协同Buildx旨在用于为多个平台构建</font>
要进行docker多平台编译：
可以使用 Buildx 和 Dockerfiles 支持的三种不同策略构建多平台映像:

-   使用内核中的 QEMU 模拟支持
-   **在多个本地节点上使用相同的构建器实例进行构建**
-   **在 Dockerfile 中使用 Stage 进行交叉编译到不同的架构**

当前 <font color="#da73ff">builder 实例</font> 支持多个 docker-container 驱动时，可以同时指定多个平台的编译任务。在这种情况下，docker将建立一个包含所有指定体系结构的映像的清单列表。在 `docker run` 或 `docker service` 中使用镜像时，基于节点平台，Docker 会选择正确的镜像。
> builder实例是docker构建时挂载平台节点的根镜像，一个根镜像可以拥有多个平台的模拟能力，即可以产生对应平台架构的镜像

当你调用一个构建时，你可以设置`--platform`标志来指定用于构建输出的目标平台(例如，Linux/AMD64、Linux/ARM64或Darwin/AMD64)

> 其中**buildx**是docker的跨平台编译脚本，用于创建多个builder实例
> **buildkit**是docker的编译工具链

### Start

buildx包含在linux的安装包内，也可以使用dockerfile安装：
```shell
# syntax=docker/dockerfile:1
FROM docker
COPY --from=docker/buildx-bin:latest /buildx /usr/libexec/docker/cli-plugins/docker-buildx
RUN docker buildx version
```

默认的存储位置在
| OS      | Binary name       | Destination folder                |
|---------|-------------------|-----------------------------------|
| Linux   | docker-buildx     | $HOME/.docker/cli-plugins         |
| macOS   | docker-buildx     | $HOME/.docker/cli-plugins         |
| Windows | docker-buildx.exe | %USERPROFILE%\.docker\cli-plugins |

要设置buildx为默认的镜像构建器，需要
```shell
docker buildx install
# 解除默认构建器
 docker buildx uninstall
```

增加buildx虚拟构建镜像，以及其支持平台：
```shell
# 查看当前平台支持情况
docker buildx ls
# 安装binfmt，它内置了QEMU能提供跨平台构建功能
docker run --privileged --rm tonistiigi/binfmt --install all
# 创建一个构建器并使用
docker buildx create --name xxx --driver docker-container --bootstrap --use
# 删除一个构建起
docker buildx rm xxxx
```
到此已经可以发现构建器被buildx启用，并载入了binfmt的所有支持平台

![[Pasted image 20230324235650.png]]

也可以使用`docker buildx inspect`来查看当前buildx构建实例的架构支持情况：
```shell
root@tiger-linux:/home/tiger/Desktop/code/web/gva-docker# docker buildx inspect
Name:          crossbuilder
Driver:        docker-container
Last Activity: 2023-03-24 16:06:03 +0000 UTC

Nodes:
Name:      crossbuilder0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Buildkit:  v0.11.5
Platforms: linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/arm64, linux/riscv64, linux/ppc64le, linux/s390x, linux/386, linux/mips64le, linux/mips64, linux/arm/v7, linux/arm/v6

```

使用多个本地节点可以更好地支持QEMU无法处理的更复杂的情况，并且通常具有更好的性能
可以使用`--append`标志向构建器实例添加其他节点
```shell
docker buildx create --append --name crossbuilder xxx
```

构建使用实例：
```
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t <username>/<image>:latest .
```

### docker buildx imagetools

使用下面这个命令，可以查看使用buildx构建了几种架构的镜像
```shell
docker buildx imagetools inspect <username>/<image>:latest
```

从而可以通过完整的镜像名，在机器上运行针对其他架构的镜像，如下面：
```shell
docker run --rm docker.io/<username>/<image>:latest@sha256:723c22f366ae44e419d12706453a544ae92711ae52f510e226f6467d8228d191 bash
```

也可以直接使用docker-desktop来使用以上工具