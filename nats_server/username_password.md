## 用户名和密码

您可以使用用户名和密码验证一个或多个客户端;这使您能够更好地控制凭证机密的管理和发布。

对于单个用户:
```
authorization: {
    user: a,
    password: b
}
```

你也可以指定一个单一的用户名/密码:

```
> nats-server --user a --pass b
```

为多个用户:
```
authorization: {
    users: [
        {user: a, password: b},
        {user: b, password: a}
    ]
}
```

### 加密的密码

用户名/密码还支持使用[`mkpasswd`](/nats_tools/mkpasswd.md)工具加密的密码。只需将明文密码替换为加密的密码:

```
> mkpasswd
ass: (Uffs#rG42PAu#Oxi^BNng
bcrypt hash: $2a$11$V1qrpBt8/SLfEBr4NJq4T.2mg8chx8.MTblUiTBOLV3MKDeAy.f7u
```
配置文件:

```
authorization: {
    users: [
        {user: a, password: "$2a$11$V1qrpBt8/SLfEBr4NJq4T.2mg8chx8.MTblUiTBOLV3MKDeAy.f7u"},
        ...    
    ]
}
```

### 重新加载配置


当您从服务器配置文件中添加/删除密码时，您希望您的更改生效。要在不重启服务器和断开客户端连接的情况下重新加载，请执行以下操作:

```
> nats-server --signal reload
```