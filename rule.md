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
    struct hlist_nulls_head *hash;//����Ѿ�����ȷ�ϵ�����hash��
    struct hlist_head *expect_hash;//��������hash��
    struct hlist_nulls_head unconfirmed; //���û����ȷ�ϵ�����hash��
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
    unsigned int valid_hooks; /*�ñ�����Ч��hook��λͼ*/
    struct xt_table_info *private; //ָ�������洢rule�Ľṹ��
    const char name[XT_TABLE_MAXNAMELEN];//������
};

struct xt_table_info
{
    unsigned int size; //��Ĵ�С
    unsigned int number; //���д��rule����
    unsigned int initial_entries;//��ʼ����ʱ������Ĭ��rule����
    //����hook(chain)�ڱ��е�ƫ����
    unsigned int hook_entry[NF_INET_NUMHOOKS];
    //����hook(chain)��Ĭ�Ϲ����ڱ��е�ƫ����
    unsigned int underflow[NF_INET_NUMHOOKS];
    //���飬�洢����cpu���Լ�rule�������ڴ��׵�ַ
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
* ipt_entry��rule��������׼��ƥ�����
* ipt_entry_match����չ��ƥ������ǿ�ѡ�
* ÿ��rule����һ��target����ϵͳ�Դ��ģ�Ҳ����չ�ġ�

#### match

* ͨ�õ�match���ݽṹ
```
struct xt_match
{
    struct list_head list;
    //match���֣���iptables���õ�-m������ͬ
    const char name[XT_FUNCTION_MAXNAMELEN-1];
    //����ƥ�亯��
    bool (*match)();
    /* ƥ�亯����μ�麯�� */
    bool (*checkentry)(const struct xt_mtchk_param *);
    /* Called when entry of this type deleted. */
    void (*destroy)(const struct xt_mtdtor_param *);
    const char *table;
    //match���Զ������ݳ���
    unsigned int matchsize;
};
```

* ��׼match

```
struct ipt_entry
{
    struct ipt_ip ip; //ipv4����ͨ�õ�ƥ��Ԫ��
    /*�����ͼ��rule�׵�ַ��target��ƫ����*/
    /* Size of ipt_entry + matches */
    u_int16_t target_offset;
    /* Size of ipt_entry + matches + target */
    /*��һ��rule��ƫ����*/
    u_int16_t next_offset;
    /* Back pointer */
    unsigned int comefrom;
    /*���б��ĵ�ͳ�Ƽ���*/
    struct xt_counters counters;
    /* The matches (if any), then the target. */
    unsigned char elems[0];
};

//ipv4����ͨ�õ�ƥ��Ԫ�أ�Դ/Ŀ��ip��ַ/���룬Դ/Ŀ�Ľӿ���/�ӿ������룬�����Э������
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
    //λͼ��ÿһλ��������ƥ��Ԫ������ʱ�Ƿ��Ǵ������ţ�ȡ����
    u_int8_t invflags;
};
```

* ��չmatch
![](http://blog.chinaunix.net/attachment/201405/25/26517122_1401019764FExp.jpg)

* ȫ��match��target����
`static struct xt_af *xt;`

#### target

* ��׼target
```
struct xt_standard_target
{
    struct xt_entry_target target;
    //verdict ������ָ������ֵ��Ҳ������goto����һ��rule��ƫ������
    //Ҳ������queue�Ķ��кš�
    int verdict;
};
```

* ��չtarget
![](http://blog.chinaunix.net/attachment/201405/25/26517122_1401020102yWqy.jpg)
����չmatch����
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
