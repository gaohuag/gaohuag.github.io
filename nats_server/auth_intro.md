# 身份验证

NATS服务器提供了多种认证客户端的方法:

- [Token Authentication](tokens.md)
- [Username/Password credentials](username_password.md)
- [TLS Certificate](tls_mutual_auth.md)
- [NKEY with Challenge](nkey_auth.md)
- [Accounts](accounts.md)
- [JWTs](jwt_auth.md)

身份验证处理允许NATS客户机连接到服务器。

除了JWT身份验证之外，身份验证和授权都是在配置的`authorization`部分配置的。

## 授权映射

 `authorization` 提供了 _authentication_ 和 _authorization_ 配置:

| 属性 | 描述 |
| :------  | :---- |
| [`token`](tokens.md) | 指定一个全局令牌，可用于对服务器进行身份验证(不能配置用户和密码了)|
| [`user`](username_password.md) | 为服务器的客户端指定一个_全局的_用户名(不能使用`token`了) |
| [`password`](username_password.md) | 为服务器的客户端指定一个_全局的_密码 (不能使用`token`了) |
| `users` | 用户配置映射列表 |
| `timeout` | 等待客户端验证的最大秒数 |

对于多个用户名和密码凭据，请指定一个`user`列表。


## 用户配置映射

 `user` 配置映射为单个用户指定凭据和权限选项:

| Property | Description |
| :------  | :---- |
| [`user`](username_password.md) | 用于客户端身份验证的`用户名` |
| [`password`](username_password.md) | 用于客户端身份验证的`密码 |
| [`nkey`](nkey_auth.md) | 标识用户的公钥 |
| [`permissions`](authorization.md) | 配置用户可访问的主题的权限映射 |

本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/auth_intro.md)