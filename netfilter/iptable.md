#### 内核中包括的防火墙子模块
* 链路层：ebtables
* ipv4：iptables
* ipv6：ip6tables
* ARP：arptables

#### IPv4防火墙四大功能
1. 对报文的过滤（对应filter表）
2. 对报文的修改（对应mangle表）
3. 对会话的连接跟踪（connection track)
4. 网络地址转换（NAT)

* Netfilter 中有包含一些表（table)，不同的表用来存储不同功能的配置信息。
* 每个table 里有多个chain，chain表示对报文的拦截处理点。
* 每个chain 包含一些用户配置的rule,一条rule包含了一个或多个匹配规则（match)和一个执行动作（target）。如果报文符合匹配规则后，需要根据该执行动作（target）来处理报文。

![](http://blog.chinaunix.net/attachment/201405/22/26517122_1400771884LU9s.jpg)
