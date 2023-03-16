##### 网络模式

- 几种 `network_mode` 
```shell
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

- 自定义 `network` 的模板
```shell
version: "3"
services:
  some-service:
	...
    networks:
     - some-network
     - other-network

...
networks:
  some-network:
  other-network:
```

- `ports` 暴露端口的定义
```shell
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

##### 环境
- `working_dir`指定容器中工作目录，覆盖`Dockerfile`
```shell
working_dir: /code
```

- `restart`指定容器退出后的重启策略为始终重启。该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 `always` 或者 `unless-stopped`。

```
restart: always
```

- 有时候会需要输入空循环的`shell`命令来保持容器开启，方便调试

```shell
/bin/sh -c "while true;do sleep 3600;done"
```
