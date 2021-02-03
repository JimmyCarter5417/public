http://blog.chinaunix.net/uid-26517122-id-4281305.html

* 状态防火墙
* 控制连接：master连接
* 数据连接：期望连接


netfilter内核参数proc位置
/proc/sys/net/netfilter/

conntrack工具

引用计数
nf_conntrack_get
nf_conntrack_put

初始化conntrack
init_conntrack

设置event_cache标志位，在ipv4_confirm最后实现事件通告机制
nf_conntrack_event_cache

struct nf_conntrack_tuple

每个网络命名空间有如下一个数据结构的实例，来管理和存放生成的连接的一些信息。
```
struct netns_ct 
{
    struct hlist_nulls_head *hash;//存放已经经过确认的连接hash表
    struct hlist_head *expect_hash;//期望连接hash表
    struct hlist_nulls_head unconfirmed; //存放没经过确认的连接hash表
    struct hlist_nulls_head dying;
    struct ip_conntrack_stat *stat;
};
```

一个连接包含正反两个方向的两条报文流。
```
struct nf_conn {
    //对连接的引用计数
    struct nf_conntrack ct_general;
    spinlock_t lock;//正向和反向的连接元组信息。
    struct nf_conntrack_tuple_hash tuplehash[IP_CT_DIR_MAX];
    //该连接的连接状态
    unsigned long status;
    //如果该连接是期望连接，指向跟其关联的主连接
    struct nf_conn *master;
    //连接垃圾回收定时器
    struct timer_list timeout;
    /*存储特定协议的连接跟踪信息*/
    union nf_conntrack_proto proto;
    /*指向扩展结构，该结构中包含一些基于连接的功能扩展处理函数 */
    struct nf_ct_ext *ext;
   //网络命名空间
    struct net *ct_net;
};
```

`struct hlist_head *nf_ct_helper_hash; // 4层协议helper`

#### 普通ct创建过程
* original首包：
	* 入口：
		* 查找ct失败，建立一个新的连接，创建正反向元组信息，并把该连接的正向连接挂到unconfirmed链表上
		* 把该连接和报文进行关联，连接状态是NEW。
	* 出口：
		* 把该连接的正向连接从unconfirmed链表上摘下来，把该连接的***正反向连接***加入到连接hash表中。并把该连接确认状态置为confirmed状态，即置位status的IPS_CONFIRMED_BIT位。

* reply首包
	* 入口
		* 根据tuple找到ct，发现方向是reply，给连接中的status置位IPS_SEEN_REPLY_BIT，表明该连接已经收到了回应报文。这时把报文的连接状态变为ESTABLISHED
	* 出口
		* 可以正常根据tuple找到ct

#### 带期望的ct创建过程
* original首包：
	* 入口：
		* 创建ct时，通过`dport / L3 / L4`查找ct_helper_hash，找到四层协议注册的helper
		* 为ct创建help类型的扩展，保存helper
	* 出口：
		* 执行四层协议helper函数
			* 创建期望连接`ip/ dport / L3 / L4 ...`
			* 将期望连接加到全局的期望连接hash
* reply首包
	* 入口
		* 根据tuple无法找到ct，新建ct
		* 通过`dip / dport / L3 / L4`，从期望连接hash中找到对应的期望连接
		* 将ct->status置位IPS_EXPECTED_BIT（连接状态即为RELATED）
		* 设置ct的master和helper
	* 出口：
		* 可以正常根据tuple找到ct

每个struct nf_conn实例代表一个连接。每个skb都有一个指针，指向和它相关联的连接。
```
struct sk_buff 
{
  struct nf_conntrack *nfct;//指向struct nf_conn实例
  int nfctinfo:3; //记录报文的连接状态
};
```
conntrack扩展
```
	struct nf_ct_ext 
	{
		struct rcu_head rcu;
		u16 offset[NF_CT_EXT_NUM];
		u16 len;
		char data[0];
	};

	enum nf_ct_ext_id
	{
		NF_CT_EXT_HELPER, //helper机制
		NF_CT_EXT_NAT,//网络地址转换
		NF_CT_EXT_ACCT,//连接统计计数
		NF_CT_EXT_ECACHE,//Netfilter事件通知机制
		NF_CT_EXT_NUM,
	};

	struct nf_ct_ext_type
	{
		void (*destroy)(struct nf_conn *ct);
		void (*move)(void *new, void *old);
		enum nf_ct_ext_id id;  //标识扩展功能的id
		unsigned int flags;
		u8 len;  //扩展功能私有数据的长度
		u8 align;  //扩展功能私有数据的对齐字节
		u8 alloc_size;
	};
```

给ct添加扩展功能
`nf_ct_ext_add(ct, id, gfp)`

注册扩展功能
`int nf_ct_extend_register(struct nf_ct_ext_type *type)`

```
struct nf_conn_help {
	struct nf_conntrack_helper __rcu *helper;
};

struct nf_conntrack_helper {
	const struct nf_conntrack_expect_policy *expect_policy;
	struct nf_conntrack_tuple tuple;
	int (*help)(struct sk_buff *skb,
		    unsigned int protoff,
		    struct nf_conn *ct,
		    enum ip_conntrack_info conntrackinfo);
};
```

![](http://blog.chinaunix.net/attachment/201406/7/26517122_1402116198JgpH.jpg)

#### L3对conntrack的处理函数
```
struct nf_conntrack_l3proto __rcu *nf_ct_l3protos[AF_MAX]
int nf_ct_l3proto_register(struct nf_conntrack_l3proto *proto)

struct nf_conntrack_l3proto {
    u_int16_t l3proto;
    const char *name;
    /*从报文的L3头部提取出conntrack 的元组信息*/
    bool (*pkt_to_tuple)();
    /*conntrack 正反向连接的元组中L3信息的转换函数*/
    bool (*invert_tuple)();
    /*获取L4协议类型的函数*/
    int (*get_l4proto)();
};
```

#### L4对conntrack的处理函数
```
struct nf_conntrack_l4proto __rcu **nf_ct_protos[PF_MAX]
int nf_ct_l4proto_register(struct nf_conntrack_l4proto *l4proto)

struct  nf_conntrack_l4proto {
    /*从报文的L4头部提取出conntrack 的元组信息*/
    bool (*pkt_to_tuple)();
    /*conntrack 正反向连接的元组中L4信息的转换函数*/
    bool (*invert_tuple)();
    /*各个协议对报文的额外处理函数*/
    int (*packet)();
    /*Called when a new connection for this protocol found*/
    bool (*new)();
    /*检查报文数据的正确性*/
    int (*error)();
};
```

#### ct hook
![](http://blog.chinaunix.net/attachment/201406/7/26517122_1402124267lXx5.jpg)

* conntrack要分析报文的L4层信息，因此如果打开了ip conntrack功能，conntrack就会在netfilter入口对ip分片报文进行重组，重组后再进行处理。conntrack以最高优先级在Netfilter的入口处注册了hook函数，来进行IP分片报文的重组。
* 在Netfilter的入口点，conntrack会拦截报文，查找已建立的conntrack和报文进行关联，如果conntrack没有建立，就新建conntrack。
* 在Netfilter的出口点，conntrack拦截报文，对已经建立的conntrack的状态进行确认和更新。

[核心函数](http://blog.chinaunix.net/uid-26517122-id-4293135.html)
