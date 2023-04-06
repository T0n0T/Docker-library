#针对于linux环境搭建 
Docker的潜在使用工具，包括了
- `Docker Engine` docker的主要部件，功能实现皆基于此，包含了基本的命令行调用命令
- `Docker Desktop` docker的桌面管理程序，图形化的界面管理本地的容器
- `docker compose` 可以使用`docker-compose.yml`文件一次协调多个Docker容器的配置

---

### Docker Engine 安装

```bash
# 保证旧有docker程序在本地无残留
sudo apt-get remove docker docker-engine docker.io containerd runc
# 获取docker所需要的前置工具
sudo apt-get update
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
# 使用gnupg获取docker官方gpg秘钥
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
# 设置docker的远端存储库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 更新apt源
sudo apt-get update
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
# 获取并安装最新docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

> 若出现 `curl` 问题，如设置代理出现：

```bash
curl: (97) SOCKS4: Failed receiving connect request ack: Failure when receiving data     from the peer
```

> 则按照以下进行操作接触sock代理即可：

```bash
env | grep -i proxy #查询当前端口设置
export ALL_PROXY="" all_proxy="" #最终目的是使sock端口能够为默认状态，不通过代理
```

在安装完成后，可以进行docker的 <font color="#da73ff">hello仪式</font> ：

```bash
sudo docker run hello-world
```

在这之后需要对docker进行换源，国内除docker官方中文站的源，常用的是阿里源，网易源，中科大源，添加dockerdaemon配置，并在文件中增加如下json-code后重启

```bash
vim /etc/docker/daemon.json
```

```shell
{
    "registry-mirrors":[
         "http://docker.mirrors.ustc.edu.cn",
         "http://hub-mirror.c.163.com",
         "http://registry.docker-cn.com"
    ] ,
    "insecure-registries":[
       "docker.mirrors.ustc.edu.cn",
         "registry.docker-cn.com"
    ] ，
    "dns":[
	    "8.8.8.8",
	    "114.114.114.114"
    ]
}
```

```bash
systemctl restart docker
# 查看源设置是否成功
docker info
```

---

### Docker Desktop安装

> 如果没有桌面管理需求，不是必须项

	安装前需要确保gnome-terminal已经安装，但桌面级的ubuntu基本具备这个条件；
	并需要确保旧有desktop配置无残留

```bash
sudo apt install gnome-terminal
sudo apt remove docker-desktop
rm -r $HOME/.docker/desktop
sudo rm /usr/local/bin/com.docker.cli
sudo apt purge docker-desktop
```

[Install Docker Desktop on Ubuntu](https://docs.docker.com/desktop/install/ubuntu/)下载 `deb` 并安装：

```bash
sudo apt-get update
# 下面这条命令需要修改为对应版本安装包名称
sudo apt-get install ./docker-desktop-<version>-<arch>.deb
```

完成安装后，`docker desktop` 会自行运行脚本配置DNS服务器，并链接到 `docker engine` ，使用如下命令检查 `3` 种docker组件的安装情况：
```bash
docker compose version
docker --version
docker version
```

---

### docker compose安装

若在完成安装后，docker compose没有被安装，则需要单独安装

```shell
# 下载二进制执行文件
curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 赋权
chmod +x /usr/local/bin/docker-compose
# 检查
docker compose version
```