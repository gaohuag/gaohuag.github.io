# NATS 客户端
nats服务器不与任何客户端绑定。但是大多数客户端库都带有允许发布、订阅、发送请求和回复消息的工具。
如果安装了客户机库，可以尝试使用绑定的客户端。否则，您先安装上客户端。

### 如果你要安装go的例子，执行下面命令:

```
> go get github.com/nats-io/go-nats-examples/tools/nats-pub
> go get github.com/nats-io/go-nats-examples/tools/nats-sub
```

### 或者下载压缩文件：

你可以从 [go-nats-examples repo](https://github.com/nats-io/go-nats-examples/releases/tag/0.0.50) 下载编译好的二进制文件。


### Testing your setup

打开控制台并且 [启动一个nats-server](running.md):
```
> nats-server
[29670] 2019/05/16 08:45:59.836809 [INF] Starting nats-server version 2.0.0
[29670] 2019/05/16 08:45:59.836889 [INF] Git commit [not set]
[29670] 2019/05/16 08:45:59.837161 [INF] Listening for client connections on 0.0.0.0:4222
[29670] 2019/05/16 08:45:59.837168 [INF] Server id is NAYH35Q7ROQHLQ3K565JR4OPTJGO5EK4FJX6KX5IHHEPLQBRSYVWI2NO
[29670] 2019/05/16 08:45:59.837170 [INF] Server is ready
```


在另外一个终端开启一个订阅者:
```
> nats-sub ">"
Listening on [>]
```

注意，当客户端连接时，服务器没有记录任何有趣的内容，因为除非发生有趣的事情，否则服务器输出相对比较安静。

要使服务器输出更详细，可以指定`-V `标志来启用服务器协议跟踪消息的日志记录。

我们按下`<ctrl>+c`键，使用`-V`标志重新启动服务器:

```
> nats-server -V
[29785] 2019/05/16 08:46:12.731278 [INF] Starting nats-server version 2.0.0
[29785] 2019/05/16 08:46:12.731347 [INF] Git commit [not set]
[29785] 2019/05/16 08:46:12.731488 [INF] Listening for client connections on 0.0.0.0:4222
[29785] 2019/05/16 08:46:12.731493 [INF] Server id is NCEOJJ5SBJKUTMZEDNU3NBPJSLJPCMQJUIQIWKFHWE5DPETJKHX2CO2Y
[29785] 2019/05/16 08:46:12.731495 [INF] Server is ready
[29785] 2019/05/16 08:46:13.467099 [TRC] 127.0.0.1:49805 - cid:1 - <<- [CONNECT {"verbose":false,"pedantic":false,"tls_required":false,"name":"NATS Sample Subscriber","lang":"go","version":"1.7.0","protocol":1,"echo":true}]
[29785] 2019/05/16 08:46:13.467200 [TRC] 127.0.0.1:49805 - cid:1 - <<- [PING]
[29785] 2019/05/16 08:46:13.467206 [TRC] 127.0.0.1:49805 - cid:1 - ->> [PONG]
```
如果您已经创建了订阅者，您应该注意到订阅者上的输出，该输出告诉您它已断开连接并重新连接。上面的服务器输出更有趣。
您可以看到订阅者发送一个`CONNECT`协议消息和一个`PING`，服务器用一个`PONG`响应这个`PING`。

> 您可以在这里了解更多关于[NATS协议](/nats_protocol/nats-protocol.md),的信息，但是比协议描述更有趣的是[一个交互式演示](/nats_protocol/nats-protocol-demo.md)。

在第三个终端上，发布您的第一条消息:
```
> nats-pub hello world
Published [hello] : 'world'
```

在订阅者窗口，您应该看到:
```
[#1] Received on [hello]: 'world'
```


### 针对远程服务器进行测试

如果NATS服务器运行在不同的机器或不同的端口，则客户端必须通过_NATS URL_指定服务器。
NATS url的形式是:`NATS://:`和`tls://:`。带有`tls`协议的url具有安全的tls连接。

```
> nats-sub -s nats://server:port ">"
```

如果您想在远程服务器上尝试，NATS 团队维护一个演示服务器，您可以通过 `demo.nats.io` 访问该服务器。

```
> nats-sub -s nats://demo.nats.io ">"
```
本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/clients.md)

