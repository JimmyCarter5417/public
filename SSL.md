* 密码学目标：
    * 私密性
* 完整性：
    * MAC消息验证码
* 身份验证：
    * 数字签名
* 不可抵赖：
    * 数字签名
  
[HTTPS协议详解(一)：HTTPS基础知识](https://blog.csdn.net/hherima/article/details/52469267)  
[HTTPS协议详解(二)：TLS/SSL工作原理](https://blog.csdn.net/hherima/article/details/52469360)  
[HTTPS协议详解(三)：PKI 体系](https://blog.csdn.net/hherima/article/details/52469488)  
[HTTPS协议详解(四)：TLS/SSL握手过程](https://blog.csdn.net/hherima/article/details/52469674)  
[HTTPS协议详解(五)：HTTPS性能与优化](https://blog.csdn.net/hherima/article/details/52469787)  
[TLS1_2.pdf](https://tools.ietf.org/pdf/rfc5246.pdf)  


#### 基础
* X.509是一种数字证书的格式标准
* 证书链： 根证书A - 中介证书B - 终端证书C
* A/B为CA
* key私钥
* csr证书签名请求，这不是证书，可以简单理解成公钥，提交给CA生成证书
* crt/cer证书
	* PEM可读/DER不可读
	* 签名/公钥/颁发者/。。。

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

https://blog.csdn.net/zhanyiwp/article/details/105665374?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-6&spm=1001.2101.3001.4242

https://blog.csdn.net/u013066244/article/details/79364011

https://www.cnblogs.com/xiaolincoding/p/14318338.html

![](https://img-blog.csdnimg.cn/2020042119095762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW55aXdw,size_16,color_FFFFFF,t_70)

* ClientHello
	* Version
		* 客户端支持的最高协议版本
	* Random
		* 客户端随机数记作 random_C，可以防止重放攻击
	* Session ID
		* 在第一次连接时，会话 ID是空的，表示客户端并不希望恢复某个已存在的会话。由服务端生成通过 ServerHello 返回给客户端
	* Cipher Suites
		* 客户端支持的密码套件列表，按优先级顺序排列
		* TLS_DHE_RSA_WITH_AES_256_CBC_SHA
		* 格式
			* 密钥交换算法：RSA, DH, ECDH, DHE, ECDHE
			* 认证算法：RSA, DSA, ECDSA
			* 对称加密算法：AES、不推荐：DES/3DES/RC4
			* 消息验证算法：HMAC（SHA1/SHA256/SHA384）
	* Compression
		* 客户端支持的压缩方法，默认的压缩方法是 null，代表没有压缩
	* Extensions
		* 一系列扩展，如sni
		
* ServerHello
	* 消息的结构与 ClientHello 类似，只是每个字段只包含一个选项，其中包含服务端的 random_S 参数
	
* Certificate
	* 
