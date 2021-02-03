http://blog.chinaunix.net/uid-26517122-id-4281305.html

状态防火墙
控制连接：master连接
数据连接：期望连接


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
* 入口：
    * 第一次建立一个新的连接，创建正反向元组信息，并把该连接的正向连接挂到unconfirmed链表上
    * 把该连接和报文进行关联，连接状态是NEW。
* 出口：
    * 把该连接的正向连接从unconfirmed链表上摘下来，把该连接的正反向连接加入到连接hash表中。并把该连接确认状态置为confirmed状态，即置位status的IPS_CONFIRMED_BIT位。

每个struct nf_conn实例代表一个连接。每个skb都有一个指针，指向和它相关联的连接。
```
struct sk_buff 
{
  struct nf_conntrack *nfct;//指向struct nf_conn实例
  nfctinfo:3; //记录报文的连接状态
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

在conntrack的建立和处理过程中,给连接添加扩展功能
nf_ct_ext_add(ct, id, gfp)
