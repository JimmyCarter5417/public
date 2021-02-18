* 状态机

	* 概念
		* 有限自动机FA
			* 确定性有限自动机DFA
			* 非确定性有限自动机NFA
		* 有限状态集
		* 输入字符集
		* 转移函数
		* 起始状态
		* 接受状态
		* 从**起始状态**，接受一系列输入字符，可以转移到**接受状态**，即认为这一系列字符可以**被自动机接受**
		* 如果两台自动机能够**接受**的输入字符串完全相同，则这两台自动机**等价**
		* 对于每一个NFA，都存在与之等价的DFA
	* 区别
		* NFA中一个状态可以不经过任何符号就可以实现状态转换(即存在ε-转移)
		* 对于一个特定的符号输入，DFA只会跳转到一个状态，而NFA则可能跳转到多个状态
	* NFA
		* 需要记录中间状态，可回溯
		* 广度优先搜索BFS 或 深度优先搜索DFS
		* 从正则表达式翻译过来的状态机是ε-NFA
	* DFA
		* 速度快，内存占用大
		* 对DFA进行简化，求出最简DFA
		* 简化的本质是合并性质相同的状态，以减少整个图的大小

------------
* AC

	https://blog.csdn.net/Changxing_J/article/details/104698941
	* 在**前缀树**的基础上，为前缀树的每个节点建立一棵**后缀树**
	* AC自动机：有向有环图
	* 状态 -> 节点
	* 若字符匹配成功，则按goto表转移到下一状态；若转移的状态对应有output，则输出已匹配上的模式串
	* 若字符匹配失败，则递归地按自动机的fail表进行转移
	* goto表
		* 模式串创建的前缀树
		* 可接受任意字符
	* output表
		* 记录**状态**是否对应**某个或某些模式串**
		* 可看作是状态的一个成员变量
	* fail表
		* 保存状态间一对一的关系，存储状态转移失败后应当回退的**最佳状态**（能记住的已匹配上的字符串存在于goto表中的最长后缀的那个状态）
		* **初始状态**和**与初始状态直接相连的所有状态**，其fail指针都指向初始状态
		* 从初始状态开始进行广度优先遍历（BFS），若当前状态S接受字符c直达的状态为T，则沿着S的fail指针回溯，直到找到第一个前驱状态F，使得F.goto(c)!=null。将T的fail指针设为F.goto(c)。简单来说，就是寻找状态S的存在于goto表中的最长后缀的状态F
		* 将F的output添加到T的output中（状态5的he）

		![](https://img-blog.csdnimg.cn/20200306163844494.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NoYW5neGluZ19K,size_16,color_FFFFFF,t_70)

------------
* BM

	https://blog.csdn.net/DBC_121/article/details/105569440
	https://segmentfault.com/a/1190000022490177
	https://www.cnblogs.com/lanxuezaipiao/p/3452579.html
	https://yfsyfs.gitee.io/2019/06/25/%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8C%B9%E9%85%8D%E4%B9%8B-BM%E7%AE%97%E6%B3%95/
	* 增加匹配失败后向右移动的字符数，减少无效匹配次数
	* 从右向左匹配
	* 坏字符规则：
		* 当文本串中的某个字符跟模式串的某个字符不匹配时，我们称文本串中的这个失配字符为坏字符
		* 右移位数 = 坏字符在模式串中的位置 - **坏字符在模式串中最右出现的位置**
		* 如果坏字符不包含在模式串之中，则最右出现位置为-1
		* 附：
			* 坏字符可能在模式串中存在多个，为避免错过正确匹配的情况，应该移动更少的位数，移动至更靠后的那个坏字符处
			* 文本串 `a a a a a a a a` ，模式串 `b a a a`，会计算出负值！
	* 好后缀规则：
		* 右移位数 = 好后缀在模式串中的位置 - **好后缀在模式串上一次出现的位置**
		* 如果好后缀在模式串中没有再次出现，则为-1
		* 三种情况：
			1. 好后缀在模式串中，那么移动模式串至好后缀匹配的地方
			2. 好后缀不在模式串中，并且好后缀的后缀子串和模式串的前缀子串无重合部分，那么直接移动模式串至好后缀的后一位
			3. 好后缀不在模式串中，但是好后缀的后缀子串和模式串的前缀子串有重合部分，那么需要移动模式串至和好后缀的后缀子串重合的地方

			![](https://image-static.segmentfault.com/322/196/3221961855-ad540b88f12e3eb3_articlex)
	* 分别计算坏字符规则和好后缀规则往右滑动的位数，取最大值

	![](https://img-blog.csdnimg.cn/20200417165619199.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0RCQ18xMjE=,size_16,color_FFFFFF,t_70#pic_center)
	
	* 计算坏字符数组bmBc[]
		* 找到坏字符在模式串中出现的最右边的位置
		```
		void PreBmBc(char *pattern, int m, int bmBc[])
		{
			int i;

			for(i = 0; i < 256; i++)
			{
				bmBc[i] = m;
			}

			for(i = 0; i < m - 1; i++)
			{
				bmBc[pattern[i]] = m - 1 - i;
			}
		}
		```
	* 计算好后缀数组bmGs[]和suffix[]
		* bmGs[]：如果在pattern[i]不匹配的话，模式串应该向右移动的距离

		![](https://yfsyfs.gitee.io/2019/06/25/%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8C%B9%E9%85%8D%E4%B9%8B-BM%E7%AE%97%E6%B3%95/5.png)

		* suffix[]：从它出发的能与后缀匹配的最大长度

		![](https://yfsyfs.gitee.io/2019/06/25/%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%8C%B9%E9%85%8D%E4%B9%8B-BM%E7%AE%97%E6%B3%95/8.png)

------------
* WM

------------
* KMP

