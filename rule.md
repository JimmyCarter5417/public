```
struct net {
	struct netns_nf		nf;
	struct netns_xt		xt;
	struct netns_ct		ct;
}

struct netns_nf {
	struct nf_hook_entries __rcu *hooks_ipv4[NF_INET_NUMHOOKS];
}

struct netns_xt {
	struct list_head tables[NFPROTO_NUMPROTO];
}

struct netns_ct 
{
    struct hlist_nulls_head *hash;//存放已经经过确认的连接hash表
    struct hlist_head *expect_hash;//期望连接hash表
    struct hlist_nulls_head unconfirmed; //存放没经过确认的连接hash表
    struct hlist_nulls_head dying;
    struct ip_conntrack_stat *stat;
};
```

#### table
![](http://blog.chinaunix.net/attachment/201405/25/26517122_1401018420awZW.jpg)
```
struct xt_table
{
    struct list_head list;
    unsigned int valid_hooks; /*该表中有效的hook点位图*/
    struct xt_table_info *private; //指向真正存储rule的结构体
    const char name[XT_TABLE_MAXNAMELEN];//表名字
};

struct xt_table_info
{
    unsigned int size; //表的大小
    unsigned int number; //表中存的rule个数
    unsigned int initial_entries;//初始化表时创建的默认rule个数
    //各个hook(chain)在表中的偏移量
    unsigned int hook_entry[NF_INET_NUMHOOKS];
    //各个hook(chain)中默认规则在表中的偏移量
    unsigned int underflow[NF_INET_NUMHOOKS];
    //数组，存储各个cpu上自己rule拷贝的内存首地址
    void *entries[0];
};

struct xt_table *xt_register_table(struct net *net,
   const struct xt_table *input_table,
   struct xt_table_info *bootstrap,
   struct xt_table_info *newinfo)
```

#### rule
![](http://blog.chinaunix.net/attachment/201405/22/26517122_1400771884LU9s.jpg)
![](http://blog.chinaunix.net/attachment/201405/25/26517122_14010191826iw8.jpg)
* ipt_entry是rule，包含标准的匹配规则
* ipt_entry_match是扩展的匹配规则，是可选项。
* 每个rule最后带一个target。有系统自带的，也有扩展的。

#### match

* 通用的match数据结构
```
struct xt_match
{
    struct list_head list;
    //match名字，和iptables配置的-m参数相同
    const char name[XT_FUNCTION_MAXNAMELEN-1];
    //规则匹配函数
    bool (*match)();
    /* 匹配函数入参检查函数 */
    bool (*checkentry)(const struct xt_mtchk_param *);
    /* Called when entry of this type deleted. */
    void (*destroy)(const struct xt_mtdtor_param *);
    const char *table;
    //match的自定义数据长度
    unsigned int matchsize;
};
```

* 标准match

```
struct ipt_entry
{
    struct ipt_ip ip; //ipv4报文通用的匹配元素
    /*详见上图，rule首地址到target的偏移量*/
    /* Size of ipt_entry + matches */
    u_int16_t target_offset;
    /* Size of ipt_entry + matches + target */
    /*下一条rule的偏移量*/
    u_int16_t next_offset;
    /* Back pointer */
    unsigned int comefrom;
    /*命中报文的统计计数*/
    struct xt_counters counters;
    /* The matches (if any), then the target. */
    unsigned char elems[0];
};

//ipv4报文通用的匹配元素，源/目的ip地址/掩码，源/目的接口名/接口名掩码，传输层协议类型
struct ipt_ip 
{
    /* Source and destination IP addr */
    struct in_addr src, dst;
    /* Mask for src and dest IP addr */
    struct in_addr smsk, dmsk;
    char iniface[IFNAMSIZ], outiface[IFNAMSIZ];
    unsigned char iniface_mask[IFNAMSIZ], outiface_mask[IFNAMSIZ];
    /* Protocol, 0 = ANY */
    u_int16_t proto;
    /* Flags word */
    u_int8_t flags;
    /* Inverse flags */
    //位图，每一位代表以上匹配元素配置时是否是带！符号（取反）
    u_int8_t invflags;
};
```

* 扩展match
![](http://blog.chinaunix.net/attachment/201405/25/26517122_1401019764FExp.jpg)

* 全局match和target链表
`static struct xt_af *xt;`

#### target

* 标准target
```
struct xt_standard_target
{
    struct xt_entry_target target;
    //verdict 可以是指定返回值，也可以是goto到下一个rule的偏移量，
    //也可以是queue的队列号。
    int verdict;
};
```

* 扩展target
![](http://blog.chinaunix.net/attachment/201405/25/26517122_1401020102yWqy.jpg)
与扩展match类似
```
struct xt_target {
	struct list_head list;
	const char name[XT_EXTENSION_MAXNAMELEN];
	/* Returns verdict. Argument order changed since 2.6.9, as this
	   must now handle non-linear skbs, using skb_copy_bits and
	   skb_ip_make_writable. */
	unsigned int (*target)(struct sk_buff *skb,
			       const struct xt_action_param *);
}
```
