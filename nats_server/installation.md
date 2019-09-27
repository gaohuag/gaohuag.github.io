# NATS Server 安装

 NATS 的哲学是简单。安装只是解压缩zip文件并将二进制文件复制到适当的目录;

您还可以使用您最喜欢的包管理器。下面是一些安装或运行NATS的不同方法:


- [Docker](#installing-via-docker)
- [Kubernetes](#installing-on-kubernetes-with-nats-operator)
- [Package Manager](#installing-via-a-package-manager)
- [Release Zip](#downloading-a-release-build)
- [Development Build](#installing-from-the-source)


### 通过 Docker 安装

使用Docker可以轻松安装服务器without scattering binaries and other artifacts on your system. 
唯一的先决条件是 [安装 docker](https://docs.docker.com/install).

```
> docker pull nats:latest
latest: Pulling from library/nats
Digest: sha256:0c98cdfc4332c0de539a064bfab502a24aae18ef7475ddcc7081331502327354
Status: Image is up to date for nats:latest
docker.io/library/nats:latest
```

在 Docker上运行 NATS :

```
> docker run -p 4222:4222 -ti nats:latest
[1] 2019/05/24 15:42:58.228063 [INF] Starting nats-server version #.#.#
[1] 2019/05/24 15:42:58.228115 [INF] Git commit [#######]
[1] 2019/05/24 15:42:58.228201 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2019/05/24 15:42:58.228740 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2019/05/24 15:42:58.228765 [INF] Server is ready
[1] 2019/05/24 15:42:58.229003 [INF] Listening for route connections on 0.0.0.0:6222
```

更多关于[容器化的 NATS](/nats_docker/README.md)的信息。


### 通过 NATS Operator 在 Kubernetes 上安装 

Installation via the NATS Operator is beyond this tutorial. You can read about the [NATS
Operator](https://github.com/nats-io/nats-operator) here.


### Installing via a Package Manager

On Windows:
```
> choco install nats-server
```

On Mac OS:
```
> brew install nats-server
```

To test your installation (provided the executable is visible to your shell):

```
> nats-server
[41634] 2019/05/13 09:42:11.745919 [INF] Starting nats-server version 2.0.0
[41634] 2019/05/13 09:42:11.746240 [INF] Listening for client connections on 0.0.0.0:4222
...
[41634] 2019/05/13 09:42:11.746249 [INF] Server id is NBNYNR4ZNTH4N2UQKSAAKBAFLDV3PZO4OUYONSUIQASTQT7BT4ZF6WX7
[41634] 2019/05/13 09:42:11.746252 [INF] Server is ready
```

### Downloading a Release Build

You can find the latest release of nats-server [here](https://github.com/nats-io/nats-server/releases/latest).

Download the zip file matching your systems architecture, and unzip. For this example, assuming version 2.0.0 of the server and a Linux AMD64:

```
> curl -L https://github.com/nats-io/nats-server/releases/download/v2.0.0/nats-server-v2.0.0-linux-amd64.zip -o nats-server.zip

> unzip nats-server.zip -d nats-server
Archive:  nats-server.zip
   creating: nats-server-v2.0.0-darwin-amd64/
  inflating: nats-server-v2.0.0-darwin-amd64/README.md
  inflating: nats-server-v2.0.0-darwin-amd64/LICENSE
  inflating: nats-server-v2.0.0darwin-amd64/nats-server

> cp nats-server-v2.0.0darwin-amd64/nats-server /usr/local/bin

```

### Installing From the Source

If you have Go installed, installing the binary is easy:

```
> GO111MODULE=on go get github.com/nats-io/nats-server/v2
```

This mechanism will install a build of [master](https://github.com/nats-io/nats-server), 
which almost certainly will not be a released version. If you are a developer and want to play with the latest, this is the easiest way. 

To test your installation (provided the $GOPATH/bin is set):

```
> nats-server
[41634] 2019/05/13 09:42:11.745919 [INF] Starting nats-server version 2.0.0
[41634] 2019/05/13 09:42:11.746240 [INF] Listening for client connections on 0.0.0.0:4222
...
[41634] 2019/05/13 09:42:11.746249 [INF] Server id is NBNYNR4ZNTH4N2UQKSAAKBAFLDV3PZO4OUYONSUIQASTQT7BT4ZF6WX7
[41634] 2019/05/13 09:42:11.746252 [INF] Server is ready
```


