# 运行
nat服务器有许多命令行参数。首先，您不必指定任何参数。

在没有任何参数的情况下，NATS服务器将开始监听端口4222上的NATS客户机连接。

默认情况下，不开启安全（tls）传输

### 独立运行

当服务器启动时，它会打印一些信息，包括服务器在哪里监听客户端连接:

```
> nats-server
[41634] 2019/05/13 09:42:11.745919 [INF] Starting nats-server version 2.0.0
[41634] 2019/05/13 09:42:11.746240 [INF] Listening for client connections on 0.0.0.0:4222
...
[41634] 2019/05/13 09:42:11.746249 [INF] Server id is NBNYNR4ZNTH4N2UQKSAAKBAFLDV3PZO4OUYONSUIQASTQT7BT4ZF6WX7
[41634] 2019/05/13 09:42:11.746252 [INF] Server is ready
```


### Docker 中运行

如果你在docker容器中运行NATS服务器:

```
> docker run -p 4222:4222 -ti nats:latest
[1] 2019/05/13 14:55:11.981434 [INF] Starting nats-server version 2.0.0
...
[1] 2019/05/13 14:55:11.981545 [INF] Starting http monitor on 0.0.0.0:8222
[1] 2019/05/13 14:55:11.981560 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2019/05/13 14:55:11.981565 [INF] Server is ready
[1] 2019/05/13 14:55:11.982492 [INF] Listening for route connections on 0.0.0.0:6222
...
```

更多信息参考 [containerized NATS is available here](/nats_docker/README.md).
