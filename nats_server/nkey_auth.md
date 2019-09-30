## NKey 身份验证

NKeys是一种基于[Ed25519](https://ed25519.cr.yp.to/)的新型、高度安全的公钥签名系统。


使用NKeys，服务器可以验证身份，而无需存储或查看私有密钥。身份验证系统要求连接的客户端提供其公钥，并使用其私钥对质询进行数字签名。
服务器为每个连接请求生成一个随机挑战，使其不受回放攻击的影响。生成的签名根据提供的公钥进行验证，从而验证客户机的身份。
如果服务器知道公钥，则身份验证成功。

> NKey是令牌身份验证的最佳替代方案，因为连接的客户端必须证明它控制了授权公钥的私钥。

要生成nkeys，您需要 [`nk` tool](/nats_tools/nk.md).


### 生成NKeys并配置服务器

生成一个 _User_ NKEY:

```
> nk -gen user -pubout
SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYCU3VY
UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4
```

第一个输出行以字母`S`开头，表示_Seed_。第二个字母U代表_User_。种子是私钥;你应该把它们当作秘密对待，小心地保护它们。

第二行以字母`U`开头，代表_User_，它是一个可以安全共享的公钥。

要使用nkey身份验证，请添加一个用户，并将`nkey`属性设置为要验证的用户的公钥:

```text
authorization: {
  users: [
    { nkey: UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4 }
  ]
}
```

Note that the user section sets the `nkey` property (user/password/token properties are not needed). Add `permission` sections as required.


### 客户端配置

现在您有了一个用户nkey，让我们配置一个客户端来使用它进行身份验证。例如，以下是nodejs客户端的连接选项:

```javascript
const NATS = require('nats');
const nkeys = require('ts-nkeys');

const nkey_seed = ‘SUACSSL3UAHUDXKFSNVUZRF5UHPMWZ6BFDTJ7M6USDXIEDNPPQYYYCU3VY’;
const nc = NATS.connect({
  port: PORT,
  nkey: 'UDXU4RCSJNZOIQHZNWXHXORDPRTGNJAHAHFRGZNEEJCPQTT2M7NLCNF4',
  sigCB: function (nonce) {
    // client loads seed safely from a file
    // or some constant like `nkey_seed` defined in
    // the program
    const sk = nkeys.fromSeed(Buffer.from(nkey_seed));
    return sk.sign(nonce);
   }
});
...
```

The client provides a function that it uses to parse the seed (the private key) and sign the connection challenge.
客户端提供一个用于解析种子(私钥)和签署连接挑战的函数。

