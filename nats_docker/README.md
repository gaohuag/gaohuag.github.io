
## NATS Server 容器化

NATS server 在 [Docker Hub](https://hub.docker.com/_/nats/)上以Docker映像的形式提供，您可以使用Docker守护进程运行它。
 NATS server Docker 镜像是非常轻量级的，大小不到10 MB。

[Synadia](https://synadia.com) 积极维护和支持NATS服务器Docker映像。

### 使用

要使用Docker容器映像，请安装Docker并拉取公共映像:

```sh
> docker pull nats
```

运行NATS服务器映像:

```sh
> docker run -d --name nats-main nats
```

默认情况下，NATS服务器暴露多个端口:
- 4222 是为客户端连接。
- 8222 是一个用于信息报告的HTTP管理端口。
- 6222 是一个用于集群的路由端口。
- 使用-p或-P自定义。

举例:

```sh
$ docker run -d --name nats-main nats
[INF] Starting nats-server version 0.6.6
[INF] Starting http monitor on port 8222
[INF] Listening for route connections on 0.0.0.0:6222
[INF] Listening for client connections on 0.0.0.0:4222
[INF] nats-server is ready
```

运行端口暴露在主机上:

```sh
> docker run -d -p 4222:4222 -p 6222:6222 -p 8222:8222 --name nats-main nats
```

运行第二台服务器并将它们聚在一起:

```sh
> docker run -d --name=nats-2 --link nats-main nats --routes=nats-route://ruser:T0pS3cr3t@nats-main:6222
```

**注意** Docker映像需要使用我们上面提供的凭证来保护路由。 
摘自 [from Docker image configuration](https://github.com/nats-io/nats-docker/blob/master/amd64/nats-server.conf#L16-L20)：

```ascii
# Routes are protected, so need to use them with --routes flag
# e.g. --routes=nats-route://ruser:T0pS3cr3t@otherdockerhost:6222
authorization {
  user: ruser
  password: T0pS3cr3t
  timeout: 2
}
```

确认路由是否已连接:

```sh
$ docker run -d --name=nats-2 --link nats-main nats --routes=nats-route://ruser:T0pS3cr3t@nats-main:6222 -DV
[INF] Starting nats-server version 2.0.0
[INF] Starting http monitor on port 8222
[INF] Listening for route connections on :6222
[INF] Listening for client connections on 0.0.0.0:4222
[INF] nats-server is ready
[DBG] Trying to connect to route on nats-main:6222
[DBG] 172.17.0.52:6222 - rid:1 - Route connection created
[DBG] 172.17.0.52:6222 - rid:1 - Route connect msg sent
[DBG] 172.17.0.52:6222 - rid:1 - Registering remote route "ee35d227433a738c729f9422a59667bb"
[DBG] 172.17.0.52:6222 - rid:1 - Route sent local subscriptions
```

## 使用Docker进行集群化

下面是一些使用Docker设置nat-server集群的例子。我们在一个名为conf的文件夹下放置了3个不同的配置(每个nat-server服务器一个)，如下所示:

```ascii
|-- conf
    |-- nats-server-A.conf
    |-- nats-server-B.conf
    |-- nats-server-C.conf
```

每个文件都有以下内容:(这里我使用的是ip 192.168.59.103作为示例，因此只需替换来自服务器的适当ip即可)

### Example 1: Setting up a cluster on 3 different servers provisioned beforehand

在本例中，三个服务器都是使用配置文件启动的，配置文件知道其他服务器的情况。
#### nats-server-A

```ascii
# Cluster Server A

port: 7222

cluster {
  host: '0.0.0.0'
  port: 7244

  routes = [
    nats-route://192.168.59.103:7246
    nats-route://192.168.59.103:7248
  ]
}
```

#### nats-server-B

```ascii
# Cluster Server B

port: 8222

cluster {
  host: '0.0.0.0'
  port: 7246

  routes = [
    nats-route://192.168.59.103:7244
    nats-route://192.168.59.103:7248
  ]
}
```

#### nats-server-C

```ascii
# Cluster Server C

port: 9222

cluster {
  host: '0.0.0.0'
  port: 7248

  routes = [
    nats-route://192.168.59.103:7244
    nats-route://192.168.59.103:7246
  ]
}
```

要启动每个服务器上的容器，您应该能够启动以下的nat-server映像:
```sh
docker run -it -p 0.0.0.0:7222:7222 -p 0.0.0.0:7244:7244 --rm -v $(pwd)/conf/nats-server-A.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 7222 -D -V
```

```
docker run -it -p 0.0.0.0:8222:8222 -p 0.0.0.0:7246:7246 --rm -v $(pwd)/conf/nats-server-B.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 8222 -D -V
```

```
docker run -it -p 0.0.0.0:9222:9222 -p 0.0.0.0:7248:7248 --rm -v $(pwd)/conf/nats-server-C.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 9222 -D -V
```

### Example 2: Setting a nats-server cluster one by one

在这种情况下:

- 我们调出A并获取其ip (nat -route://192.168.59.103:7244)

- 然后创建B，然后在其配置中使用A的地址。

- 获取B的地址nat-route://192.168.59.104:7246，创建C，使用A和B的地址。



首先，我们创建节点A，并使用以下配置启动一个nat-server服务器:
```ascii
# Cluster Server A

port: 4222

cluster {
  host: '0.0.0.0'
  port: 7244

}
```

```sh
docker run -it -p 0.0.0.0:4222:4222 -p 0.0.0.0:7244:7244 --rm -v $(pwd)/conf/nats-server-A.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 4222 -D -V
```

然后我们继续创建下一个节点。我们发现第一个节点的ip:port为`192.168.59.103:7244`，因此我们将其添加到路由配置中，如下所示:

```ascii
# Cluster Server B

port: 4222

cluster {
  host: '0.0.0.0'
  port: 7244

  routes = [
    nats-route://192.168.59.103:7244
  ]
}
```

然后启动服务器B:

```sh
docker run -it -p 0.0.0.0:4222:4222 -p 0.0.0.0:7244:7244 --rm -v $(pwd)/conf/nats-server-B.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 4222 -D -V
```

最后，我们创建另一个节点c。我们现在知道了A和B的路由，所以我们可以添加到它的配置:

```ascii
# Cluster Server C

port: 4222

cluster {
  host: '0.0.0.0'
  port: 7244

  routes = [
    nats-route://192.168.59.103:7244
    nats-route://192.168.59.104:7244
  ]
}
```

然后启动它:

```sh
docker run -it -p 0.0.0.0:4222:4222 -p 0.0.0.0:7244:7244 --rm -v $(pwd)/conf/nats-server-C.conf:/tmp/cluster.conf nats -c /tmp/cluster.conf -p 9222 -D -V
```

### 测试集群
    

现在，应该可以执行以下操作:订阅节点a，然后发布到节点c。您应该能够毫无问题地接收消息。

```sh
nats-sub -s "nats://192.168.59.103:7222" hello &

nats-pub -s "nats://192.168.59.105:7222" hello world

[#1] Received on [hello] : 'world'

# nats-server on Node C logs:
[1] 2015/06/23 05:20:31.100032 [TRC] 192.168.59.103:7244 - rid:2 - <<- [MSG hello RSID:8:2 5]

# nats-server on Node A logs:
[1] 2015/06/23 05:20:31.100600 [TRC] 10.0.2.2:51007 - cid:8 - <<- [MSG hello 2 5]
```

## 教程

有关使用NATS服务器Docker映像的更多说明，请参阅 [NATS Docker tutorial](nats-docker-tutorial.md)。
