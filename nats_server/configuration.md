# 配置文件格式

虽然NATS服务器有许多标志，允许对特性进行简单测试，但NATS服务器产品提供了一种灵活的配置格式，它结合了传统格式的优点和JSON和YAML等较新的样式。

NATS配置文件支持以下语法:

- 可以使用 `#` 和 `//` 加注释
- 值可以这样分配属性:
    - 等于符号: `foo = 2`
    - 冒号: `foo: 2`
    - 空格: `foo 2`
- 数组被括在括号内: `["a", "b", "c"]`
- 映射用大括号括起来: `{foo: 2}`
- Maps can be assigned with no key separator
- 分号可以用作终止符

### 字符串和数字

配置解析器是非常宽松的，正如您所看到的:
- 值可以是原语（数字、字符串）、列表或映射
- 字符串和数字通常做正确的事情

以数字开头的字符串值可能 _会_ 产生问题。要强制使用字符串之类的值，请使用引号括起来。
String values that start with a digit _can_ create issues. To force such values as strings, quote them.

*错误的配置*: 
```
listen: 127.0.0.1:4222
authorization: {
    # BAD!
    token: 3secret
}
```

正确的配置:
```
listen: 127.0.0.1:4222
authorization: {
    token: "3secret"
}
```

### 变量

服务器配置可以指定变量。变量允许您从配置中的一个或多个部分引用一个值。


变量:
- 是块作用域的
- 使用`$`前缀引用。
- 可以从具有相同名称的环境变量解析

> 如果环境变量的值以一个数字开头，那么根据您正在运行的服务器版本，您可能无法解析它。


```
# Define a variable in the config
TOKEN: "secret"

# Reference the variable
authorization {
    token: $TOKEN
}
```

类似的配置，但这一次，值在环境变量中:

```
# TOKEN is defined in the environment
authorization {
    token: $TOKEN
}
```

export TOKEN="hello"; nats-server -c /config/file

### Include 指令

The `include`指令允许您将服务器配置拆分为多个文件。这对于将配置分割成块非常有用，您可以在不同的服务器之间轻松地重用这些块。
             
Includes *必须* 使用相对路径，并且相对于主配置(通过`-c`选项指定的配置):

server.conf:
```
listen: 127.0.0.1:4222
include ./auth.conf
```

> 注意`include`后面没有`=`或`:`，因为它是一个_指令_。

auth.conf:
```
authorization: {
    token: "f0oBar"
}
```

```
> nats-server -c server.conf
```

### 配置属性

| 属性 | 描述 |
| :------  | :---- |
| [`authorization`](auth_intro.md) | 用于客户端身份验证/授权的配置映射 |
| [`cluster`](cluster_config.md) | 用于集群配置的配置映射 |
| `debug` | 如果为 `true` 则启用调试日志消息 |
| [`gateway`](/gateways/gateway.md) | 网关配置映射 |
| `host` | 用于客户端连接的主机 |
| [`http_port`](monitoring.md) | 用于服务器监控的 http 端口 |
| [`https_port`](monitoring.md) | 用于服务器监控的 https 端口 |
| [`leafnode`](/leafnodes/leafnode_conf.md) | 叶节点配置映射 |
| `listen`   | 客户端连接使用的 host/port |
| `max_connections` | 最大活动客户端连接数 |
| `max_control_line` | 协议行(包括主题长度)的最大长度 |
| `max_payload` | 消息有体中的最大字节数 |
| `max_pending` | 为连接缓冲的最大字节数 |
| `max_subscriptions` | 客户端连接的最大订阅数 |
| [`operator`](/nats_tools/nsc/nsc.md#nats-server-configuration) | 指向操作符 JWT 的路径 |
| [`ping_interval`](/developer/connecting/pingpong.md) | 间隔，以秒为单位，服务器隔多久检查连接是否是活动的|
| `port` | 用于客户端连接的端口号 |
| [`resolver`](/nats_tools/nsc/nsc.md#nats-server-configuration)  | Resolver type `MEMORY` or `URL` for account JWTs |
| [`tls`](tls.md#tls-configuration) | 用于客户端和http监控的tls配置映射 
| `trace` | 如果是 `true` 启用协议跟踪日志消息 |
| `write_deadline` | 当向客户端(慢消费者)写入时，服务器将阻塞的最大秒数 |


### 配置重新加载

服务器可以通过发送[signal](/nats_admin/signals.md)重新加载大多数配置更改，而不需要重启服务器或客户机断开连接:

```
> nats-server --signal reload
```
本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/configuration.md)