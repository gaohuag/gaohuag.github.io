## 令牌验证

身份验证的令牌是一个字符串，如果由客户端提供了，则允许它连接。这是NATS服务器提供的最直接的身份验证。


要使用令牌认证，您可以指定一个 `authorization` 部分，并设置 `token` 属性:
```
authorization {
    token: "s3cr3t"
}
```
令牌身份验证可用于客户端和集群的授权部分。

或启动服务器时，带上`--auth`参数:
```
> nats-server --auth s3cr3t
```

客户端可以通过指定服务器URL轻松连接:
```
> nats-sub -s nats://s3cr3t@localhost:4222 ">"
Listening on [>]
```

### 加密令牌

可以对令牌进行加密使得更加安全，因为令牌的明文版本不会持久存储在服务器配置文件中。

您可以使用[`mkpasswd`](/nats_tools/mkpasswd.md)工具生成 加密的 令牌和密码:


```
> mkpasswd
pass: dag0HTXl4RGg7dXdaJwbC8
bcrypt hash: $2a$11$PWIFAL8RsWyGI3jVZtO9Nu8.6jOxzxfZo7c/W0eLk017hjgUKWrhy
```


下面是一个简单的配置文件:
```
authorization {
    token: "$2a$11$PWIFAL8RsWyGI3jVZtO9Nu8.6jOxzxfZo7c/W0eLk017hjgUKWrhy"
}
```

客户端仍然需要明文令牌连接:

```
nats-sub -s nats://dag0HTXl4RGg7dXdaJwbC8@localhost:4222 ">"
Listening on [>]
```


