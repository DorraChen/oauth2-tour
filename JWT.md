[TOC]

## 1.1.1 JWT

### 什么是JWT

根据 [官方文档](https://datatracker.ietf.org/doc/html/rfc7519) 的描述, JSON Web Token(JSON Web Token)是一个开放标准(RFC 7519),它定义了一种紧凑的、自包含的方式,用于作为 JSON 对象在各方之间安全地传输信息. 简单来说,JWT 就是用一种结构化封装的方式来生成 token 的技术.

"JWT令牌是一种结构化、信息化令牌."怎样理解这句话?
结构化可以组织用户的授权信息, 信息化是指令牌本身包含了授权信息.

因为JSON的通用性,所以JWT是可以进行跨语言支持的,JAVA,JavaScript,NodeJS,PHP等很多语言都可以使用.

JWT令牌需要在公网上做传输,所以在传输过程中,JWT令牌需要进行Base64编码以防止乱码,同时还需要进行签名和加密处理来防止数据信息泄露. JWT这种结构化体分为三部分:
- HEADER(头部)
    - 表示装载令牌类型和算法等信息,是 JWT 的头部. typ 表示第二部分 PAYLOAD 的类型,alg 表示使用的签名算法.
- PAYLOAD(数据体)
    - sub (subject)：令牌的主体, 一般设为资源拥有者的唯一标识
    - exp (expiration time)：令牌的过期时间
    - iat (Issued At)：令牌颁发的时间
    - iss (issuer)：签发者
    - aud (audience)：接收jwt的一方
    - nbf (Not Before)：生效时间(定义在什么时间之前,该jwt都是不可用的)
    - jti (JWT ID)：编号(jwt的唯一身份标识,主要用来作为一次性token,从而回避重放攻击)
    - 除了官方字段, 还可以定义私有字段(一个JWT内可以包含一切合法的JSON格式数据), JWT 默认是不加密的,任何人都可以读到,所以不要把秘密信息放在这个部分.
- SIGNATURE(签名)
    - 表示对JWT的签名, signature部分是对前两部分的加密签名, 防止数据篡改. 当受保护资源接收到第三方软件的签名,需要验证令牌的签名是否合法.
    - SIGNATURE的生成, 需要指定一个密钥secret, 存于服务端, 然后使用header中指定的签名算法, 比如HMAC SHA256, 按照公式`HMACSHA256(header (base64后的) + "." +payload (base64后的), secret)`生成签名.  注意: header和payload串型化的算法是 Base64URL, 与Base64有些区别,因为JWT作为令牌,有些场合会被放到URL,Base64 有三个字符`+`、`/`和`=`,故而使用Base64URL算法(`=`被省略、`+`替换成`-`，`/`替换成`_`).

经过签名后的JWT整体结构: header.payload.signature     (其中header和payload都是base64加密后的)

### JWT常用问题
Q1: 由于服务器不保存session状态,因此无法在使用过程中废止某个token,或者更改token的权限.也就是说,一旦JWT签发了,在到期之前就会始终有效. 假设用户修改了密码,或者取消了授权,要怎么办呢? 需要把JWT存储到分布式内存数据库中吗?

A1: 不能将JWT存储到数据库中,因为这样违背了JWT设计初衷-将信息通过结构化方式存入令牌本身.为了实现这种一般会有两种方式:
	(1)将每次生成的JWT令牌的秘钥粒度缩小到用户级别,一个用户一个秘钥.每次用户修改密码或者取消授权,就可以连同秘钥一起改了.这种方案需要配套一个秘钥管理服务.
	(2)如果不提供取消授权的功能,只考虑用户修改密码的情况,可以把用户密码作为JWT的秘钥,这样也是用户粒度级别的.但是我觉得这种方案不太安全,而且不太清楚这种场景是不是比较少出现.



### JWT常用框架

[JWT在线校验工具](https://jwt.io/)

目前Java开源的比较方便的JWT工具: [JJWT](https://github.com/jwtk/jjwt), 封装了 Base64URL 编码和对称 HMAC、非对称 RSA 的一系列签名算法,使用 JJWT,我们只关注上层的业务逻辑实现,而无需关注编解码和签名算法的具体实现,这类开源工具可以做到"开箱即用".

### JWT注意点总结

- 在不加密的情况下,payload部分是客户端可以解密的,所以这种情况下不要存放敏感信息.
- JWT的有效期应该设置得比较短;对于一些比较重要的权限,使用时应该再次对用户进行认证.
- 保护好secret私钥.
- 建议使用https协议.
- 对于JWT令牌生命周期的管理,除了令牌自然过期和刷新令牌的实现,还需要提供支持第三方软件主动发起令牌失效请求的api.(比如用户对第三方软件订购时长到期或者退订,授权token还没到期,这种情况下建议第三方软件遵守令牌撤回协议)
- 令牌在OAuth2.0系统中对于第三方软件都是不透明的,不论是结构化令牌还是非结构化令牌,对于第三方软件来说都不关心,需要关心令牌的,是授权服务和受保护资源服务.


## 参考文章
- https://datatracker.ietf.org/doc/html/rfc7519
- https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html
- https://www.jianshu.com/p/576dbf44b2ae