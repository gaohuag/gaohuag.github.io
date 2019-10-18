# 探索NATS请求/响应

NATS supports request/reply messaging. In this tutorial you explore how to exchange point-to-point messages using NATS.
NATS支持 请求/响应 模式的消息传递。在本教程中，您将探索如何使用nats交换点对点消息。

## 先决条件

Go和NATS服务器应该被安装。

### 1. 启动NATS服务器
      

```sh
% nats-server
```

### 2. 启动两个终端会话

您将使用这些会话来运行NATS请求和应答客户端。

### 3. 切换到examples目录

```sh
% cd $GOPATH/src/github.com/nats-io/nats.go/examples
```

### 4. 在一个终端中，运行响应客户端监听器

```sh
% go run nats-rply/main.go foo "this is my response"
```

您应该看到消息 `Receiver is listening`, 并且NATS接收方客户端正在监听 "help.please" 主题。响应客户端充当接收者，监听请求的消息。在NATS中，接收者是订阅者。

### 5. 在另一个终端中，运行请求客户端

```sh
% go run nats-req/main.go foo "request payload"
```

NATS请求者客户端在“help.please”主题里 发送消息 "some message"。

The NATS receiver client receives the message, formulates the reply ("OK, I CAN HELP!!!), and sends it to the inbox of the requester.
NATS响应客户端接收到消息，包装好回复的消息(“OK, I CAN HELP!!”)，并将其发送到请求者的收件箱。