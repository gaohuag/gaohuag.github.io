## 协议演示

当您体验到使用NATS是多么容易时，NATS协议的优点很快就会体现出来。因为NATS协议是基于文本的，所以您可以跨几乎任何平台或语言使用NATS。
在下面的演示中，我们使用[Telnet](https://en.wikipedia.org/wiki/Telnet)。

在网络上，您可以使用一个简单的[协议命令集](nats-protocol.md)发布和订阅。


## 指令

**1. 打开一个终端会话**

您将使用这个终端作为订阅者。

**2. 连接到NATS。**

```
telnet demo.nats.io 4222
```

期望的结果:

```
$ telnet demo.nats.io 4222
Trying 107.170.221.32...
Connected to demo.nats.io.
Escape character is '^]'.
INFO {"server_id":"NCXMJZYQEWUDJFLYLSTTE745I2WUNCVG3LJJ3NRKSFJXEG6RGK7753DJ","version":"2.0.0","proto":1,"go":"go1.11.10","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":5089}
```

**3. 订阅主题**

订阅通配符主题 `foo.*`，使用主题ID `90`.

```
sub foo.* 90
```

服务器端返回: `+OK` 表示已成功登记。

```
sub foo.* 90
+OK
```

**4. 再打开一个终端**

您将为发布者使用这个终端。

**5. 连接 NATS.**

```
telnet demo.nats.io 4222
```

期望结果:

```
$ telnet demo.nats.io 4222
Trying 107.170.221.32...
Connected to demo.nats.io.
Escape character is '^]'.
INFO {"server_id":"NCXMJZYQEWUDJFLYLSTTE745I2WUNCVG3LJJ3NRKSFJXEG6RGK7753DJ","version":"2.0.0","proto":1,"go":"go1.11.10","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":5089}
```
**6. 发布一条消息**

消息包括命令(`pub`)、主题(`foo.bar`)和payload的长度(`5`)。按回车并提供payload(`hello`)，然后再次按回车。

```
pub foo.bar 5
hello
```

发布结果: `+OK` 指示消息发布成功。

```
pub foo.bar 5
hello
+OK
```

Subscriber result: `MSG` + subject name + subscription ID + message payload size + message payload `hello`.
订阅者得到的结果:`MSG` + 主题名 + 订阅ID + payload大小 + payload `hello`。

```
sub foo.* 90
+OK
MSG foo.bar 90 5
hello
```

**7. 发布另外一条带回复主题的消息**

```
pub foo.bar optional.reply.subject 5
hello
+OK
```

订阅结果:`MSG`，表示收到消息。

```
MSG foo.bar 90 optional.reply.subject 5
hello
```

**8. 取消订阅的主题**

您可以使用`UNSUB`命令取消订阅消息。

运行订阅程序来取消订阅:

```
unsub 90 
```

订阅者结果:“+OK”表示已成功撤销订阅。

```
unsub 90
+OK
```

**9. 重新连接到服务器并订阅。**

```
telnet demo.nats.io 4222
```

```
sub foo.* 90
```

**10. 关于ping/pong 时间间隔**

如果您将telnet会话打开几分钟，您可能会注意到您的客户端收到来自服务器的`ping`请求。
如果您的客户端不活动，或者在ping/pong间隔内没有响应服务器ping信号，服务器将断开客户端的连接。错误消息是`-ERR 'Stale Connection'`。

你可以向服务器发送一个`ping`请求，然后收到一个`PONG`回复。例如:

```
$ telnet demo.nats.io 4222
Trying 107.170.221.32...
Connected to demo.nats.io.
Escape character is '^]'.
INFO {"server_id":"NCXMJZYQEWUDJFLYLSTTE745I2WUNCVG3LJJ3NRKSFJXEG6RGK7753DJ","version":"2.0.0","proto":1,"go":"go1.11.10","host":"0.0.0.0","port":4222,"max_payload":1048576,"client_id":5089}

ping
PONG
```


本人翻译水平有限，读者朋友实在看不懂了，请参考[原文](https://github.com/nats-io/docs/blob/master/nats_protocol/nats-protocol-demo.md)