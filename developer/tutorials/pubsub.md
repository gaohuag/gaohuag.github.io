# 探索 NATS 发布/订阅

NATS是一个发布订阅消息系统。监听某个主题的订阅者将收到该主题的消息。如果订阅者没有在线订阅主题，消息将不会被接收。
订阅者可以使用通配符`*`来匹配单个token，或使用`>`匹配主题的尾部。

<div class="graphviz"><code data-viz="dot">
digraph nats_pub_sub {
  graph [splines=ortho, nodesep=1];

  sub1 [shape="box", label="SUB\ncom.msg.one"];
  pub1 [shape="box", label="PUB\ncom.msg.one"];
  non_active [shape="box", label="Non-Active\nSubscriber"];

  {
    rank=same
    pub1 sub1 non_active
  }

  natsserver [shape="box", label="NATS", width=8];

  sub2 [shape="box", label="SUB\ncom.msg.one"];
  sub3 [shape="box", label="SUB\ncom.msg.two"];
  sub4 [shape="box", label="SUB\ncom.msg.*"];

  {
    rank=same
    sub2 sub3 sub4
  }

  pub1 -> natsserver [penwidth=2];
  natsserver -> sub1 [penwidth=2];
  natsserver -> non_active [style=dashed color=red arrowhead="none"];

  natsserver -> sub2 [penwidth=2];
  natsserver -> sub3 [style=dashed color=red arrowhead="none"];
  natsserver -> sub4 [penwidth=2];
}
</code></div>

## 先决条件

Go和NATS服务器应该被安装。您也可以选择使用演示服务器（`nats://demo.nat .io`）做测试。

### 1. 启动NATS服务器

```sh
% nats-server
```

当服务器成功启动时，您将看到以下消息:

```sh
[1] 2019/31/05 15:18:22.301550 [INF] Starting nats-server version 2.0.0
[1] 2019/31/05 15:18:22.301762 [INF] Listening for client connections on 0.0.0.0:4222
[1] 2019/31/05 15:18:22.301769 [INF] nats-server is ready
```

NATS服务器监听TCP端口4222上的客户端连接。

### 2. 启动shell或命令提示符会话

您将使用此会话运行一个示例NATS客户端订阅程序。


### 3. 切换到Go客户端示例目录

```sh
% cd $GOPATH/src/github.com/nats-io/nats.go/examples
```

### 4. 运行客户端订阅程序

```sh
% go run nats-sub/main.go <subject>
```

其中 `<subject>` 是要监听的主题。有效的主题是系统中唯一的字符串。

例如：

```sh
% go run nats-sub/main.go msg.test
```

您应该看到消息: *Listening on [msg.test]*

### 5. 启动另一个shell或命令提示符会话

您将使用此会话运行NATS发布服务器客户端。

## 6. 切换到Go客户端示例目录

```sh
% cd $GOPATH/src/github.com/nats-io/nats.go/examples
```

### 7. 发布NATS消息

```sh
% go run nats-pub/main.go <subject> <message>
```

其中 `<subject>` 是主题名称， `<message>` 是要发布的文本。

例如:

```sh
% go run nats-pub/main.go msg.test hello
```

或者

```sh
% go run nats-pub/main.go msg.test "NATS MESSAGE"
```

### 8. 验证消息发布和接收

您应该看到发布者发送了消息: *Published [msg.test] : 'NATS MESSAGE'*

订阅者收到消息: *[#1] Received on [msg.test]: 'NATS MESSAGE'*

注意，如果接收者没有收到消息，请检查发布者和订阅者是否使用相同的主题名称。

### 9. 发布另一个消息

```sh
% go run nats-pub/main.go msg.test "NATS MESSAGE 2"
```

您应该看到订阅者收到了消息2。请注意，每次您的订阅客户端收到关于该主题的消息时，消息计数都会增加:
### 10. 启动另一个shell或命令提示符会话

您将使用此会话运行第二个NATS订阅服务器。

### 11. 切换到Go客户端示例目录

```sh
% cd $GOPATH/src/github.com/nats-io/nats.go/examples
```

### 12. 订阅消息

```sh
% go run nats-sub/main.go msg.test
```

### 13. 使用发布服务器客户端发布另一条消息

```sh
% go run nats-pub/main.go msg.test "NATS MESSAGE 3"
```

验证两个订阅客户机都收到了消息。

### 14. 启动另一个shell或命令提示符会话

您将使用此会话运行第三个NATS订阅服务器。

### 15. CD to the examples directory

```sh
% cd $GOPATH/src/github.com/nats-io/nats.go/examples
```

### 16. 订阅不同的消息
       
```sh
% go run nats-sub/main.go msg.test.new
```
除最后一个订阅者外的所有订阅者都收到消息。为什么?因为订阅者没有监听发布者使用的消息主题。

### 17. 更新最后一个订阅服务器以使用通配符

NATS支持对消息订阅者使用通配符。不能使用通配符主题发布消息。

更改最后一个用户订阅 msg.*，如下:

```sh
% go run nats-sub/main.go msg.*
```

### 18. 发布另一个消息
        
这一次，所有三个订阅客户机都应该收到消息。
        