
## Accounts

_Accounts_ 基于 [Accounts](accounts.md) 和 [NKeys](nkey_auth.md) 身份验证基础上进行扩展，以创建分散的身份验证和授权模型。

With other authentication mechanisms, configuration for identifying a user or an account is in the server configuration file. JWT authentication leverages [JSON Web Tokens (JWT)](https://jwt.io/) to describe the various entities supported. When a client connects, servers query for account JWTs and validate a trust chain. Users are not directly tracked by the server, but rather verified as belonging to an account. This enables the management of users without requiring server configuration updates.

Effectively, accounts provide for a distributed configuration paradigm. 
Previously each user (or client) needed to be known and authorized a priori in the server’s configuration requiring an administrator to modify and update server configurations. 
Accounts eliminate these chores.

对于其他身份验证机制，用于标识用户或帐户的配置位于服务器配置文件中。JWT身份验证利用[JSON Web令牌(JWT)](https://jwt.io/)来描述所支持的各种实体。
当客户端连接时，服务器查询帐户JWTs并验证信任链。服务器并不直接跟踪用户，而是将其验证为属于某个帐户。
这样就可以在不需要更新服务器配置的情况下管理用户。

实际上，帐户提供了一个分布式配置范例。以前，在需要管理员修改和更新服务器配置的服务器配置中，需要预先知道和授权每个用户(或客户机)。
帐户消除这些杂务。

### JSON Web Tokens

[JSON Web Tokens (JWT)](https://jwt.io/) 是一个开放的行业标准[RFC7519](https://tools.ietf.org/html/rfc7519)的方法来代表双方之间的索赔安全。

索赔是对一个主题的信息进行索赔的一种奇特的方式。在这个上下文中，_subject_是被描述的实体(而不是消息传递主题)。
标准的JWT索赔要求通常经过数字签名和验证。


国家运输安全委员会进一步限制JWTs，规定JWTs必须:
- Digitally signed _always_ and only using [Ed25519](https://ed25519.cr.yp.to/). 
- NATS adopts the convention that all _Issuer_ and _Subject_ fields in a JWT claim must be a public [NKEY](nkey_auth.md). 
- _Issuer_ and _Subject_ must match specific roles depending on the claim [NKeys](https://github.com/nats-io/nkeys).

#### NKey Roles

NKey Roles are:

- Operators
- Accounts
- Users

角色是分层的，并形成信任链。运营商发行账户，而账户又反过来发行用户。服务器信任特定的操作员。如果帐户是由受信任的操作人员颁发的，
则帐户用户是受信任的。

Roles are hierarchical and form a chain of trust. 
Operators issue Accounts which in turn issue Users. Servers trust specific Operators. If an account is issued by an operator that is trusted, account users are trusted.


### 身份验证过程

When a _User_ connects to a server, it presents a JWT issued by its _Account_. The user proves its identity by signing a server-issued cryptographic challenge with its private key. The signature verification validates that the signature is attributable to the user's public key. Next, the server retrieves the associated account JWT that issued the user. It verifies the _User_ issuer matches the referenced account. Finally, the server checks that a trusted _Operator_ issued the _Account_, completing the trust chain verification. 
当_User_连接到服务器时，它会显示由_Account_发出的JWT。用户通过使用其私钥签署服务器发出的密码挑战来证明其身份。签名验证验证签名是否属于用户的公钥。
接下来，服务器检索发给用户的关联帐户JWT。它验证_User_发行者是否与引用的帐户匹配。最后，服务器检查可信_Operator_是否发出了_Account_，
从而完成信任链验证。


### 授权过程
从授权的角度来看，该帐户提供关于从其他帐户导入的消息传递主题(包括任何辅助相关授权)以及导出到其他帐户的消息传递主题的信息。
帐户也有限制，比如它们可能拥有的最大连接数。用户JWT可以表达对其可以发布或订阅的消息传递主题的限制。
当一个新用户被添加到一个帐户时，帐户配置不需要更改，因为每个用户都可以而且应该有自己的用户JWT，可以通过解析其父帐户来验证该用户。


From an authorization point of view, the account provides information on messaging subjects that are imported from other accounts (including any ancillary related authorization) as well as messaging subjects exported to other accounts. Accounts can also bear limits, such as the maximum number of connections they may have. A user JWT can express restrictions on the messaging subjects to which it can publish or subscribe.

When a new user is added to an account, the account configuration need not change, as each user can and should have its own user JWT that can be verified by simply resolving its parent account.

### JWTs 和隐私

需要记住的一个关键细节是，虽然在其他系统中，JWTs被用作会话或身份验证的证据，但NATS JWTs仅被用作配置，描述如下:
- 实体的公共ID
- 发出证书的实体的公共ID
- 实体的能力


身份验证是一个公钥加密过程——客户端签署一个nonce证明身份，而信任链和配置提供授权。

服务器从不知道私钥，但可以验证签名者或颁发者确实匹配指定的或已知的公钥。

最后，所有NATS JWTs(运营商、账户、用户和其他人)都将使用[Ed25519](https://ed25519.cr.yp.to/)算法进行签名。如果不是，系统就会拒绝它们。

Authentication is a public key cryptographic process — a client signs a nonce proving identity while the trust chain and configuration provides the authorization.

The server is never aware of private keys but can verify that a signer or issuer indeed matches a specified or known public key.

Lastly, all NATS JWTs (Operators, Accounts, Users and others) are expected to be signed using
 the [Ed25519](https://ed25519.cr.yp.to/) algorithm. If they are not, they are rejected by the system.

### Sharing Between Accounts

While accounts provide isolation, there are many cases where you want to be able to consume messages produced by one account in another.
 There are two kinds of shares an account can _export_:
虽然帐户提供了隔离，但是在许多情况下，您希望能够在另一个帐户中使用由一个帐户生成的消息。一个账户可以输出两种类型:

- Streams
- Services

Streams are messages published by a foreign account; Subscribers in an _importing_ account can receive messages from a stream _exported_ by another.

Services are endpoints exported by a foreign account; Requesters _importing_ the service can publish requests to the _exported_ endpoint. 

Streams and Services can be public; Public exports can be imported by any account. Or they can be private. Private streams and services require an authorization token from the exporting account that authorizes the foreign account to import the stream or service.

An importing account can remap the subject where a stream subscriber will receive messages or where a service requestor can make requests. This enables the importing account to simplify their subject space.

Exports and imports from an account are explicit, and they are visible in the account's JWT. For private exports, the import will embed an authorization token or a URL storing the token. Imports and exports make it easy to audit where data is coming from or going to.
流是由外部帐户发布的消息;输入帐户中的订阅者可以从另一个导出的流接收消息。

服务是由外部帐户导出的端点;请求者_importing_服务可以将请求发布到_exports端点。

流和服务可以是公共的;公共出口可以通过任何方式进口。或者他们可以是私人的。私有流和服务需要导出帐户的授权令牌，该令牌授权外部帐户导入流或服务。

导入帐户可以重新映射主题，流订阅者可以在其中接收消息，服务请求者可以在其中发出请求。

### 配置

实体JWT配置是使用 [`nsc` tool](/nats_tools/nsc/README.md)。基本步骤包括:

- [Creation of an operator JWT](/nats_tools/nsc/nsc.md#creating-an-operator)
- [Configuring an Account Server](/nats_tools/nsc/nsc.md#account-server-configuration)
- [Setting up the NATS server to resolve Accounts](/nats_tools/nsc/nsc.md#nats-server-configuration)

之后, `nsc` 用于创建和编辑帐户和用户。
