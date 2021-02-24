[HTTPS协议详解(一)：HTTPS基础知识](https://blog.csdn.net/hherima/article/details/52469267)  
[HTTPS协议详解(二)：TLS/SSL工作原理](https://blog.csdn.net/hherima/article/details/52469360)  
[HTTPS协议详解(三)：PKI 体系](https://blog.csdn.net/hherima/article/details/52469488)  
[HTTPS协议详解(四)：TLS/SSL握手过程](https://blog.csdn.net/hherima/article/details/52469674)  
[HTTPS协议详解(五)：HTTPS性能与优化](https://blog.csdn.net/hherima/article/details/52469787)  
[TLS1_2.pdf](https://tools.ietf.org/pdf/rfc5246.pdf)  
[图解 ECDHE 密钥交换算法](https://www.cnblogs.com/xiaolincoding/p/14318338.html)  
[HTTPS性能优化](https://toutiao.io/posts/d6nu08/preview)  

SACK  
TFO  

完美前向保密PFS，指长期使用的主密钥泄漏不会导致过去的会话密钥泄漏。

#### session复用

* Session Cache
  * 使用Client Hello中的Session ID查询服务端的Session Cache,如果服务端有对应的缓存，则直接使用已有的Session信息提前完成握手，称为简化握手。
  * 优点：
    * TLS协议的标准字段
  * 缺点：
    * 需要消耗服务端内存来存储Session内容
    * 只支持单机多进程间共享缓存，不支持多机间分布式缓存
* Session Ticket
  * Server将Session信息加密成Ticket发送给浏览器，浏览器后续握手请求时会发送Ticket，Server端如果能成功解密和处理Ticket，就能完成简化握手
  * 优点：
    * 不需要服务端消耗大量资源来存储Session内容
  * 缺点：
    * TLS协议的一个扩展特性，目前的支持率不是很广泛
    * 需要维护一个全局的key来加解密，需要考虑key的安全性和部署

* session ID 和session ticket都在client hello中，则使用session ticket

#### 证书验证

* CRL
  * 由证书颁发机构定期更新的一个列表，包含了所有已被作废的证书，浏览器可以定期下载这个列表用于验证证书合法性
* OCSP
  * 在线查询接口，浏览器可以实时查询单个证书的合法性
  * OCSP Stapling（OCSP 封套），是指服务端在证书链中包含颁发机构对证书的 OCSP 查询结果，从而让浏览器跳过自己去验证的过程
  * 服务端在发送完证书后，紧接着又发来了它的 OCSP 响应，从而避免了浏览器自己去验证证书造成阻塞
![](https://st.imququ.com/i/webp/static/uploads/2015/11/tls-ocsp-response.png.webp)


#### RSA与DH区别

* RSA 密钥协商算法「不支持」前向保密，ECDHE 密钥协商算法「支持」前向保密
* 使用了 RSA 密钥协商算法，TLS 完成四次握手后，才能进行应用数据传输，而对于 ECDHE 算法，客户端可以不用等服务端的最后一次 TLS 握手，就可以提前发出加密的 HTTP 数据，节省了一个消息的往返时间


#### 基础

* X.509是一种数字证书的格式标准
* 证书链： 根证书A - 中间证书B - 终端证书C

#### 握手

* 3种握手
  * 完整的握手， 对服务器进行身份验证
  * 恢复之前的会话采用的简短握手
  * 对客户端和服务器都进行身份验证的握手
 
* 完整握手流程
  * 交换各自支持的功能，对需要的连接参数达成一致
  * 验证出示的证书，或使用其他方式进行身份验证
  * 对将用于保护会话的共享主密钥达成一致
  * 验证握手消息并未被第三方团体修改

<https://blog.csdn.net/zhanyiwp/article/details/105665374?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-6&spm=1001.2101.3001.4242>

<https://blog.csdn.net/u013066244/article/details/79364011>

<https://www.cnblogs.com/xiaolincoding/p/14318338.html>


* ClientHello
  * Version
    * 客户端支持的最高协议版本
  * Random
    * 客户端随机数记作 random_C，可以防止重放攻击
  * Session ID
    * 在第一次连接时，会话 ID是空的，表示客户端并不希望恢复某个已存在的会话。由服务端生成通过 ServerHello 返回给客户端
    * 会话复用：
      *
  * Cipher Suites
    * 客户端支持的密码套件列表，按优先级顺序排列
    * TLS_DHE_RSA_WITH_AES_256_CBC_SHA
    * 格式
      * 密钥交换算法：RSA, DH, ECDH, DHE, ECDHE
      * 认证算法：RSA, DSA, ECDSA
      * 对称加密算法：AES、不推荐：DES/3DES/RC4
      * 摘要算法：HMAC（SHA1/SHA256/SHA384）
  * Compression
    * 客户端支持的压缩方法，默认的压缩方法是 null，代表没有压缩
  * Extensions
    * 一系列扩展，如SNI/SessionTicket
  
* ServerHello
  * 消息的结构与 ClientHello 类似，只是每个字段只包含一个选项，其中包含服务端的 random_S 参数

* Certificate
  * 配置证书链的最佳实践是只包含站点证书和中间证书，不要包含根证书，也不要漏掉中间证书
  * 大部分证书链都是「站点证书 - 中间证书 - 根证书」三级，这时服务端只需要发送前两个证书即可。但也有的证书链有四级，那就需要发送站点证书外加两个中间证书了。
  * 如果需要进一步减小证书大小，可以选择 ECC 证书
  * 校验有效性：
    * 证书链的可信性
    * 证书是否吊销：离线 CRL 与 在线 OCSP
    * 证书是在有效期
    * 证书域名是否与当前的访问域名匹配

* ServerKeyExchange
  * 部分算法需要，如DH

* ClientKeyExchange
  * 对于ECDHE 算法，ClientKeyExchange 传递的是 DH 算法的客户端参数，如果使用的是 RSA 算法则此处应该传递加密的预主密钥

* ChangeCipherSpec
  * 通知服务器后续的通信都采用协商的通信密钥和加密算法进行加密通信

* Finished (Encrypted Handshake Message)
  * 包含 verify_data 字段，它的值是握手过程中所有消息的散列值
