## NATS Cluster Protocol
NATS服务器集群协议描述在[cluster](/nats_server/clustering.md)内的NATS服务器之间传递的协议，用于共享关于新服务器的帐户、订阅、转发消息和共享集群拓扑。它是一个简单的基于文本的协议。
服务器之间通过常规TCP/IP或TLS套接字进行通信，使用一组由换行终止的协议操作。

NATS服务器实现了一个快速高效的[零分配字节解析器](https://youtu.be/ylRKac5kSOk?t=10m46s)。

NATS cluster 协议与NATS客户端协议非常相似。 在cluster的上下文中，可以将服务器看作代表其连接的客户端操作的代理，订阅、取消订阅、发送和接收消息。

## NATS Cluster 协议约定

**主题名称与通配符**: NATS集群协议在主题名称和通配符方面具有与客户端相同的特性和限制。
客户端绑定到一个帐户，但是集群协议处理所有帐户。

**字段分隔符**: NATS协议消息的字段由空格字符'` `' (空格符)  or `\t` (tab)分隔。多个空白字符将被视为单个字段分隔符。

**新行**: 与其他基于文本的协议一样，NATS使用`CR`后跟`LF`(`CR+LF`, `\r\n`, `0x0D0A`)来终止协议消息。
这个换行符还用于在`RMSG`协议消息中标记实际消息payload的开始。

## NATS Cluster 协议消息

下表简要描述了NATS cluster 协议消息。
与在客户端协议中一样，NATS协议操作名不区分大小写，因此 `SUB foo 1\r\n` 与 `sub foo 1\r\n`是等价的。
点击名称查看更详细的信息，包括语法:


| OP Name              | Sent By          |    Description
| -------------------- |:-----------------|:--------------------------------------------
| [`INFO`](#info)      | All Servers      | 初始TCP/IP连接后发送并更新集群知识
| [`CONNECT`](#connect)| All Servers      | 建立连接
| [`RS+`](#sub)        | All Servers      | 代表感兴趣的客户端为给定帐户订阅主题。
| [`RS-`](#unsub)      | All Servers      | 取消对给定帐户的主题的订阅(或自动取消订阅)。
| [`RMSG`](#rmsg)      | Origin Server    | 将给定主题和帐户的消息传递给另一台服务器。
| [`PING`](#pingpong)  | All Servers      | PING 包活消息
| [`PONG`](#pingpong)  | All Servers      | PONG 包活回复消息
| [`-ERR`](#-err)      | All Servers      | 指示协议错误。可能导致远程服务器断开连接。


下面几节解释每个协议消息。

## INFO

#### 描述

一旦服务器接受来自另一台服务器的连接，它就会发送关于自己的信息，以及其他服务器成功通过服务器身份验证和交换消息所需的配置和安全需求。
连接服务器还发送一条`INFO`消息。接受服务器将添加一个`ip`字段，其中包含连接服务器的地址和端口，并将新服务器的`INFO`消息转发到它路由到的所有服务器。
集群中的任何服务器如果接收到带有`ip`字段的`INFO`消息，都将尝试连接到该地址的服务器，除非已经连接。这种代表连接服务器的
`INFO`消息传播提供了加入集群的新服务器的自动发现。

#### 语法

`INFO {["option_name":option_value],...}`

有效的选项如下:

* `server_id`: NATS 服务器的唯一标识符
* `version`: NATS服务器的版本
* `go`: 构建 NATS 服务器的 golang 版本
* `host`: 群集参数/选项中指定的主机
* `port`: 端口号
* `auth_required`: 如果设置了这个，那么服务器应该尝试在连接时进行身份验证。
* `tls_required`: 如果设置了这个参数，那么服务器必须使用TLS进行身份验证。
* `max_payload`: 服务器将接受的最大 payload 大小。
* `connect_urls` : 客户端可以连接到的服务器url列表。
* `ip`:  服务器的可选路由连接地址， `nats-route://<hostname>:<port>`

#### 举例

下面是一个由NATS服务器接收的带有`ip`字段的`INFO`字符串示例。

`INFO {"server_id":"KP19vTlB417XElnv8kKaC5","version":"2.0.0","go":"","host":"localhost","port":5222,"auth_required":false,"tls_required":false,"tls_verify":false,"max_payload":1048576,"ip":"nats-route://127.0.0.1:5222/","connect_urls":["localhost:4222"]}`

## CONNECT

#### 描述

`CONNECT`消息类似于[`INFO`](#INFO)消息。一旦 NATS 服务器与另一台服务器建立了TCP/IP连接，并且接收到一条[`INFO`](#INFO)消息，
服务器将发送一条`CONNECT`消息，以提供关于当前连接和安全信息的更多信息。

#### 语法

`CONNECT {["option_name":option_value],...}`

下面是有效的选项:

* `tls_required`: 指示服务器是否需要SSL连接。
* `auth_token`:  授权令牌
* `user`: 连接用户名(如果设置了`auth_required`)
* `pass`: 连接密码(如果设置了`auth_required`)
* `name`: 生成的服务器名
* `lang`: 服务器的实现语言(go).
* `version`: 服务器的版本。

#### 例子

下面是来自服务器的默认字符串的一个示例。

`CONNECT {"tls_required":false,"name":"wt0vffeQyoDGMVBC2aKX0b"}\r\n`

## <a name="SUB"></a>RS+

#### 描述
`RS+`在给定帐户上启动对主题的订阅，可以选择使用分布式队列组名称和权重因子。
注意，队列订阅将使用RS+来增加和减少队列权重，除非权重因子为0。
     
#### 语法

**订阅**: `RS+ <account> <subject>\r\n`

**队列订阅**: `RS+ <account> <subject> <queue> <weight>\r\n`

where:

* `account`: 主题兴趣关联的账户
* `subject`: 主题
* `queue`: 队列名称
* `weight`: 可选的队列组权重，表示有多少 兴趣/订阅者

## <a name="UNSUB"></a>RS-

#### 描述

`RS-` 给定账号上取消订阅指定主题。当服务器不再对给定的主题感兴趣时，就会发送它。

#### 语法

**Subscription**: `RS- <account> <subject>\r\n`

where:

* `account`: 主题的账号
* `subject`: 主题

## RMSG

#### Description

 `RMSG` 协议消息是发送消息给另外一个服务端

#### 语法

`RMSG <account> <subject> [reply-to] <#bytes>\r\n[payload]\r\n`

where:

* `account`: 主题的账号
* `subject`: 主题
* `reply-to`: 可选的回复主题
* `#bytes`: payload 的大小，单位是 bytes
* `payload`:  payload 数据

## PING/PONG

#### Description

`PING`和`PONG`在服务器之间实现了一个简单的包活机制。一旦两台服务器彼此建立连接，NATS服务器将以配置的时间间隔不断地向其他服务器发送`PING`消息。
如果其他服务器未能在配置的响应间隔内响应`PONG`消息，则服务器将终止其连接。如果您的连接闲置太久，就会被切断。

如果另一个服务器发送ping请求，服务器将使用pong消息进行响应，通知另一个服务器它仍然存在。
#### Syntax

`PING\r\n`
`PONG\r\n`

## -ERR

#### 描述

服务器使用`-ERR`消息指示到另一台服务器的协议、授权或其他运行时连接错误。大多数错误导致远程服务器关闭连接。



本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_protocol/nats-server-protocol.md)
