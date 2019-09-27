# Flags


NATS服务器有许多标记来定制其行为，而无需编写配置文件。

可选的标志围绕:

- Server Options
- Logging
- Authorization
- TLS Security
- Clustering
- Information


### Server Options

| Flag | Description |
| :-------------------- | :-------- |
| `-a`, `--addr` | Host address to bind to (default: `0.0.0.0` - all interfaces). |
| `-p`, `--port` | NATS client port (default: 4222). |
| `-P`, `--pid` | File to store the process ID (PID). |
| `-m`, `--http_port` | HTTP port for monitoring dashboard (exclusive of `--https_port`). |
| `-ms`, `--https_port` | HTTPS port monitoring for monitoring dashboard (exclusive of `--http_port`). |
| `-c`, `--config` | Path to NATS server configuration file. |
| `-sl`, `--signal` | Send a signal to nats-server process. See [process signaling](/nats_admin/signals.md). |
| `--client_advertise` | Client HostPort to advertise to other servers. |
| `-t` | 测试配置并退出 |



### 身份验证选项

以下选项控制直接的身份验证:
The following options control straightforward authentication:

| Flag | Description |
| :-------------------- | :-------- |
| `--user` | Required _username_ for connections (exclusive of `--token`). |
| `--pass` | Required _password_ for connections (exclusive of `--token`). |
| `--auth` | Required _authorization token_ for connections (exclusive of `--user` and `--password`). |

有关更多信息，请参见 [token authentication](tokens.md) 和 [username/password](username_password.md)。 


### 日志选项

在服务器上可以使用以下标志来配置日志:

| Flag | Description |
| :-------------------- | :-------- |
| `-l`, `--log` | File to redirect log output |
| `-T`, `--logtime` | Specify `-T=false` to disable timestamping log entries |
| `-s`, `--syslog` | Log to syslog or windows event log |
| `-r`, `--remote_syslog` | The syslog server address, like `udp://localhost:514` |
| `-D`, `--debug` | Enable debugging output |
| `-V`, `--trace` | Enable protocol trace log messages |
| `-DV` | 同时支持调试和协议跟踪消息 |

您可以阅读更多关于 [日志配置](logging.md) 的信息。


### TLS Options

| Flag | Description |
| :-------------------- | :-------- |
| `--tls` | Enable TLS, do not verify clients |
| `--tlscert` | Server certificate file |
| `--tlskey` | Private key for server certificate |
| `--tlsverify` | Enable client TLS certificate verification |
| `--tlscacert` | Client certificate CA for verification |


### Cluster Options

服务器上可用以下标志配置集群:


| Flag | Description |
| :-------------------- | :-------- |
| `--routes` | Comma-separated list of cluster URLs to solicit and connect |
| `--cluster` | Cluster URL for clustering requests |
| `--no_advertise` | Do not advertise known cluster information to clients |
| `--cluster_advertise` | Cluster URL to advertise to other servers |
| `--connect_retries` | For implicit routes, number of connect retries |

您可以阅读更多关于[集群配置](clustering.md)的信息。


### 常见的选项

| Flag | Description |
| :-------------------- | :-------- |
| `-h`, `--help` | Show this message |
| `-v`, `--version` | Show version |
| `--help_tls` | TLS help |

