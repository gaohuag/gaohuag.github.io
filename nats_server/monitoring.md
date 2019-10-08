## 监控 NATS

To monitor the NATS messaging system, `nats-server` provides a lightweight HTTP server on a dedicated monitoring port.
The monitoring server provides several endpoints, providing statistics and other information about the following:
为了监控 NATS消息传递系统，`nat-server`在专用的监听端口上提供了一个轻量级HTTP服务器。

监控服务器提供了几个端点，提供了以下方面的统计信息和其他信息:


* [通用服务器信息](#General-Information)
* [连接](#Connection-Information)
* [路由](#Route-Information)
* [订阅路由](#Subscription-Routing-Information)
* [网关](#Gateway-Information)

All endpoints return a JSON object.

The NATS monitoring endpoints support JSONP and CORS, making it easy to create single page monitoring web applications.
所有端点返回一个JSON对象。

NATS监听端点支持JSONP和CORS，这使得创建单页监视web应用程序变得很容易。

### 从命令行启用监控

要启用监控服务器，请使用监控标记`-m`和监控端口启动 NATS 服务器，或者在[配置文件](configuration.md#configuration-properties)中打开它。

    -m, --http_port PORT             HTTP PORT for monitoring
    -ms,--https_port PORT            Use HTTPS PORT for monitoring

Example:

```sh
$ nats-server -m 8222
[4528] 2019/06/01 20:09:58.572939 [INF] Starting nats-server version 2.0.0
[4528] 2019/06/01 20:09:58.573007 [INF] Starting http monitor on port 8222
[4528] 2019/06/01 20:09:58.573071 [INF] Listening for client connections on 0.0.0.0:4222
[4528] 2019/06/01 20:09:58.573090 [INF] nats-server is ready</td>
```

要进行测试，请运行 `nats-server -m 8222`, 然后跳转到 <a href="http://demo.nats.io:8222/" target="_blank">http://demo.nats.io:8222/</a>

### 从配置文件中启用监控

您还可以使用以下配置文件启用监控:

```yaml
http_port: 8222
```

例如，要在本地监视此服务器，端点应该是<a href="http://demo.nats.io:8222/varz" target="_blank">http://demo.nats.io:8222/varz</a>，
该接口暴露出各种一般统计数据。

## 监控终端

以下部分描述了每个受支持的监控端点: `varz`, `connz`, `routez`, `subsz`, 和 `gatewayz`。

没有任何参数是必需的，但是使用参数可以让您根据情况调整监控和工具。

### 一般信息

 `/varz` 端点返回关于服务器状态和配置的一般信息。

**Endpoint:** `http://server:port/varz`

| Result  | Return Code       |
|:---|:----|
| Success | 200 (OK)          |
| Error   | 400 (Bad Request) |

#### 参数

N/A

#### 举例

<a href="http://demo.nats.io:8222/varz" target="_blank">http://demo.nats.io:8222/varz</a>

#### 响应

```json
{
  "server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW",
  "version": "2.0.0",
  "proto": 1,
  "go": "go1.12",
  "host": "0.0.0.0",
  "port": 4222,
  "max_connections": 65536,
  "ping_interval": 120000000000,
  "ping_max": 2,
  "http_host": "0.0.0.0",
  "http_port": 8222,
  "https_port": 0,
  "auth_timeout": 1,
  "max_control_line": 4096,
  "max_payload": 1048576,
  "max_pending": 67108864,
  "cluster": {},
  "gateway": {},
  "leaf": {},
  "tls_timeout": 0.5,
  "write_deadline": 2000000000,
  "start": "2019-06-24T14:24:43.928582-07:00",
  "now": "2019-06-24T14:24:46.894852-07:00",
  "uptime": "2s",
  "mem": 9617408,
  "cores": 4,
  "cpu": 0,
  "connections": 0,
  "total_connections": 0,
  "routes": 0,
  "remotes": 0,
  "in_msgs": 0,
  "out_msgs": 0,
  "in_bytes": 0,
  "out_bytes": 0,
  "slow_consumers": 0,
  "subscriptions": 0,
  "http_req_stats": {
    "/": 0,
    "/connz": 0,
    "/gatewayz": 0,
    "/routez": 0,
    "/subsz": 0,
    "/varz": 1
  },
  "config_load_time": "2019-06-24T14:24:43.928582-07:00"
}
```

### 连接信息

`/connz` 端点暴露出关于当前和最近关闭的连接的更详细的信息。
它使用的分页机制默认为1024个连接。

**Endpoint:** `http://server:port/connz`

| Result  | Return Code       |
|:---|:---|
| Success | 200 (OK)          |
| Error   | 400 (Bad Request) |

#### 参数

| 参数 | 值 | 描述 |
|:---|:---|:---|
| sort   | (*see sort options*)     | 排序结果。默认是使用连接ID排序.          |
| auth   | true, 1, false, 0        | 包括用户名。默认不包含                   |
| subs   | true, 1, false, 0        | 包括订阅信息。默认不包含                 |
| offset | number > 0               | 分页偏移。默认值为0。                    |
| limit  | number > 0               | 返回的结果数目。默认是1024。             |
| cid    | number, valid id         | 通过id返回一个连接                       |
| state  | open, *closed,  any      | 返回连接的状态。默认是返回的。            |

*服务器将默认保存最后10,000个关闭的连接。*

##### 排序选项

| Option | Sort by|
|:---|:---|
|cid        | Connection ID                                        |
|start      | Connection start time, same as CID                   |
|subs       | Number of subscriptions                              |
|pending    | Amount of data in bytes waiting to be sent to client |
|msgs_to    | Number of messages sent                              |
|msgs_from  | Number of messages received                          |
|bytes_to   | Number of bytes sent                                 |
|bytes_from | Number of bytes received                             |
|last       | Last activity                                        |
|idle       | Amount of inactivity                                 |
|uptime     | Lifetime of the connection                           |
|stop       | Stop time for a closed connection                    |
|reason     | Reason for a closed connection                       |

#### 举例

最多1024个连接: <a href="http://demo.nats.io:8222/connz" target="_blank">http://demo.nats.io:8222/connz</a>

控制 limit 和 offset: <a href="http://demo.nats.io:8222/connz?limit=16&offset=128" target="_blank">http://demo.nats.io:8222/connz?limit=16&offset=128</a>.

关闭连接信息: <a href="http://demo.nats.io:8222/connz?state=closed" target="_blank">http://demo.nats.io:8222/connz?state=closed</a>.

您也可以使用subs=1查看订阅的详细信息。 例如: <a href="http://demo.nats.io:8222/connz?limit=1&offset=1&subs=1" target="_blank">http://demo.nats.io:8222/connz?limit=1&offset=1&subs=1</a>。


#### 响应

```json
{
  "server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW",
  "now": "2019-06-24T14:28:16.520365-07:00",
  "num_connections": 2,
  "total": 2,
  "offset": 0,
  "limit": 1024,
  "connections": [
    {
      "cid": 1,
      "ip": "127.0.0.1",
      "port": 49764,
      "start": "2019-06-24T14:27:25.94611-07:00",
      "last_activity": "2019-06-24T14:27:25.954046-07:00",
      "rtt": "275µs",
      "uptime": "50s",
      "idle": "50s",
      "pending_bytes": 0,
      "in_msgs": 0,
      "out_msgs": 0,
      "in_bytes": 0,
      "out_bytes": 0,
      "subscriptions": 1,
      "name": "NATS Sample Subscriber",
      "lang": "go",
      "version": "1.8.1",
      "subscriptions_list": [
        "hello.world"
      ]
    },
    {
      "cid": 2,
      "ip": "127.0.0.1",
      "port": 49767,
      "start": "2019-06-24T14:27:43.403923-07:00",
      "last_activity": "2019-06-24T14:27:43.406568-07:00",
      "rtt": "96µs",
      "uptime": "33s",
      "idle": "33s",
      "pending_bytes": 0,
      "in_msgs": 0,
      "out_msgs": 0,
      "in_bytes": 0,
      "out_bytes": 0,
      "subscriptions": 1,
      "name": "NATS Sample Subscriber",
      "lang": "go",
      "version": "1.8.1",
      "subscriptions_list": [
        "foo.bar"
      ]
    }
  ]
}
```

### 路由信息

The `/routez` endpoint reports information on active routes for a cluster.
Routes are expected to be low, so there is no paging mechanism with this endpoint.

**Endpoint:** `http://server:port/routez`

| Result  | Return Code       |
|:---|:---|
| Success | 200 (OK)          |
| Error   | 400 (Bad Request) |

#### Arguments

| Argument | Values | Description |
|:---|:---|:---|
| subs | true, 1, false, 0 | Include internal subscriptions.  Default is false.|

As noted above, the `routez` endpoint does support the `subs` argument from the `/connz` endpoint. For example: <a href="http://demo.nats.io:8222/routez?subs=1" target="_blank">http://demo.nats.io:8222/routez?subs=1</a>

#### Example

* Get route information:  <a href="http://demo.nats.io:8222/routez?subs=1" target="_blank">http://demo.nats.io:8222/routez?subs=1</a>

#### Response

```json
{
  "server_id": "NACDVKFBUW4C4XA24OOT6L4MDP56MW76J5RJDFXG7HLABSB46DCMWCOW",
  "now": "2019-06-24T14:29:16.046656-07:00",
  "num_routes": 1,
  "routes": [
    {
      "rid": 1,
      "remote_id": "de475c0041418afc799bccf0fdd61b47",
      "did_solicit": true,
      "ip": "127.0.0.1",
      "port": 61791,
      "pending_size": 0,
      "in_msgs": 0,
      "out_msgs": 0,
      "in_bytes": 0,
      "out_bytes": 0,
      "subscriptions": 0
    }
  ]
}
```

### Subscription Routing Information

The `/subz` endpoint reports detailed information about the current subscriptions and the routing data structure.  It is not normally used.

**Endpoint:** `http://server:port/subz`

| Result  | Return Code       |
|:---|:---|
| Success | 200 (OK)          |
| Error   | 400 (Bad Request) |

#### Arguments

| Argument | Values | Description |
|:---|:---|:---|
| subs   | true, 1, false, 0 | Include subscriptions.  Default is false.               |
| offset | integer > 0       | Pagination offset.  Default is 0.                       |
| limit  | integer > 0       | Number of results to return.  Default is 1024.          |
| test   | subject           | Test whether a subsciption exists.                      |

#### Example

* 获取路由信息:  <a href="http://demo.nats.io:8222/subsz" target="_blank">http://demo.nats.io:8222/subsz</a>

#### Response

```json
{
  "num_subscriptions": 2,
  "num_cache": 0,
  "num_inserts": 2,
  "num_removes": 0,
  "num_matches": 0,
  "cache_hit_rate": 0,
  "max_fanout": 0,
  "avg_fanout": 0
}
```

### Gateway Information

The `/gatewayz` endpoint reports information about gateways used to create a NATS supercluster.
Like routes, the number of gateways are expected to be low, so there is no paging mechanism with this endpoint.

**Endpoint:** `http://server:port/gatewayz`

| Result  | Return Code       |
|:---|:---|
| Success | 200 (OK)          |
| Error   | 400 (Bad Request) |

#### Arguments

| Argument | Values | Description |
|:---|:---|:---|
| accs     | true, 1, false, 0   | Include account information.  Default is false.  |
| gw_name  | string              | Return only remote gateways with this name.      |
| acc_name | string              | Limit the list of accounts to this account name. |

#### Examples

* Retrieve Gateway Information: <a href="http://demo.nats.io:8222/gatewayz" target="_blank">http://demo.nats.io:8222/gatewayz</a>

#### Response

```json
{
  "server_id": "NANVBOU62MDUWTXWRQ5KH3PSMYNCHCEUHQV3TW3YH7WZLS7FMJE6END6",
  "now": "2019-07-24T18:02:55.597398-06:00",
  "name": "region1",
  "host": "2601:283:4601:1350:1895:efda:2010:95a1",
  "port": 4501,
  "outbound_gateways": {
    "region2": {
      "configured": true,
      "connection": {
        "cid": 7,
        "ip": "127.0.0.1",
        "port": 5500,
        "start": "2019-07-24T18:02:48.765621-06:00",
        "last_activity": "2019-07-24T18:02:48.765621-06:00",
        "uptime": "6s",
        "idle": "6s",
        "pending_bytes": 0,
        "in_msgs": 0,
        "out_msgs": 0,
        "in_bytes": 0,
        "out_bytes": 0,
        "subscriptions": 0,
        "name": "NCXBIYWT7MV7OAQTCR4QTKBN3X3HDFGSFWTURTCQ22ZZB6NKKJPO7MN4"
      }
    },
    "region3": {
      "configured": true,
      "connection": {
        "cid": 5,
        "ip": "::1",
        "port": 6500,
        "start": "2019-07-24T18:02:48.764685-06:00",
        "last_activity": "2019-07-24T18:02:48.764685-06:00",
        "uptime": "6s",
        "idle": "6s",
        "pending_bytes": 0,
        "in_msgs": 0,
        "out_msgs": 0,
        "in_bytes": 0,
        "out_bytes": 0,
        "subscriptions": 0,
        "name": "NCVS7Q65WX3FGIL2YQRLI77CE6MQRWO2Y453HYVLNMBMTVLOKMPW7R6K"
      }
    }
  },
  "inbound_gateways": {
    "region2": [
      {
        "configured": false,
        "connection": {
          "cid": 9,
          "ip": "::1",
          "port": 52029,
          "start": "2019-07-24T18:02:48.76677-06:00",
          "last_activity": "2019-07-24T18:02:48.767096-06:00",
          "uptime": "6s",
          "idle": "6s",
          "pending_bytes": 0,
          "in_msgs": 0,
          "out_msgs": 0,
          "in_bytes": 0,
          "out_bytes": 0,
          "subscriptions": 0,
          "name": "NCXBIYWT7MV7OAQTCR4QTKBN3X3HDFGSFWTURTCQ22ZZB6NKKJPO7MN4"
        }
      }
    ],
    "region3": [
      {
        "configured": false,
        "connection": {
          "cid": 4,
          "ip": "::1",
          "port": 52025,
          "start": "2019-07-24T18:02:48.764577-06:00",
          "last_activity": "2019-07-24T18:02:48.764994-06:00",
          "uptime": "6s",
          "idle": "6s",
          "pending_bytes": 0,
          "in_msgs": 0,
          "out_msgs": 0,
          "in_bytes": 0,
          "out_bytes": 0,
          "subscriptions": 0,
          "name": "NCVS7Q65WX3FGIL2YQRLI77CE6MQRWO2Y453HYVLNMBMTVLOKMPW7R6K"
        }
      },
      {
        "configured": false,
        "connection": {
          "cid": 8,
          "ip": "127.0.0.1",
          "port": 52026,
          "start": "2019-07-24T18:02:48.766173-06:00",
          "last_activity": "2019-07-24T18:02:48.766999-06:00",
          "uptime": "6s",
          "idle": "6s",
          "pending_bytes": 0,
          "in_msgs": 0,
          "out_msgs": 0,
          "in_bytes": 0,
          "out_bytes": 0,
          "subscriptions": 0,
          "name": "NCKCYK5LE3VVGOJQ66F65KA27UFPCLBPX4N4YOPOXO3KHGMW24USPCKN"
        }
      }
    ]
  }
}
```

## 创建监控应用程序

NATS 监控端点支持 [JSONP](https://en.wikipedia.org/wiki/JSONP) 和 [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing#How_CORS_works). 
您可以轻松地创建用于监控的单页面web应用程序。为此，只需将`callback`查询参数传递给任何端点。

例如:

```sh
http://demo.nats.io:8222/connz?callback=cb
```

下面是一个JQuery示例实现:

```javascript
$.getJSON('http://demo.nats.io:8222/connz?callback=?', function(data) {
  console.log(data);
});

```

## 监控工具

除了编写定制的监控工具之外，您还可以在 Prometheus 中监视nat-server。
[Prometheus NATS出口商](https://github.com/nats-io/promethees-nats-exports)允许您配置希望在Prometheus观测和存储的指标。
您可以使用一个示例[Grafana](https://grafana.com)指示板来可视化服务器指标。