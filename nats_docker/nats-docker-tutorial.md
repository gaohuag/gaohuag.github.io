## NATS Docker Tutorial

在本教程中，您将运行[NATS服务器Docker映像](https://hub.docker.com/_/nats/)。Docker映像提供了[NATS服务器](/README.md)的一个实例。
Synadia积极维护和支持nats-server Docker映像。NATS镜像大小只有6MB。
**1. 安装 Docker。**

参见 [Get Started with Docker](http://docs.docker.com/mac/started/) 获取指导.

运行Docker最简单的方法是使用 [Docker Toolbox](http://docs.docker.com/mac/step_one/).

**2. 运行 nats-server Docker 镜像。**

```sh
> docker run -p 4222:4222 -p 8222:8222 -p 6222:6222 --name nats-server -ti nats:latest
```

**3. 确认NATS服务器正在运行。**

你应该看到以下内容:

```sh
Unable to find image 'nats:latest' locally
latest: Pulling from library/nats
2d3d00b0941f: Pull complete 
24bc6bd33ea7: Pull complete 
Digest: sha256:47b825feb34e545317c4ad122bd1a752a3172bbbc72104fc7fb5e57cf90f79e4
Status: Downloaded newer image for nats:latest
```

然后，表示NATS服务器正在运行:


```sh
[1] 2019/06/01 18:34:19.605144 [INF] Starting nats-server version 2.0.0
[1] 2019/06/01 18:34:19.605191 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2019/06/01 18:34:19.605286 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2019/06/01 18:34:19.605312 [INF] Server is ready
[1] 2019/06/01 18:34:19.608756 [INF] Listening for route connections on 0.0.0.0:6222
```

注意，下载NATS服务器Docker映像的速度很快的。它只有6 MB大小。

**4. 测试NATS服务器，确认它正在运行。**

测试客户端连接端口的一种简单方法是使用telnet。

```sh
> telnet localhost 4222
```

预期结果:

```sh
Trying ::1...
Connected to localhost.
Escape character is '^]'.
INFO {"server_id":"NDP7NP2P2KADDDUUBUDG6VSSWKCW4IC5BQHAYVMLVAJEGZITE5XP7O5J","version":"2.0.0","proto":1,"go":"go1.11.10","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":13249} 
```

您还可以测试监控端点，用浏览器查看`http://localhost:8222`。
