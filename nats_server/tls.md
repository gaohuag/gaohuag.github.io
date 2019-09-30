## TLS 配置

NATS服务器使用现代TLS语义来加密客户端、路由和监控连接。

服务器配置围绕一个`tls`映射，它有以下属性:
| 属性 | 描述 |
| :------  | :---- |
| `ca_file` | TLS certificate authority file. |
| `cert_file` | TLS certificate file. |
| `cipher_suites` | When set, only the specified TLS cipher suites will be allowed. Values must match the golang version used to build the server.  |
| `curve_preferences` | List of TLS cipher curves to use in order. |
| `insecure` | Skip certificate verification. |
| `key_file` | TLS certificate key file. |
| `timeout` | TLS handshake timeout in fractional seconds. |
| `verify_and_map` | If `true`, require and verify client certificates and map certificate values for authentication purposes. |
| `verify` | If `true`, require and verify client certificates. |

最简单的配置:
```
tls: {
  cert_file: "./server-cert.pem"
  key_file: "./server-key.pem"
}
```
或使用[服务器选项](./flags.md#tls-options):
```
> nats-server --tls --tlscert=./server-cert.pem --tlskey=./server-key.pem
[21417] 2019/05/16 11:21:19.801539 [INF] Starting nats-server version 2.0.0
[21417] 2019/05/16 11:21:19.801621 [INF] Git commit [not set]
[21417] 2019/05/16 11:21:19.801777 [INF] Listening for client connections on 0.0.0.0:4222
[21417] 2019/05/16 11:21:19.801782 [INF] TLS required for client connections
[21417] 2019/05/16 11:21:19.801785 [INF] Server id is ND6ZZDQQDGKYQGDD6QN2Y26YEGLTH6BMMOJZ2XJB2VASPVII3XD6RFOQ
[21417] 2019/05/16 11:21:19.801787 [INF] Server is ready
```

请注意，日志表明需要客户端连接后才能使用TLS。如果您在调试模式下运行服务器与`-D`或`-DV`，日志将显示为每个连接的客户端选择的加密方法:

```
[22242] 2019/05/16 11:22:20.216322 [DBG] 127.0.0.1:51383 - cid:1 - Client connection created
[22242] 2019/05/16 11:22:20.216539 [DBG] 127.0.0.1:51383 - cid:1 - Starting TLS client connection handshake
[22242] 2019/05/16 11:22:20.367275 [DBG] 127.0.0.1:51383 - cid:1 - TLS handshake complete
[22242] 2019/05/16 11:22:20.367291 [DBG] 127.0.0.1:51383 - cid:1 - TLS version 1.2, cipher suite TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

当在配置的根目录中指定`tls`部分时，监控端口将使用`https_port`。其他部分，如`cluster`，可以指定`tls`块。


### TLS 超时
“超时”设置使您能够控制允许客户端连接升级到tls的时间。如果您的客户端在TLS握手期间正在经历断开连接，那么您将希望增加该值，
但是，如果您确实意识到扩展的“超时”将使您的服务器受到攻击，其中客户端没有升级到TLS，因此会消耗资源。相反，如果过多地减少TLS“超时”，
则很可能出现握手错误。

```
tls: {
  cert_file: "./server-cert.pem"
  key_file: "./server-key.pem"
  # clients will fail to connect (value is too low)
  timeout: 0.0001
}
```