# 认证超时

很像 [`tls timeout` 选项](/nats_server/tls.md#tls-timeout)，身份验证可以指定一个`timeout`值。
如果客户机没有在指定的时间内对服务器进行身份验证，服务器将断开服务器的连接以防止滥用。



超时以秒为单位指定(可以是小数)。

如果客户端没有在指定的时间内对服务器进行身份验证，服务器将断开服务器的连接以防止滥用。

超时以秒为单位指定(可以是小数)。

与TLS超时一样，长时间的超时可能是滥用的机会。如果设置了身份验证超时，一定要注意它应该比`tls timeout`设置的要长，因为身份验证过程包括了tls升级的时间。
```
authorization: {
    timeout: 3
    users: [
        {user: a, password b},
        {user: b, password a}
    ]
}
```


本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_server/auth_timeout.md)