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
  * 给定两个大素数p、q 容易相乘得到n，而对n进行因式分解却很困难
* DH/ECDH
  * （椭圆曲线）离散对数问题
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

***
#### tls扩展

* sni
* session ticket
* ocsp