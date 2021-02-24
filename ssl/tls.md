https://www.cnblogs.com/thammer/p/7654925.html

#### TLS协议

* 主体协议
  * record协议：对称加密传输
  * handshake协议：认证密钥协商
* 辅助协议
  * ChangeCipherSpec协议
  * Alert协议
  * ApplicationData协议

#### 加密和认证的组合方式

* MAC-then-Encrypt
  * 有漏洞
* Encrypt-then-MAC
  * AEAD加密，GCM模式，不需要MAC算法

#### 对称加密算法

* stream cipher
  * 流加密，RC4，不安全！
* block cipher
  * 块加密，AES-CBC
* AEAD cipher
  * aes-gcm-128/aes-gcm-256

#### 非对称的算法

* 非对称加密
  * RSAES-PKCS1-v1_5，RSAES-OAEP
* 非对称密钥协商
  * DH，DHE，ECDH，ECDHE
* 非对称数字签名
  * RSASSA-PKCS1-v1_5，RSASSA-PSS，ECDSA，DSA