---
title: 浅析JWT
date: 2022-01-13 22:27:22
excerpt: 浅析JWT
categories: 学习
---



# 浅析JWT

### JWT简介

> Json web token (JWT)，是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519)。该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

我们在学习jwt的同时，也需要了解传统session认证

### session认证

> 我们知道，http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了，这就是传统的基于session认证。



这种传统的**session**我们很难得到拓展，并且session相关的数据是保存在服务器中，随着用户数量的增加，服务器的载荷也就越大，这时候许多问题就暴露了出来

- **Session**：每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而**随着认证用户的增多，服务端的开销会明显增大。**
- **扩展性**：用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上，这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力。
- **CSRF**: 因为是 基于cookie 来进行用户识别的，**cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击。**



### JWT与Session的差异

**相同点**：**它们都是存储用户信息；**

**不同点**

- **Session是在服务器端的，而JWT是在客户端的。**

- **Session方式存储用户信息的最大问题在于要占用大量服务器内存，增加服务器的开销。而JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。**
- **Session的状态是存储在服务器端，客户端只有session id；而Token的状态是存储在客户端**

![图片-1684742759615](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151430767.png)



### 基于Token的身份认证 与 基于服务器的身份认证

#### 基于服务器的身份认证

在讨论基于Token的身份认证是如何工作的以及它的好处之前，我们先来看一下以前我们是怎么做的：

> HTTP协议是无状态的，也就是说，如果我们已经认证了一个用户，那么他下一次请求的时候，服务器不知道我是谁，我们必须再次认证

传统的做法是将已经认证过的用户信息存储在服务器上，比如Session。用户下次请求的时候带着Session ID，然后服务器以此检查用户是否认证过。

这种基于服务器的身份认证方式存在一些问题：

- **Sessions : 每次用户认证通过以后，服务器需要创建一条记录保存用户信息，通常是在内存中，随着认证通过的用户越来越多，服务器的在这里的开销就会越来越大。**
- **Scalability : 由于Session是在内存中的，这就带来一些扩展性的问题。**
- **CORS : 当我们想要扩展我们的应用，让我们的数据被多个移动设备使用时，我们必须考虑跨资源共享问题。当使用AJAX调用从另一个域名下获取资源时，我们可能会遇到禁止请求的问题。**
- **CSRF : 用户很容易受到CSRF攻击。**

#### 用Token的好处

- **无状态和可扩展性：Tokens存储在客户端。完全无状态，可扩展。我们的负载均衡器可以将用户传递到任意服务器，因为在任何地方都没有状态或会话信息。**
- **安全：Token不是Cookie。（The token, not a  cookie.）每次请求的时候Token都会被发送。而且，由于没有Cookie被发送，还有助于防止CSRF攻击。即使在你的实现中将token存储到客户端的Cookie中，这个Cookie也只是一种存储机制，而非身份认证机制。没有基于会话的信息可以操作，因为我们没有会话!**
- **token在一段时间以后会过期，这个时候用户需要重新登录。这有助于我们保持安全。还有一个概念叫token撤销，它允许我们根据相同的授权许可使特定的token甚至一组token无效。**

### 什么时候用JWT

下列场景中使用JSON Web Token是很有用的：

- **Authorization** (授权) : 这是使用JWT的最常见场景。一旦用户登录，后续每个请求都将包含JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录是现在广泛使用的JWT的一个特性，因为它的开销很小，并且可以轻松地跨域使用。
- **Information Exchange** (信息交换) : 对于安全的在各方之间传输信息而言，JSON Web  Tokens无疑是一种很好的方式。因为JWTs可以被签名，例如，用公钥/私钥对，你可以确定发送人就是它们所说的那个人。另外，由于签名是使用头和有效负载计算的，您还可以验证内容没有被篡改。



### JWT结构

JSON Web Token由三部分组成，它们之间用圆点(.)连接。这三部分分别是：

- **Header**
- **Payload**
- **Signature**

下面具体来看看每个部分

#### Header（头部）

jwt头部承载两部分信息

- **声明类型**：类似于jwt
- **声明加密的算法**：通常直接使用 HMAC SHA256。这的加密算法也就是签名算法。

```
{
  'typ': 'JWT',
  'alg': 'HS256'
}
```

然后将头部进行**base64加密**得到第一部分

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```



> **可以将JWT中的alg算法修改为none：**
>
> **JWT支持将算法设定为“None”。如果“alg”字段设为“ None”，那么JWT的第三部分会被置空，这样任何token都是有效的。这样就可以伪造token进行随意访问。**



#### Payload（载荷）

payload**就是存放有效信息的地方**，这些有效信息包含三个部分

- **registered** 标准中注册的声明
- **public** 公共的声明
- **private** 私有的声明



**标签中注册的声明**

- **iss: jwt签发者**
- **sub: jwt所面向的用户**
- **aud: 接收jwt的一方**
- **exp: jwt的过期时间，这个过期时间必须要大于签发时间**
- **nbf: 定义在什么时间之前，该jwt都是不可用的.**
- **iat: jwt的签发时间**
- **jti: jwt的唯一身份标识，主要用来作为一次性token，从而回避重放攻击。**



**公共的声明**

公共的声明可以添加任何的信息，一般添加**用户的相关信息**或**其他业务需要的必要信息**。但不建议添加敏感信息，因为该部分在客户端可解密。



**私有的声明**

私有声明**是提供者和消费者所共同定义的声明**，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。



下面是一个自己定义的**payload**

```
{
  "sub": "1234567890",
  "iss": "http://localhost:8000/auth/login",
  "iat": 1451888119,
  "exp": 1454516119,
  "nbf": 1451888119,
  "name": "John Doe",
  "admin": true
}
```

将其base64加密得到jwt第二部分

```
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9
```



> 注意，不要在JWT的payload或header中放置敏感信息，除非它们是加密的。



#### Signature（签证）

签证信息由**3个部分组成**

- **header（base64加密后的）**
- **payload（base64加密后的）**
- **secret**

这个部分需要base64加密后的header和base64加密后的payload使用`.`连接组成的字符串，然后通过header中声明的加密方式进行加盐`secret`组合加密，然后就构成了jwt的第三部分。

```
// javascript
var encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);

var signature = HMACSHA256(encodedString, 'secret'); // TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```

```
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```



> **注意：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret，那就意味着客户端是可以自我签发jwt了。**



### JWT认证过程

客户端接受服务器的**JWT**将其存储在**Cookie**或loaclSrorage中。此后，客户端将在与服务器进行交互中都会带JWT。如果它存储在Cookie中，就可以自动发送，但不会跨域，因此一般是将它放在HTTP请求的**<u>Header Authorization</u>**字段中当跨域时，也可以将JWT被放置于**POST**请求的数据主体中。

服务器每次收到信息都会对它的前两部分进行加密，然后比对加密后的结果是否跟客户端传送过来的第三部分相同，如果相同则验证通过，否则失败。

一般是在请求头里加入`Authorization`，并加上`Bearer`标注：

```
fetch('api/user/1', {
  headers: {
    'Authorization': 'Bearer ' + token
  }
})
```



![图片-1684742819251](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151431006.png)



JWT本身包含认证信息，因此一旦信息泄露，任何人都可以获得令牌的所有权限。为了减少盗用，JWT的有效期不宜设置太长。对于某些重要操作，用户在使用时应该每次都进行进行身份验证。为了减少盗用和窃取，JWT不建议使用HTTP协议来传输代码，而是使用加密的HTTPS协议进行传输。



### JWT安全隐患

- 修改算法为none
- 修改算法从RS256到HS256
- 信息泄露 **密钥泄露**
- 爆破密钥

这里我们着重说一下**jwt伪造**，这个考点在ctf中经常出现



#### JWT伪造

当我面拿到一串**JWT**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6ImZhbHNlIn0.oe4qhTxvJB8nNAsFWJc7_m3UylVZzO3FwhkYuESAyUM
```

我们去在线解jwt网站encode https://jwt.io/

![图片-1684742810710](https://cdn.jsdelivr.net/gh/R3damancy1/blog-pic/202404151432089.png)

**所以，我们的目的就是把false改成true，而且要通过服务器的验证，这点很重要，并不是直接把false改成true就万事大吉了。因为服务器收到token后会对token的有效性进行验证。**

**验证方法：首先服务端会产生一个key，然后以这个key作为密钥，使用第一部分选择的加密方式（这里就是HS256），对第一部分和第二部分拼接的结果进行加密，然后把加密结果放到第三部分。**

**服务器每次收到信息都会对它的前两部分进行加密，然后比对加密后的结果是否跟客户端传送过来的第三部分相同，如果相同则验证通过，否则失败。**

**因为加密算法我们已经知道了，我们只要再得到加密的key，我们就能伪造数据，并且通过服务器的检查**



我们需要对**secret**进行爆破

推荐工具  https://github.com/brendan-rius/c-jwt-cracker



**破解出来的加密密钥key就是：54l7y**，

**现在我们可以任意构造我们想要的jwt**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhZG1pbiI6InRydWUifQ.hv55uOLPmX_C0Nx8qz-O2psgUq6V3EA0fgRqGFm5W6Q
```

构造成功