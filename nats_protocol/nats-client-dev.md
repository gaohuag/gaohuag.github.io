## NATS 客户端开发指南

本指南提供了开发 NATS 客户端的注意事项，包括:

- 连接处理
- 认证
- 详细模式 (acks)
- 迂腐模式
- Ping/pong 间隔
- 解析协议
- 确定解析策略
- 存储和分发订阅回调
- 实现请求和响应
- 错误处理，断开连接和重新连接
- 集群支持

学习实现一个客户端最好的方式是学习由 Synadia 团队维护的任何一个客户端。
这些客户端通常功能齐全，所以如果你能使用它们，那就更好了，

但是如果你必须编写一个客户端，这些可能会超出你的需要，同时仍然捕获了许多

这里讨论设计的注意事项。

- [go](https://github.com/nats-io/nats.go)
- [node](https://github.com/nats-io/nats.js)
- [typescript](https://github.com/nats-io/nats.ts)
- [python2](https://github.com/nats-io/nats.py2)
- [python asyncio](https://github.com/nats-io/nats.py)
- [java](https://github.com/nats-io/nats.java)
- [c#](https://github.com/nats-io/nats.net)
- [ruby](https://github.com/nats-io/nats.rb)
- [c](https://github.com/nats-io/nats.c)

## 客户端连接选项

客户端可以使用认证或者不认证模式连接, 还有启动确认的详细模式. 有关详细信息，请参阅[协议文档](/nats_protocol/nats-protocol.md#connect)。

## 启动确认

默认情况下，客户端可以以未经身份验证的模式连接到服务器。您可以将NATS服务器配置为需要密码身份验证才能连接。



例如，使用命令行:

```
nats-server -DV -m 8222 -user foo -pass bar
```

然后，客户机必须进行身份验证才能连接到服务器。例如:

```
nats.Connect("nats://foo:bar@localhost:4222")
```

## 详细模式

当启用“详细”(在“CONNECT”消息中启用)时，NATS服务器将返回“+OK”以确认收到了有效的协议消息。NATS服务器自动以详细模式运行。
由于性能原因，大多数客户端实现禁用了详细模式(在“CONNECT”消息中将其设置为“false”)。


## 迂腐模式

客户端也可能支持“迂腐”模式。迂腐模式向服务器表明需要严格的协议执行。

## Ping/pong 时间间隔

NATS实现请。当客户端连接到服务器时，服务器期望该客户机是活动的。每隔一段时间，NATS服务器就会ping每个订阅者，等待一个回复。
如果在配置的超时时间内没有响应，服务器将断开客户端的连接。

## 解析协议

NATS提供了一种基于文本的消息格式。基于文本的[协议](/documentation/internal als/nat -protocol/)使实现NATS客户端变得很容易。
关键的考虑是决定一个解析策略。

NATS服务器实现了一个快速高效的[零分配字节解析器](https://youtu.be/ylRKac5kSOk?t=10m46s)。离线时，NATS消息是字节slice。
在网络上，消息通过TCP连接以不可变字符串的形式传输。由客户端实现解析消息的逻辑。



NATS消息结构包括主题字符串、一个可选的回复字符串和一个可选的数据字段(字节数组)。类型“Msg”是订阅者和PublishMsg()使用的结构。

```
type Msg struct {
    Subject string
    Reply   string
    Data    []byte
    Sub     *Subscription
}
```
NATS发布者将数据参数发布到给定的主题。数据参数保持不变，需要在接收端正确解释。客户端如何解析 NATS 消息取决于编程语言。

## 决定解析策略

通常，NATS客户端的协议解析是一个字符串操作。例如，在Python中，字符串操作比正则表达式更快。Go和Java客户机还使用字符串操作来解析消息。
但是Ruby客户端使用正则解析协议，因为在Ruby中，正则比字符串操作更快。

总之，不同的编程语言使用不同的解析方法。所以，在编写客户端时需要考虑如何解析消息。

## 存储和分发订阅回调
  
在订阅服务时，需要存储和分发回调处理程序。在客户端，您需要这个数据结构的哈希映射。哈希映射将存储订阅ID映射到订阅的回调。
   
哈希映射的键是订阅ID。该键用于在哈希映射中查找回调。在离线处理NATS消息时， 将参数subject、reply subject和有效数据传递给回调处理程序，
回调处理程序执行其任务。因此，必须存储订阅ID到回调函数的映射。在订阅中有回调。
  
## 实现请求/响应
   
何时使用发布/订阅和请求/响应取决于您的用例。运行这些教程，了解每种实现风格之间的差异。

## 错误处理、断开连接和重新连接

错误处理的注意事项主要包括处理客户端断开连接和实现重试逻辑。

## 集群支持

NATS客户端具有重连接逻辑。因此，如果您正在实现集群，则需要预先实现重连回调，这意味着您不能在运行时修改它。当您开始时，您需要已经拥有这些信息。


本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_protocol/nats-client-dev.md)