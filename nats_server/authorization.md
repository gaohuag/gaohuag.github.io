## 授权

NATS服务器支持在每个用户的基础上使用主题级权限进行授权。基于许可的授权可以通过`users`列表进行多用户身份验证。

每个权限指定用户可以发布和订阅的主题。解析器在理解意图方面很慷慨，因此数组和单例都被处理。对于更复杂的配置，可以指定一个“permission”对象，
该对象显式地允许或拒绝主题。指定的主题可以指定通配符。权限可以使用[variables](configuration.md#variables)。

通过在“授权”对象中创建“权限”条目来配置授权。

### 权限配置映射

“权限”映射指定可以由指定客户端订阅或发布的主题。

| 属性 | 描述 |
| :------  | :---- |
| `publish` | 客户端可以发布的主题、主题列表或权限映射 |
| `subscribe` | 客户端可以订阅的主题、主题列表或权限映射 |

### 权限映射表

The `permission` map provides additional properties for configuring a `permissions` map. 
Instead of providing a list of subjects that are allowed,
 the `permission` map allows you to explicitly list subjects you want to`allow` or `deny`:

| Property | Description |
| :------  | :---- |
| `allow` | 允许客户端使用的主题名称列表 |
| `deny` | 被客户端拒绝的主题列表 |



**重要提示** NATS授权可以是 _allow 列表_, _deny 列表_, 或者两种都包括。
 It is important to not break request/reply patterns. In some cases (as shown below) you need to add rules as above with Alice and Bob for the `_INBOX.>` pattern. 
 
 如果未经授权的客户端发布或订阅未被_允许_的主题，则操作将失败并被记录在服务器上，并向客户端返回一条错误消息。

### 例子

下面是一个使用_变量_的授权配置示例，它定义了四个用户，其中三个用户被分配了权限。

```ascii
authorization {
  ADMIN = {
    publish = ">"
    subscribe = ">"
  }
  REQUESTOR = {
    publish = ["req.a", "req.b"]
    subscribe = "_INBOX.>"
  }
  RESPONDER = {
    subscribe = ["req.a", "req.b"]
    publish = "_INBOX.>"
  }
  DEFAULT_PERMISSIONS = {
    publish = "SANDBOX.*"
    subscribe = ["PUBLIC.>", "_INBOX.>"]
  }
  users = [
    {user: admin,   password: $ADMIN_PASS, permissions: $ADMIN}
    {user: client,  password: $CLIENT_PASS, permissions: $REQUESTOR}
    {user: service,  password: $SERVICE_PASS, permissions: $RESPONDER}
    {user: other, password: $OTHER_PASS}
  ]
}
```

> *DEFAULT_PERMISSIONS* 一个特殊的权限名称。如果已定义，则它适用于没有特定权限集的所有用户。

- _admin_ 拥有 `ADMIN` 权限，可以发布/订阅任何主题。我们使用通配符`>`匹配任何主题。

- _client_ 是一个 `REQUESTOR` 可以在主题 `req.a` 或者 `req.b`上发布请求。并订阅主题 (`_INBOX.>`)的任何响应。

- _service_ 是一个响应`req.a` 和 `req.b`请求的 `RESPONDER`, 因此它需要能够订阅请求主题并响应发布请求到 `req.a` 和 `req.b`的客户端。
回复主题是收件箱。通常收件箱名称是以前缀`_INBOX.`开始，后面跟着一些字符串。  `_INBOX.>`匹配所有以`_INBOX.`开头的主题。


- _other_ 没有授予权限，因此继承默认权限集。通过将继承的默认权限分配给授权配置块内的 `default_permissions` 条目来设置默认权限。

> 注意，在上面的示例中，任何可以订阅`_INBOX.>`的客户端可以接收所有已发布的响应。权限要求严格的情况希望是主题的前缀，
以进一步限制客户端可以订阅的主题。或者，[_Accounts_](accounts.md)允许完全隔离，限制帐户成员可以看到的内容。

本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/authorization.md)
