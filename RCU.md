https://www.cnblogs.com/LoyenWang/p/12681494.html  
https://www.cnblogs.com/LoyenWang/p/12770878.html  

#### 特点
* 读写锁替代品
    * 读者之间不需要同步
    * 写者之间需要同步
    * 读者写者之间不需要直接同步
    * 最大程度减少读者开销，常用于读者性能要求高的场景
* 优点
    * 读者开销很少，不需要获取任何锁，不需要执行原子指令或内存屏障
    * 没有死锁问题
    * 没有优先级反转问题
    * 很好的实时延迟
* 缺点
    * 写者之间同步开销比较大
    * 使用复杂

#### 三个元素
![](https://img2020.cnblogs.com/blog/1771657/202004/1771657-20200411183349989-1834656562.png)  
* Reader  
    * 使用rcu_read_lock和rcu_read_unlock来界定读者临界区  
    * 使用rcu_dereference来获取RCU-protected指针  
    * 当使用不可抢占的RCU时，rcu_read_lock/rcu_read_unlock之间不能使用可以睡眠的代码（？？？？？？）  
* Updater  
    * Update操作分为：Removal移除，Reclamation回收  
    * 多个Updater更新数据时，需要使用互斥机制进行保护  
    * Updater使用rcu_assign_pointer来移除旧的指针指向，指向更新后的临界资源  
    * Updater使用synchronize_rcu或call_rcu来启动Reclaimer，对旧的临界资源进行回收  
* Reclaimer  
    * Reclaimer回收旧的临界资源  
    * 为了确保没有读者正在访问要回收的临界资源，Reclaimer需要等待所有的读者退出临界区，这个等待的时间叫做宽限期（Grace Period）  
  
#### 三种机制  
* 发布订阅  
![](https://img2020.cnblogs.com/blog/1771657/202004/1771657-20200411183432761-182154843.png)  
    * updater发布  
        * rcu_assign_pointer  
    * reader订阅  
        * rcu_dereference  
    * 两个版本临界资源  
![](https://img2020.cnblogs.com/blog/1771657/202004/1771657-20200411183605937-603269025.png)  
        * 在Updater更新后Reclaimer回收前，存在新旧两个版本的临界资源  
        * 在synchronize_rcu返回后，Reclaimer对旧的临界资源进行回收，最后剩下一个版本  
    * 宽限期  
![](https://img2020.cnblogs.com/blog/1771657/202004/1771657-20200411183529886-207985640.png)  
        * 从Removal结束到Reclamation开始之间的间隔  
        * 宽限期的结束代表着Reader都已经退出了临界区，因此回收工作也就是安全的  
        * 宽限期是否结束，与CPU执行状态有关：“静止状态”  
![](https://img2020.cnblogs.com/blog/1771657/202004/1771657-20200424230751096-891760571.png)  
            * 当CPU正在访问RCU保护的临界区时，认为是“活动状态“  
            * 当CPU离开临界区后，则认为它是“静止状态”  
                * 在时钟tick中检测CPU处于用户模式或者idle模式  
                * 在不支持抢占的RCU中，检测到CPU有context切换  
                * 当所有的CPU都至少经历过一次QS后，宽限期结束并触发回收工作  
        * 在synchronize_rcu调用后，此时可能还有新的读者来读取临界资源，但是Grace Period只等待Pre-Existing的读者  
        * 只要这些之前就存在的RCU读者退出临界区后，意味着宽限期的结束，因此就进行回收处理工作了  
        * synchronize_rcu并不是在最后一个Pre-ExistingRCU读者离开临界区后立马就返回，它可能存在一个调度延迟  
  
#### 实现  
* 经典RCU  
    * 全局的cpumask位图记录静止状态  
    * 当CPU数量很大时锁争用会带来很大开销  
* tree RCU  
    * 将CPU分组，64个CPU一组  
    * 本小组CPU争用同一个锁  
    * 当本小组某个CPU经历了一个QS后，将对应的位清除  
    * 如果本小组最后一个CPU经历完QS后，那么将上一层对应该组的位清除  
  
#### 三种RCU  
* 可抢占RCU  
    * rcu_read_lock/rcu_read_unlock  
    * 在读端临界区可以被其他进程抢占  
* 不可抢占RCU(RCU-sched)  
    * rcu_read_lock_sched/rcu_read_unlock_sched
    * 在读端临界区不允许其他进程抢占
* 关下半部RCU(RCU-bh)
    * rcu_read_lock_bh/rcu_read_unlock_bh  
    * 在读端临界区禁止软中断  
  
#### 优先级反转  
* （信号量）  
* 定义  
    * 高优先级任务打断低优先级任务  
    * 高优先级任务被低优先级任务阻塞  
    * 高优先级任务迟迟得不到调度，但其他中等优先级的任务却能抢到CPU资源  
    * 从现象上来看，好像是中优先级的任务比高优先级任务具有更高的优先权  
* 解决方法  
    * 优先级天花板  
    * 优先级继承  
    * 二者结合  
