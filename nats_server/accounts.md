## 账户

_Accounts_ 在身份验证基础上展开。使用传统身份验证(JWT身份验证除外)，所有客户端都可以发布和订阅任何内容，除非显式地进行了其他配置。
为了保护客户和信息，您必须仔细地划分主题空间并谨慎地分配给客户端访问权限。
                       


_Accounts_ 允许对客户机进行分组，将它们与其他帐户中的客户端*隔离*，从而在服务器中启用 *multi-tenancy*。对于帐户，主题空间不是全局共享的，
这极大地简化了消息传递环境。客户无需设计复杂的主题名称雕刻模式，就可以使用短主题，而无需显式的授权规则。
帐户配置在`accounts`映射中完成。账户实体的内容包括:

| 属性 | 描述 |
| :-- | :-- |
| `users` | [user configuration maps](auth_intro.md#user-configuration-map)列表 |
| `exports` | 导出的映射列表 |
| `imports` | 导入的映射列表 |


`accounts` 列表是一个映射，其中映射上的键是一个帐户名称。

```
accounts: {
    A: {
        users: [
            {user: a, password: a}
        ]
    },
    B: {
        users: [
            {user: b, password: b}
        ]
    },
}
```
> 在上面最简单的配置中，您有一个名为`A`的帐户，其中只有一个用户名`a`和密码`a`标识的用户，还有一个帐户`B`，里面有一个用户名`B`和密码`b`的用户。

> 这两个帐户彼此独立。用户在`A`中发布的消息在`B`中不可见。

> 用户配置映射与任何其他NATS[用户配置映射](auth_intro.md#user-configuration-map)相同。您可以使用:

- username/password
- nkeys
- 还可以添加权限

> 虽然_account_意味着一个或多个用户，但是将一个帐户看作一个应用程序的消息容器要简单得多，也很有启发性。
帐户中的用户是必须一起工作以提供一些功能的最少的服务。
  
> 简单地说，与具有复杂授权配置的许多用户的一个大型帐户相比，拥有多个(甚至一个)客户端的帐户多是一种更好的设计拓扑。
### 导出和导入

通过从一个帐户中_exporting_流和服务并将它们导入到另一个帐户中，可以在不同帐户之间启用消息交换。每个帐户控制导出和导入的内容。

`exports`配置列表使您能够定义其它客户端可以导入的服务和流。服务和流表示为[Export configuration map](#export-configuration-map)。

### 流

流是应用程序发布的消息。导入应用程序不能从应用程序发出请求，但可以消费生成的消息。

### 服务 

服务是应用程序可以消费和处理的消息，允许其他帐户发出由帐户执行的请求。

### 导出配置映射

导出配置映射绑定一个主题作为“服务”或“流”使用，并可选地定义可以导入流或服务的特定帐户。以下是支持的配置属性:

| 属性 | 描述 |
| :-- | :-- |
| `stream` | 一个主题或带有通配符的主题，帐户将发布这些通配符(不含`service`)。 |
| `service` | 帐户将订阅的主题或带有通配符的主题(不包括 `stream`)。|
| `accounts` | 可以导入流或服务的帐户名列表。如果没有指定，服务或流是公共的，任何帐户都可以导入它。|

Here are some example exports:
```
accounts: {
    A: {
        users: [
            {user: a, password: a}
        ]
        exports: [
            {stream: puba.>}
            {service: pubq.>}
            {stream: b.>, accounts: [B]}
            {service: q.b, accounts: [B]}
        ]
    }
    ...
}
```

下面是`A`导出的:

- 通配符主题 `puba.>`的公共流
- 通配符主题 `pubq.>` 的公共服务
- 给账号 `B` 使用的流通配符主题`b.>` 
- 给账号 `B` 使用的服务主题`q.b` 


## 源配置映射

 _source configuration map_ 通过指定导入导出的“account”和“subject”来描述来自远程帐户的导出。
此映射嵌入在 [import configuration map](#import-configuration-map)中:

| 属性 | 描述 |
| :-- | :-- |
| `account` | 拥有导出的帐户名。|
| `subject` | 使导入帐户可以访问流或服务的主题 |


### 导入配置映射

导入允许帐户消费另一个帐户发布的流，或向另一个帐户实现的服务发出请求。所有导入都需要导出帐户上对应的导出。帐户不能自导入。


| 属性 | 描述 |
| :-- | :-- |
| `stream` | Stream import source configuration. (exclusive of `service`) |
| `service` | Service import source configuration (exclusive of `stream`) |
| `prefix`    | A local subject prefix mapping for the imported stream.|
| `to` | A local subject mapping for imported service. |

`prefix` 和 `to` 选项允许您重新映射本地用于接收流消息或发布服务请求的主题。

```
accounts: {
    A: {
        users: [
            {user: a, password: a}
        ]
        exports: [
            {stream: puba.>}
            {service: pubq.>}
            {stream: b.>, accounts: [B]}
            {service: q.b, accounts: [B]}
        ]
    },
    B: {
        users: [
            {user: b, password: b}
        ]
        imports: [
            {stream: {account: A, subject: b.>}}
            {service: {account: A, subject: q.b}}
        ]
    }
    C: {
        users: [
            {user: c, password: c}
        ]
        imports: [
            {stream: {account: A, subject: puba.>}, prefix: from_a}
            {service: {account: A, subject: pubq.C}, to: Q}
        ]
    }
}
```

Account `B` imports:

- the private stream from `A` that only `B` can receive on `b.>`
- the private service from `A` that only `B` can send requests on `q.b`

Account `C` imports the public service and stream from `A`, but also:

- remaps the `puba.>` stream to be locally available under `from_a.puba.>`. The messages will have their original subjects prefixed by `from_a`.
- remaps the `pubq.C` service to be locally available under `Q`. Account `C` only needs to publish to `Q` locally.

It is important to reiterate that:

- stream `puba.>` from `A` is visible to all external accounts that imports the stream.
- service `pubq.>` from `A` is available to all external accounts so long as they know the full subject of where to send the request. Typically an account will export a wildcard service but then coordinate with a client account on specific subjects where requests will be answered. On our example, account `C` access the service on `pubq.C` (but has mapped it for simplicity to `Q`).
- stream `b.>` is private, only account `B` can receive messages from the stream.
- service `q.b` is private; only account `B` can send requests to the service.
- When `C` publishes a request to `Q`, local `C` clients will see `Q` messages. However, the server will remap `Q` to `pubq.C` and forward the requests to account `A`.



本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/accounts.md)