# 探索 NATS 队列

NATS支持使用队列组实现负载平衡。订阅者注册队列组名。组中随机选择一个订阅者来接收消息。

## 先决条件

Go和NATS服务器应该被安装。

### 1. 启动NATS服务器
       

```sh
nats-server
```

### 2. 为每个客户端示例克隆存储库

```sh
go get github.com/nats-io/nats.go
git clone https://github.com/nats-io/nats.js.git
git clone https://github.com/nats-io/nats.rb.git
```

### 3. 运行具有队列组名称的Go客户端订阅者

```sh
cd $GOPATH/src/github.com/nats-io/nats.go/examples
go run nats-qsub/main.go foo my-queue
```

### 4. 安装并运行具有队列组名称的node客户端订阅者

```sh
npm install nats
cd nats.js/examples
node node-sub --queue=my-queue foo
```

### 5. 安装并运行具有队列组名称的Ruby客户端订阅者

```sh
gem install nats
nats-queue foo my-queue &
```

### 6. 运行另一个 *没有* 队列组的Go客户端订阅者.

```sh
cd $GOPATH/src/github.com/nats-io/nats.go/examples
go run nats-sub/main.go foo
```

### 7. 使用Go客户端发布NATS消息

```sh
cd $GOPATH/src/github.com/nats-io/nats.go/examples
go run nats-pub/main.go foo "Hello NATS!"
```

### 8. 验证消息发布和接收

您应该看到发布者发送消息: *Published [foo] : 'Hello NATS!'*


您应该看到，my-queue组订阅者中只有一个收到消息。此外，不在my-queue组中的Go客户机订阅者也应该接收到了消息。
### 9. 发布另一个消息

```sh
go run nats-pub/main.go foo "Hello NATS Again!"
```

您应该看到，这次是在3个队列组成员中随机选择的另一个队列组订阅者接收消息。