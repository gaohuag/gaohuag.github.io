## 客户端协议

用于NATS服务器和客户端之间通信的wire协议是一个简单的、基于文本的发布/订阅样式协议。
客户端通过一个常规的TCP/IP套接字连接到“nat-server”(NATS服务器)并与之通信，使用新行终止协议操作。

与传统的消息传递系统使用二进制消息格式(需要使用API)不同，基于文本的NATS协议使得用各种编程和脚本语言实现客户端变得很容易。
实际上，请参考主题[NATS协议演示](nats-protocol-demo.md)，使用telnet亲自体验NATS协议。

NATS服务器实现了一个快速高效的[零分配字节解析器](https://youtu.be/ylRKac5kSOk?t=10m46s)。

## 协议约定

**控制行 w/可选内容**: 客户端与服务器端之间的每一次交互根据消息内容选择由一个控制或者协议，后面跟着文本行。
大部分的协议消息不需要内容，仅仅包含 `PUB` 和 `MSG` 有效负荷.

**字段分隔符**: NATS 协议消息的字段使用空白字符 '` `' (空格)  or `\t` (tab)分割。
 多个空白字符会被当做一个字符分隔符。

**新行**: NATS 使用 `CR` 跟着 `LF` (`CR+LF`, `\r\n`, `0x0D0A`) 终止协议消息。在 `PUB` or `MSG` 协议消息中新行序列也用来标识消息负荷结束.

**主题名称**: 主题名称, 包括回复主题(收件箱) 名称, 是大小写敏感的并且必须是非空的不包含空白字符的字母数字字符串。
所有的 ascii 字母数字字符除了 spaces/tabs 并且允许使用"." 和 ">" 分割主题。
例如:

`FOO`, `BAR`, `foo.bar`, `foo.BAR`, `FOO.BAR` 和 `FOO.BAR.BAZ` 都是有效的主题名

`FOO. BAR`, `foo. .bar` 和 `foo..bar` *不是* 有效的主题名


一个主题由一个或多个tokens组成。tokens由“.”分隔，可以是任何非空格的ascii字母数字字符。完整的通配符令牌“>”仅作为最后一个token有效，
并匹配超过该点的所有token。通配符“*”匹配单个token,匹配它所列出位置中的任何token。发布主题中不应该包换 "*"和“>"。

**字符编码**: 为了实现最大的互操作性，主题名称应该是ascii字符。由于语言和性能的限制，一些客户机可能支持UTF-8主题名，服务器也可能支持。不提供非ascii支持的保证。

**通配符**: NATS 支持在主题订阅中使用通配符。

- 星号字符 (`*`) 匹配主题中任意一级中的单个标记。
- 大于号 (`>`), 也称作 _全通配符_, 匹配一个主题后面一个或者多个标记, 并且必须是多个标记。通配符主题 `foo.>` 可以匹配 `foo.bar` 或者 `foo.bar.baz.1`, 但是不匹配 `foo`。 
- 通配符必须是一个单独的标识 (`foo.*.baz` 或者 `foo.>` 语法有效; `foo*.bar`, `f*o.b*r` 和 `foo>` 是无效的)

举个例子, 通配符 `foo.*.quux` 和 `foo.>` 都可以匹配 `foo.bar.quux`, 但是只有后面一个可以匹配 `foo.bar.baz`。  使用完整的通配符,
可能西药表达对NATS中每一个主题都感兴趣: `sub > 1`, 当然受限与认证设置。

## Protocol messages

下表简要描述了NATS协议消息。NATS协议操作名不区分大小写，因此`SUB foo 1\r\n` 和 `sub foo 1\r\n` 是等价的。

点击名称查看更详细的信息，包括语法:

| 操作名              | 发送方        |    描述
| -------------------- |:-------------- |:--------------------------------------------
| [`INFO`](#info)  | Server         | 在初始TCP/IP连接后发送到客户端
| [`CONNECT`](#connect)| Client         | 发送特殊的连接信息到服务器
| [`PUB`](#pub)     | Client         | 向主题发布一条消息，其中包含可选的回复主题
| [`SUB`](#sub)        | Client         | 订阅一个主题(或者是主题匹配符)
| [`UNSUB`](#unsub)    | Client         | 取消订阅 (或者自动取消订阅) 主题
| [`MSG`](#msg)        | Server         | 发布一个消息给订阅者
| [`PING`](#pingpong)  | Both           | PING 包活消息
| [`PONG`](#pingpong)  | Both           | PONG 包活响应
| [`+OK`](#okerr)      | Server         | 在“详细”模式下确认格式良好的协议消息
| [`-ERR`](#okerr)     | Server         | 指示协议错误。可能导致客户端断开连接。

下面几节解释每个协议消息。

## INFO

####  描述

一旦服务器接受来自客户端的连接，它就会发送关于自身的信息，以及客户端成功通过服务器身份验证和交换消息所需的配置和安全需求。

当使用更新的客户端协议时（请参阅下面的[`CONNECT`](#connect)），服务器可以随时发送 `INFO` 消息。这意味着具有该协议级别的客户端需要能够异步处理 `INFO` 消息。


####  语法

`INFO {["option_name":option_value],...}`

有效的选项如下:

- `server_id`: NATS服务器的唯一标识符
- `version`: NATS服务器的版本号
- `go`: 构建NATS服务器的golang版本号
- `host`: 用于启动NATS服务器的IP地址，默认为 `0.0.0.0`，可以配置为 `-client_advertise host:port`
- `port`: NATS服务器监听的端口号
- `max_payload`: 服务器将从客户端接受的最大有效负载大小，单位为字节。
- `proto`: 表示服务器的协议版本的整数。例如服务器版本1.2.0将其设置为“1”，以表示它支持“Echo”特性。
- `client_id`: 一个可选的无符号整数(64位)，表示服务器中的内部客户端标识符。这可以用来过滤监控中的客户端连接，记录错误日志用到，等等。
- `auth_required`: 如果设置了这个参数，那么客户机应该尝试在connect时进行身份验证。
- `tls_required`: 如果设置了这个，那么客户机必须执行TLS/1.2握手。注意，这曾经是`ssl_required`，并随着协议从SSL更新到TLS。
- `tls_verify`: 如果设置了这个，客户端必须在TLS握手期间提供一个有效的证书。
- `connect_urls` : 客户端可以连接到的可选服务器url列表。

##### connect_urls

`connect_urls` 字段是一个url列表，当客户端第一次连接时，当服务器集群拓扑结构发生更改时，服务器可能发送这些url。该字段被认为是可选的，
可以根据服务器配置和客户端协议级别省略。

当NATS服务器集群扩展时，会向客户机发送一条`INFO`消息，其中包含一个更新后的`connect_urls`列表。这个云友好特性异步地通知服务器已知的客户端，允许它连接到未初始配置的服务器。

`connect_urls`将包含一个带有IP和端口的字符串列表，如下所示: ```"connect_urls":["10.0.0.184:4333","192.168.129.1:4333","192.168.192.1:4333"]```

####  举例

下面您可以看到telnet连接到 `demo.nats.io` 网站的例子：

```sh
% telnet demo.nats.io 4222

Trying 107.170.221.32...
Connected to demo.nats.io.
Escape character is '^]'.
INFO {"server_id":"Zk0GQ3JBSrg3oyxCRRlE09","version":"1.2.0","proto":1,"go":"go1.10.3","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":2392}
```

## CONNECT

####  描述

`CONNECT`消息是客户端发送给服务器的[`INFO`](#INFO)消息。一旦客户端与NATS服务器建立了TCP/IP套接字连接，并且从服务器接收到一条[`INFO`](#INFO)消息，客户端就可以向NATS服务器发送一条`CONNECT`消息，以提供关于当前连接和安全信息的更多信息。

####  语法

`CONNECT {["option_name":option_value],...}`

有效的可选项如下：

* `verbose`: 打开 [`+OK`](#okerr) 协议确认。
* `pedantic`: 开启额外的严格格式检查，例如检查主题格式是否正确
* `tls_required`: 指示客户端是否需要SSL连接。
* `auth_token`: 客户端授权令牌(如果设置了`auth_required`)
* `user`: 连接用户名(如果设置了`auth_required`)
* `pass`: 连接密码(如果设置了`auth_required`)
* `name`: 可选的客户端名称
* `lang`: 客户机的实现语言。
* `version`: 客户端的版本。
* `protocol`: *可以选的整数*。发送`0`(或不发送)表示客户端支持原始协议。发送`1`表示客户端异步地接收[`INFO`](#INFO)消息来支持集群拓扑更改的动态重新配置，
这些消息包含了客户端可以重新连接到的服务器。
* `echo`: 可选布尔值。如果设置为“true”，服务器(版本1.2.0+)将不会从该连接发送原始消息到订阅者。
，当服务器支持此功能（即`info`协议中的`proto`设置大于`1`），客户端就应设置为“真”。

####  举例

下面是一个来自Go客户端的默认字符串的例子:

```
[CONNECT {"verbose":false,"pedantic":false,"tls_required":false,"name":"","lang":"go","version":"1.2.2","protocol":1}]\r\n
```

大多数客户默认将`verbose`设置为`false`。这意味着服务器不用返回客户端[`+OK`](#okerr)来确认它收到了消息。

## PUB

####  描述

`pub`指令会将消息发布到给定的主题名称，可选提供应答主题。如果提供了应答主题，它将连同提供的消息体一起分发给符合条件的订阅者。
注意，payload本身是可选的。要省略payload，请将payload大小设置为0，但是仍然需要CRLF（\r\n）。

####  语法

`PUB <subject> [reply-to] <#bytes>\r\n[payload]\r\n`

where:

- `subject`: 发布到哪个主题
- `reply-to`: 可选的回复收件箱主题，订阅者可以用来发送一个响应给发布者/请求者
- `#bytes`: 以字节为单位的有效负载大小
- `payload`: 消息体

####  举例

要将ASCII字符串消息体“Hello NATS!”发布到主题FOO:

`PUB FOO 11\r\nHello NATS!\r\n`

将请求消息“Knock - Knock”发布到主题FRONT.DOOR。同时设置回复主题为INBOX.22:

`PUB FRONT.DOOR INBOX.22 11\r\nKnock Knock\r\n`

向主题NOTIFY发布空消息:

`PUB NOTIFY 0\r\n\r\n`

## SUB

####  描述

`SUB` 启动对主题的订阅，可以选择加入分布式队列组。

####  语法

`SUB <subject> [queue group] <sid>\r\n`

where:

- `subject`: 订阅的主题名称
- `queue group`: 如果指定，订阅者将加入此队列组
- `sid`: 客户端生成的由字母数字组成的唯一订阅ID

####  举例

订阅主题 `FOO` ，指定唯一的连接订阅标识 (sid) `1`:

`SUB FOO 1\r\n`

使用分发队列组`G1`的一部分sid`44`订阅主题`BAR`:

`SUB BAR G1 44\r\n`

## UNSUB

####  描述

`UNPUB`取消指定主题的连接，或在收到指定数量的消息后自动取消订阅。
####  语法

`UNSUB <sid> [max_msgs]`

where:

* `sid`: 要取消订阅的主题的唯一字母数字订阅ID
* `max_msgs`: 等待的最大消息数量，之后就自动取消订阅

####  例子

下面的示例涉及主题`FOO`，它被分配给sid`1`。取消订阅`FOO`:

`UNSUB 1\r\n`

从 `FOO` 主题收到 5 条消息后自动取消订阅:

`UNSUB 1 5\r\n`

## MSG

####  描述

`MSG` 协议消息用于向客户端传递应用程序消息。

####  语法

`MSG <subject> <sid> [reply-to] <#bytes>\r\n[payload]\r\n`

where:

* `subject`: 收取消息的主题名
* `sid`: 主题的唯一字母数字组成的订阅ID
* `reply-to`: 发布者等待回复的收件箱主题
* `#bytes`: 消息体的大小(以字节为单位)
* `payload`: 消息体

####  例子

下面是发布消息到主题`FOO.BAR`:

`MSG FOO.BAR 9 11\r\nHello World\r\n`

下面是带回复主题的，内容和上面消息一样:

`MSG FOO.BAR 9 INBOX.34 11\r\nHello World\r\n`

## PING/PONG

####  描述

`PING`和`PONG`在客户端和服务器之间实现了一个简单的保持连接机制。一旦客户端建立了到 NATS 服务器的连接，服务器将以配置的时间
不断地向客户端发送`PING`消息。如果客户端未能在配置的响应间隔内响应`PONG`消息，服务器将终止连接。如果您的连接闲置太久，就会断开连接。

如果服务器发送ping请求，您可以使用pong消息进行回复，通知服务器您仍然感兴趣。您还可以ping服务器，并将收到一个pong回复。乒乓间隔是可配置的。

服务器使用普通流量作为ping/pong代理，因此具有消息流的客户端可能不会从服务器接收到ping。

####  语法

`PING\r\n`

`PONG\r\n`

####  举例

下面的示例显示了演示服务器ping客户端并最终关闭它。

```
telnet demo.nats.io 4222

Trying 107.170.221.32...
Connected to demo.nats.io.
Escape character is '^]'.
INFO {"server_id":"Zk0GQ3JBSrg3oyxCRRlE09","version":"1.2.0","proto":1,"go":"go1.10.3","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":2392}
PING
PING
-ERR 'Stale Connection'
Connection closed by foreign host.
```

## +OK/ERR

####  描述

When the `verbose` connection option is set to `true` (the default value), the server acknowledges each well-formed protocol 
message from the client with a `+OK` message. Most NATS clients set the `verbose` option to `false` using the [`CONNECT`](#connect) message

The `-ERR` message is used by the server indicate a protocol, authorization, or other runtime connection error to the client. 
Most of these errors result in the server closing the connection.

Handling of these errors usually has to be done asynchronously.

当`verbose`连接选项设置为`true`(默认值)时，服务器将用`+OK`消息确认来自客户端的每个格式良好的协议消息。大多数NATS客户端使用[`CONNECT`](#CONNECT)
消息将`verbose`选项设置为`false`

服务器使用的`-ERR`消息指示到客户端的协议、授权或其他运行时连接错误。大多数错误导致服务器关闭连接。

处理这些错误通常必须异步进行。

####  语法

`+OK`

`-ERR <error message>`

一些协议错误导致服务器关闭连接。接收到这些错误后，连接将不再有效，客户端应该清理相关资源。这些错误包括:

- `-ERR 'Unknown Protocol Operation'`: 未知协议错误
- `-ERR 'Attempted To Connect To Route Port'`: 客户端试图连接到路由端口而不是客户端端口
- `-ERR 'Authorization Violation'`: 客户端无法使用[`CONNECT`](#CONNECT)消息中指定的凭据对服务器进行身份验证
- `-ERR 'Authorization Timeout'`: 客户端在建立连接后花费太长时间对服务器进行身份验证(默认为1秒)
- `-ERR 'Invalid Client Protocol'`: 客户端在[`CONNECT`](#CONNECT)消息中指定了一个无效的协议版本
- `-ERR 'Maximum Control Line Exceeded'`: 消息目标主题和回复主题长度超过了服务器配置的`max_control_line`。默认值是1024字节。
- `-ERR 'Parser Error'`: 无法解析客户端发送的协议消息
- `-ERR 'Secure Connection - TLS Required'`:  服务器需要TLS，而客户机没有启用TLS。
- `-ERR 'Stale Connection'`: 服务器太长时间没有收到来自客户端的消息，包括`pong`。
- `-ERR 'Maximum Connections Exceeded`': 此错误由服务器在创建新连接时发送，服务器连接数已超过了服务器配置的最大连接数`max_connections`。默认值是64k。
- `-ERR 'Slow Consumer'`: 连接的服务器挂起的数据大小已达到最大值(默认为10MB)。
- `-ERR 'Maximum Payload Violation'`: 客户端试图发布一个消息体大小超过服务器上配置的`max_payload`大小的消息。
该值在连接初始[`INFO`](#INFO)消息时提供给客户端。客户端需要正确计算要发送到服务器的字节大小，以便同步处理此错误。

下面列出了仍然打开连接的协议错误消息。在这些情况下，客户端不应该关闭连接。

- `-ERR 'Invalid Subject'`: 客户端发送了一个格式不正确的主题 (e.g. `sub foo. 90`)
- `-ERR 'Permissions Violation for Subscription to <subject>'`: [`CONNECT`](#connect) 消息中指定的用户没有订阅该主题的权限。
- `-ERR 'Permissions Violation for Publish to <subject>'`: [`CONNECT`](#connect) 消息中指定的用户没有发布到该主题的权限。

本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_protocol/nats-protocol.md)
