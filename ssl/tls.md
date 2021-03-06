https://www.cnblogs.com/thammer/p/7654925.html

#### 密码学分层体系

1. 基础算法原语
   * aes , rsa， md5, sha256，ecdsa, dh，ecdh
2. 选定参数后的标准算法
   * 块加密算法 block cipher: AES，
   * 流加密算法 stream cipher: RC4，ChaCha20
   * Hash函数: MD5，sha1，sha256，sha512
   * 消息认证算法: HMAC-sha256，AEAD
   * 密钥交换: DH，ECDH，RSA，PFS方式的（DHE，ECDHE）
   * 公钥加密: RSA，rabin-williams
   * 数字签名算法: RSA，DSA，ECDSA
   * 密码衍生函数KDF（从一些协商好的参数中衍生对称密钥）: TLS-12-PRF(SHA-256), bcrypto，scrypto，pbkdf2
   * 随机数生成器: /dev/urandom 等
3. 多种标准算法组合而成的半成品组件
   * 对称传输组件： aes-128-cbc + hmac-sha256，aes-128-gcm
   * 认证密钥协商算法: rsassa-OAEP + ecdh-secp256r1
   * 。。。
4. 各种组件拼装而成的成品密码学协议
   * tls协议，ssh协议 

***
#### 完美前向保密PFS
* 指长期使用的主密钥泄漏不会导致过去的会话密钥泄漏。

#### 算法原理
* RSA
  * 大整数因子分解问题
* DH/ECDH
  * （椭圆曲线）离散对数问题
* ECC
  * 椭圆曲线问题
***
#### CipherSuite加密套件

`TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
* 密钥交换算法
* 数字签名算法
* 对称加密算法
* 消息认证码算法MAC

***
#### TLS协议

* 主体协议
  * record协议：对称加密传输
    * 分片
    * 生成序列号
    * 压缩（不安全）
    * 计算MAC
    * 发给tcp
  * handshake协议：密钥协商
* 辅助协议
  * ChangeCipherSpec协议
  * Alert协议
  * ApplicationData协议
***

#### 对称加密算法

* stream cipher
  * 流加密，每次加密一堆bit
  * RC4，不安全！
* block cipher
  * 块加密，每次加密一个bit
  * （des，3des）不安全，Camellia，aes
* AEAD cipher
  * 块加密的GCM/CCM模式
  * 在内部同时实现加密和认证
  * AES-128-GCM，AES-192-GCM，AES-256-GCM，ChaCha20-IETF-Poly1305

***
#### 非对称算法

* 加密
  * RSAES-PKCS1-v1_5，RSAES-OAEP
* 密钥协商
  * DH，DHE，ECDH，ECDHE
* 数字签名
  * RSASSA-PKCS1-v1_5，RSASSA-PSS，ECDSA，DSA

> **非对称加密算法**RSA，也可以当作**密钥协商算法**来用  
> RSA有两种模式，既可以当**非对称加密算法**使用，又可以当**非对称数字签名**使用

> RSA 密钥交换、RSA 签名
> ECDHE 密钥交换、RSA 签名
> ECDHE 密钥交换、ECDSA 签名

***
* 完整握手
```
      Client                                               Server

      ClientHello                  -------->
                                                      ServerHello
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data
```
* ServerKeyExchange：服务器的证书只用做签名，不用做密钥交换；Certificate消息没有足够的信息，不能让客户端完成premaster的密钥交换
* handshake 协议的消息必须按照规定的顺序发，收到不按顺序来的消息，当成fatal error处理
*  ClientHello，ServerHello，HelloRequest
*  CertificateVerify：从ClientHello开始，一直到CertificateVerify之前的所有消息，做个签名，证明自己确实拥有客户端证书的私钥
*  在TLS实际部署中，我们一般只使用这4种：ECDHE_RSA, DHE_RSA, ECDHE_ECDSA，RSA
***
![数字签名](https://bbsmax.ikafan.com/static/L3Byb3h5L2h0dHBzL2ltYWdlczIwMTUuY25ibG9ncy5jb20vYmxvZy83MzMwMTMvMjAxNjExLzczMzAxMy0yMDE2MTEyMjE1MjYwOTAwMy0xNzEwNTg3NzMxLnBuZw==.jpg)
* session id
```
  Client                                                Server

  ClientHello                   -------->
                                                   ServerHello
                                            [ChangeCipherSpec]
                                <--------             Finished
  [ChangeCipherSpec]
  Finished                      -------->
  Application Data              <------->     Application Data
```

* session ticket
```
      Client                                               Server
      ClientHello
      (empty SessionTicket extension)------->
                                                      ServerHello
                                  (empty SessionTicket extension)
                                                     Certificate*
                                               ServerKeyExchange*
                                              CertificateRequest*
                                   <--------      ServerHelloDone
      Certificate*
      ClientKeyExchange
      CertificateVerify*
      [ChangeCipherSpec]
      Finished                     -------->
                                                 NewSessionTicket
                                               [ChangeCipherSpec]
                                   <--------             Finished
      Application Data             <------->     Application Data
   Figure 1: Message flow for full handshake issuing new session ticket
```

```
      Client                                                Server
      ClientHello
      (SessionTicket extension)      -------->
                                                       ServerHello
                                   (empty SessionTicket extension)
                                                  NewSessionTicket
                                                [ChangeCipherSpec]
                                    <--------             Finished
      [ChangeCipherSpec]
      Finished                      -------->
      Application Data              <------->     Application Data
        Figure 2: Message flow for abbreviated handshake using new
                              session ticket
```

* [重协商](https://blog.csdn.net/O4dC8OjO7ZL6/article/details/78537550)
![](https://ss.csdn.net/p?http://mmbiz.qpic.cn/mmbiz_png/8rlpLIcBeicGPnyyLLuyZWLOJqYw1fappyrEMicaxNU7qRdGEic5cjAicrozEYe1mCRpHo4vWu52LyUmhn8ySNXbwg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)
  * 如果接收方不同意的话，都会通过SSL Alert响应以拒绝重协商。
  * 安全重协商
![](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/8rlpLIcBeicGPnyyLLuyZWLOJqYw1fappsoXoyZb2H1sK7Nic224iaQYw2zMs6DG11ImLkJD8JaTelKictk3icSaWew/0?wx_fmt=png)

***
#### tls扩展

* sni
* session ticket
* status_request: ocsp
* renegotiation_info


***
* HPKP
  * HTTP Public-Key-Pins响应头
  * 将特定的加密公钥与特定的服务器相关联(定义一组 Base64 编码的 SPKI 指纹)，以降低伪造证书对 MITM 攻击的风险。

* HSTS
  *  HTTP Strict-Transport-Security响应头
  *  让浏览器得知，在接下来的一段时间内，当前域名只能通过 HTTPS 进行访问，并且在浏览器发现当前连接不安全的情况下，强制拒绝用户的后续访问要求。
  *  HSTS Preload List：只要是在这个列表里的域名，无论何时、何种情况，浏览器都只使用 HTTPS 发起连接

* Expect-CT(Certificate Transparency证书透明度) 
  * 响应头
  * 浏览器：验证该证书是否在CT的Log服务器中有记录，如果没有记录则给出安全警告。这就导致恶意网站必须将证书提交到Log Server上来规避浏览器的安全检查。
  * 域名所有者：通过监控自己域名及时获得警报。如果发现Log Server上有另一个证书与自己的域名相同，能够快速做出反应。
  * CA：由于签发的证书会有记录，因此给CA的自身监管带来了压力。

* Alternate-Protocol
  * 响应头，用于协商在未来的HTTP请求中使用QUIC/SPDY

* Upgrade
  * 请求头，浏览器想要升级到WebSocket/HTTP2

***
访问http://www.baidu.com
https://www.cnblogs.com/mylanguage/p/5635524.html

![](http://ww2.sinaimg.cn/mw690/6941baebjw1ertfxnpeeej20hn062glz.jpg)
![](http://ww4.sinaimg.cn/mw690/6941baebjw1ertfxndbb7j20gw0axwgl.jpg)

***
#### 证书

内置 ECDSA 公钥的证书是 ECC 证书
内置 RSA 公钥的证书是 RSA 证书

* CN
  * 在这里放置主要的域名（兼容性）
  * *.google.com
* SAN
  * 扩展此证书支持的域名，使得一个证书可以支持多个不同域名

***
* SSL加速卡

* SSL卸载
  * 服务器负载均衡的一个增强功能

***
# todo

* TLS 1.3