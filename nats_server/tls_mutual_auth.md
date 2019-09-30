## Client TLS Mutual Authentication

The server can require TLS certificates from a client. When needed, you can use the certificates to:

- Validate the client certificate matches a known or trusted CA
- Extract information from a trusted certificate to provide authentication

### 验证客户端证书

服务器可以使用CA证书验证客户端证书。如需验证，请在TLS配置部分添加  `verify`  选项，如下所示:

```
tls {
  cert_file: "./configs/certs/server-cert.pem"
  key_file:  "./configs/certs/server-key.pem"
  ca_file:   "./configs/certs/ca.pem"
  verify:    true
}
```

或通过命令行:

```sh
> ./nats-server --tlsverify --tlscert=./test/configs/certs/server-cert.pem --tlskey=./test/configs/certs/server-key.pem --tlscacert=./test/configs/certs/ca.pem
```

This option verifies the client's certificate is signed by the CA specified in the `ca_file` option. 
此选项验证客户端证书是由`ca_file`选项中指定的CA签署的。


### 将客户端证书映射到用户

除了验证指定的CA颁发了客户端证书外，还可以使用证书中编码的信息对客户端进行身份验证。客户端不必提供或跟踪用户名或密码。

将TLS互认证映射证书属性设置为用户身份使用`verify_and_map`，如下图所示:
```
tls {
  cert_file: "./configs/certs/server-cert.pem"
  key_file:  "./configs/certs/server-key.pem"
  ca_file:   "./configs/certs/ca.pem"
  # Require a client certificate and map user id from certificate
  verify_and_map: true
}
```

> 注意， ``verify` 被更改为 `verify_and_map`.

证书属性有两个选项可以映射到用户名。第一个是证书的Subject Alternative Name (SAN)字段中的电子邮件地址。
虽然生成具有此属性的证书超出了本文档的范围，但是您可以使用“openssl”查看证书:

```
$ openssl x509 -noout -text -in  test/configs/certs/client-id-auth-cert.pem
Certificate:
...
        X509v3 extensions:
            X509v3 Subject Alternative Name:
                DNS:localhost, IP Address:127.0.0.1, email:derek@nats.io
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
...
```

授权该用户的配置如下:
```
authorization {
  users = [
    {user: "derek@nats.io"}
  ]
}
```

> 注意:如果SAN字段中有多个电子邮件，此配置仅对第一个电子邮件地址有效。
  
第二个选项是使用来自证书主题的RFC 2253专有名称语法，如下所示

```
$ openssl x509 -noout -text -in  test/configs/certs/tlsauth/client2.pem
Certificate:
    Data:
...
        Subject: OU=CNCF, CN=example.com
...
```

授权该用户的配置如下:

```
authorization {
  users = [
    {user: "CN=example.com,OU=CNCF"}
  ]
}
```

### TLS 超时

[TLS 超时](/nats_server/tls.md#tls-timeout) 在这儿描述