# Shell 学习笔记 v1.0

Shell 是一个用C语言编写的程序，它是用户使用Linux的桥梁。Shell既是一种命令语言，又是一种程序设计语言。

Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。


## <a name='TOC'>目录</a>

  1. [脚本](#scripts)
  1. [环境](#environments)
  1. [变量](#variables)
  1. [命令](#commands)
  1. [表达式比较](#compares)
  1. [流程控制](#statements)
  1. [函数](#functions)
  1. [脚本调试](#debugs) 
  1. [参考](#references)


## <a name='scripts'>脚本</a>

Shell 脚本（shell script），是一种为shell编写的脚本程序。

## <a name='environments'>环境</a>
Shell 编程跟java、php编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。  

Linux的Shell种类众多，常见的有：

* Bourne Shell（/usr/bin/sh或/bin/sh）  
* Bourne Again Shell（/bin/bash）
* C Shell（/usr/bin/csh）
* K Shell（/usr/bin/ksh）
* Shell for Root（/sbin/sh）
* ……  

 在这里我们关注的是Bash，也就是 Bourne Again Shell，由于易用和免费，Bash在日常工作中被广泛使用。同时，Bash也是大多数Linux系统默认的Shell。

在一般情况下，人们并不区分 Bourne Shell 和 Bourne Again Shell，所以，像 **#!/bin/sh**，它同样也可以改为__#!/bin/bash__。  

***
### 第一个shell脚本
打开文本编辑器(可以使用vi/vim命令来创建文件)，新建一个文件test.sh，扩展名为sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好，如果你用php写shell 脚本，扩展名就用php好了。  

输入一些代码，第一行一般是这样：  

	#!/bin/bash  
	echo "Hello Shell" # --- John.wang

"#!" 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。  
echo命令用于向窗口输出文本。

#### 运行Shell脚本有两种方法：
**1. 作为可执行文件**    

将上面的代码保存为test.sh，并cd到相应目录:  

	chmod +x ./test.sh #使脚本具有可执行权限
	./test.sh #执行脚本

注意，一定要写成./test.sh，而不是test.sh，运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。  


**2. 作为解释器参数**  

这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：
  
	/bin/bash test.sh 
	/bin/sh   test.sh 

这种方式运行的脚本，不需要在第一行指定解释器信息，写了也没用。


## <a name='variables'>变量</a>
#### 定义变量
定义变量时，变量名不加美元符号（$），如：

    your_name="openwrt"

注意，变量名和等号之间不能有空格; 同时，变量名的命名须遵循如下规则：  

* 首个字符必须为字母（a-z，A-Z
* 中间不能有空格，可以使用下划线
* 不能使用标点符号
* 不能使用bash里的关键字(可用help命令查看保留关键字)

除了显式地直接赋值，还可以用语句给变量赋值，如：

    for file in `ls /etc`

以上语句将 /etc 下目录的文件名循环出来。

#### 内部变量
内部变量是Linux所提供的一种特殊类型的变量，这类变量在程序中用来作出判断。在shell程序内这类变量的值是不能修改的。  
部分内部变量总结如下:
   
    ---------------------------------------------------------------------------------
	$0			相当于C语言main函数的argv[0]
    ---------------------------------------------------------------------------------
	$1、$2...	这些称为位置参数（Positional Parameter），相当于C语言main函数的argv[1]、argv[2]...
    ---------------------------------------------------------------------------------
	$#			相当于C语言main函数的argc - 1，注意这里的#后面不表示注释
    ---------------------------------------------------------------------------------
	$@,$*	    表示参数列表"$1" "$2" ...，例如可以用在for循环中的in后面。
    ---------------------------------------------------------------------------------
	$?			上一条命令的Exit Status
    ---------------------------------------------------------------------------------
	$$			当前Shell的进程号
    ---------------------------------------------------------------------------------

为了测试这些变量，我们编写下面这个例子test.sh：

	#!/bin/bash  
	#my test program  
	echo "Number of parameter is $#"  
	echo "Program name is $0"  
	echo "Parameters as a single string is $*"

执行这个脚本： `./test John Wang`   
输出结果如下：

	Number of parameters is 2  
	Program name is test
	Parameters as a single string is  John Wang

#### 环境变量

Shell环境变量在shell编程方面起到很重要的作用。分析下Shell中几个比较重要的环境变量很重要。所以下面分析一下。  
每一个进程在启动的时候，系统会把环境变量传递给这个进程。  
通过env命令，可以显示系统所有的环境变量。下面列出几个常见的环境变量。

* BASH ：记录当前bash shell的路径。
* HOME ：记录当前用户的家目录，由/etc/passwd的倒数第二个域决定。
* PATH : 环境变量，显示当前PATH环境变量的内容。


#### 使用变量
使用一个定义过的变量，只要在变量名前面加美元符号即可，如：

    your_name = "openwrt"
    echo $your_name
    echo ${your_name}

变量名外面的花括号是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

    for skill in Ada Coffe Action Java do
        echo "I am good at ${skill}Script"
    done

如果不给skill变量加花括号，写成echo "I am good at $skillScript"，解释器就会把$skillScript当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

推荐给所有变量加上花括号，这是个好的编程习惯。

已定义的变量，可以被重新定义，如：

    your_name="tom"
    echo $your_name
    your_name="alibaba"
    echo $your_name

这样写是合法的，但注意，第二次赋值的时候不能写$your_name="alibaba"，使用变量的时候才加美元符（$）。

#### Shell 字符串

字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），字符串可以用单引号，也可以用双引号，也可以不用引号。

##### 单引号

    str = 'this is a string'

单引号字符串的限制：  

* 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；  
* 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。


##### 双引号

    your_name='openwrt'
    str = "Hello, I know your are \"$your_name\"! \n"
    
双引号的优点：  

* 双引号里可以有变量。
* 双引号里可以出现转义字符。


## <a name='commands'>命令</a>
`shell`的命令很多，这里重点解释`find,grep,sed,awk`等核心命令。

### find命令

    ·find   path   -option   [   -print ]   [ -exec   -ok   command ]   {} \;

find命令的参数；

`pathname: find`命令所查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录。
`-print： find`命令将匹配的文件输出到标准输出。
`-exec： find`命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' { } \;，注意{ }和\；之间的空格。
`-ok：` 和`-exec`的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

`#-print` 将查找到的文件输出到标准输出  
`#-exec   command   {} \; `  将查到的文件执行command操作,{} 和 \;之间有空格  
`#-ok 和-exec`相同，只不过在操作前要询用户

如下面这个例子：

    $ find . -name *.gif -exec ls {} \;

-exec 参数中包含了真正有价值的操作。当查找到匹配搜索条件的文件时，-exec 参数定义了将对这些文件进行什么操作。该示例告诉计算机进行如下操作：

从当前文件夹开始往下搜索，紧跟在find 之后，使用点号 (.)。  
定位所有名称以 .gif 结尾的文件（图形文件）。  
列出所有查找到的文件，使用ls命令。  

### grep命令
Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户。

格式如下：
   
    grep [options]

具体使用方法，请输入grep --help.

### sed命令

[sed][1]是一个很好的文件处理工具，本身是一个管道命令，主要是以行为单位进行处理，可以将数据行进行替换、删除、新增、选取等特定工作，下面先了解一下sed的用法  

sed命令行格式为：

    sed [-nefri] ‘command’ 输入文本 

常用选项：  

 -n∶使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN的资料一般都会被列出到萤幕上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。  
 -e∶直接在指令列模式上进行 sed 的动作编辑；  
 -f∶直接将 sed 的动作写在一个档案内， -f filename 则可以执行 filename 内的sed 动作；   
 -r∶sed 的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)  
 -i∶直接修改读取的档案内容，而不是由萤幕输出。  

常用命令：  
 a   ∶新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)  
 c   ∶取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！  
 d   ∶删除，因为是删除啊，所以 d 后面通常不接任何咚咚；  
 i   ∶插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；  
 p   ∶列印，亦即将某个选择的资料印出。通常 p 会与参数 sed -n 一起运作  
 s   ∶取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法

      
### awk命令
[awk][2]是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。

awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

awk其名称得自于它的创始人 Alfred Aho 、Peter Weinberger 和 Brian Kernighan 姓氏的首个字母。实际上 AWK 的确拥有自己的语言： AWK 程序设计语言 ， 三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。

awk命令非常复杂，需要专门进行学习，这里不作详细介绍，可参考官方手册。

## <a name='compares'>表达式比较</a>

### 逻辑运算符与逻辑表达式

#### 逻辑运算符

	逻辑卷标	 表示意思
     1.	关于档案与目录的侦测逻辑卷标！
	-f	常用！侦测『档案』是否存在 eg: if [ -f filename ]
	-d	常用！侦测『目录』是否存在
	-b	侦测是否为一个『 block 档案』
	-c	侦测是否为一个『 character 档案』
	-S	侦测是否为一个『 socket 标签档案』
	-L	侦测是否为一个『 symbolic link 的档案』
	-e	侦测『某个东西』是否存在！
	 2.	关于程序的逻辑卷标！
	-G	侦测是否由 GID 所执行的程序所拥有
	-O	侦测是否由 UID 所执行的程序所拥有
	-p	侦测是否为程序间传送信息的 name pipe 或是 FIFO （老实说，这个不太懂！）
	 3.	关于档案的属性侦测！
	-r	侦测是否为可读的属性
	-w	侦测是否为可以写入的属性
	-x	侦测是否为可执行的属性
	-s	侦测是否为『非空白档案』
	-u	侦测是否具有『 SUID 』的属性
	-g	侦测是否具有『 SGID 』的属性
	-k	侦测是否具有『 sticky bit 』的属性
	 4.	两个档案之间的判断与比较 ；例如[ test file1 -nt file2 ]
	-nt	第一个档案比第二个档案新
	-ot	第一个档案比第二个档案旧
	-ef	第一个档案与第二个档案为同一个档案（ link 之类的档案）
	 5.	逻辑的『和(and)』『或(or)』
	&&	逻辑的 AND 的意思
	||	逻辑的 OR 的意思

	运算符号	代表意义
	=	等于 应用于：整型或字符串比较 如果在[] 中，只能是字符串
	!=	不等于 应用于：整型或字符串比较 如果在[] 中，只能是字符串
	<	小于 应用于：整型比较 在[] 中，不能使用 表示字符串
	>	大于 应用于：整型比较 在[] 中，不能使用 表示字符串
	-eq	等于 应用于：整型比较
	-ne	不等于 应用于：整型比较
	-lt	小于 应用于：整型比较
	-gt	大于 应用于：整型比较
	-le	小于或等于 应用于：整型比较
	-ge	大于或等于 应用于：整型比较
	-a	双方都成立（and） 逻辑表达式 –a 逻辑表达式
	-o	单方成立（or） 逻辑表达式 –o 逻辑表达式
	-z	空字符串
	-n	非空字符串

#### 逻辑表达式

##### test 命令

使用方法：`test EXPRESSION`

    [root@localhost ~]# test 1 = 1 && echo 'ok'
    ok

>注意：所有字符 与逻辑运算符直接用“空格”分开，不能连到一起。

##### 精简表达式
* [] 表达式

    [root@localhost ~]# [ 1 -eq 1 ] && echo 'ok'           
    ok

>注意：在[] 表达式中，常见的>,<需要加转义字符，表示字符串大小比较，以acill码 位置作为比较。 不直接支持<>运算符，还有逻辑运算符|| && 它需要用-a[and] –o[or]表示

* [[]] 表达式

    [root@localhost ~]$ [[ 2 < 3 ]] && echo 'ok'  
    ok

>注意：[[]] 运算符只是[]运算符的扩充。能够支持<,>符号运算不需要转义符，它还是以字符串比较大小。里面支持逻辑运算符：|| &&.

### 数值运算符与表达式

#### 算术运算符
在Linux Shell中算术运算符包括+(加)，\-(减)， \*(乘)，/(除), %（取余），\*\*(幂)。
这些运算可以组合成符合赋值运算符，如 %=, *=, +=等。

在运算过程中，要特别注意在整数的除法运算时，其结果将舍弃整数部分，并且忽略四舍五
入，最终结果为商的整数部分。   
例：用let命令分别计算“8/5*5”与“8*5/5”，前者的结果为5，后者的结果为8。  取余和幂运算，例：用let命令分别计算“19%5”与“5**3”，前者的结果为4，后者的结果为125。  对于浮点型操作，需要用到专门的函数。
#### 位运算符
通常用于整数间的运算，位运算符是针对整数在内存中存储的二进制数据流中的位进行的操作。
位运算符：
  
    -------------------------------------------------------------------------------
	`<<`(左移)			`4<<2`				值为16（移动1位相当于乘以2） 
	`>>`(右移)			`8>>2`				值为2（移动1位相当于除以2）  
	`&`(按位与)			`8&4 `				值为0（只有两个二进制都为1时，结果才为1）  
	`|`(按位或)			`8|4 `    	        值为12（只要有一个二进制都为1时，则结果为1） 
	`~`(按位非)			`~8  `				值为-9（将二进制中的0修改为1，1修改为0）  
	`^`(按位异或)		`10^3`				值为9（两个二进制位数同时为1或0时，结果为0）
    ------------------------------------------------------------------------------- 

按位非运算符是一元运算符，无法和赋值运算符组成复合运算符。


## <a name='statements'>流程控制</a>

### 简单 if 结构

if 语句的结构如下：

    if expression
    then
        command
        command
        ...
    fi

在使用这种简单的if结构时，要特别注意测试条件后如果没有“;”，则then语句要换行，否则会产生不必要的错误。如果if 和 then可以处于同一行，则必须用“;”来终止if语句，其格式为：

    if expression; then
        command
        command
        ...
    fi

下面通过一个例子来说明if语句的用法。

    # if_exam1.sh 脚本判断输入的数是否小于15
    #!/bin/bash
    
    # 提示用户输入一个整数，
    echo "Please input a integer: "
    read integer1

    # 判断输入是否小于15，小于15将执行then 语句
    if ["$integer1" -lt 15]; then
        echo "The integer which you input is lower than 15."
    fi


### if/else/elif 结构
语法如下：

    if expression1; then
        command
        ...
    elif expression2; then
        command
        ...
    ...
    ...
    else 
       command
       ...
    fi

比较简单，不需举例说明。同时if还支持嵌套的if语句。

### case语句
case结构也可以用于多分支选择语句，常用来根据表达式的值选择要执行的语句，该语句的格式如下：

    case variable in
    value1)
        command
        ...
        ;;
    ...
    ...
    valueN)
        command
        ...
        ;;
    *)
        command
        ...
        ;;
    esac

case 结构体的变量值variable与value1, valueN 等进行一一比较，直到找到匹配的值，如果与其匹配，将执行其后的命令，遇到双分号则跳到esac后的语句执行，如果没有找到匹配的值，脚本执行默认值*)后的命令，直至双分号为止。

>注意value的值必须为常量或正则表达式。


### 列表 for 循环语句
列表for循环语句的格式如下：

    for item in {list}
    do
        command
        ...
    done
 

下面是一个例子：

    #!/bin/bash
    for item in {1..6}
    do
       echo "output, $item"
    done

for循环支持循环嵌套。

### while 循环
while 循环的格式如下：

    while expression
    do
         command
         command
         ...
    done

### until 循环语句
格式如下：

    until command #其判断条件和while正好相反，即command返回非0，或条件为假时执行循环体内的命令。
    do
        command
		...
    done



## <a name='functions'>函数</a>

Shell中函数的职能以及优势和C语言或其它开发语言基本相同，只是语法格式上的一些差异。下面是Shell中使用函数的一些基本规则：

1. 函数在使用前必须定义。  
2. 函数在当前环境下运行，它和调用它的脚本共享变量，并通过位置参量传递参数。而该位置参量将仅限于该函数，不会影响到脚本的其它地方。    
3. 通过local函数可以在函数内建立本地变量，该变量在出了函数的作用域之后将不在有效。
4. 函数中调用exit，也将退出整个脚本。  
5. 函数中的return命令返回函数中最后一个命令的退出状态或给定的参数值，该参数值的范围是0-256  之间。如果没有return命令，函数将返回最后一个Shell的退出值。  
6. 如果函数保存在其它文件中，就必须通过source或dot命令把它们装入当前脚本。  
7. 函数可以递归。  
8. 将函数从Shell中清空需要执行：unset -f function_name。  
9. 将函数输出到子Shell需要执行：export -f function_name。  
10. 可以像捕捉Shell命令的返回值一样获取函数的返回值，如$(function_name)。  

Shell中函数的声明格式如下：

    [ function ] funname [()]
    {
        action;
        [return int;]
    }

说明：

1、可以带function fun()  定义，也可以直接fun() 定义,不带任何参数。

2、参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后
跟数值n(0-255)，由于shell状态码最大是255，所以当返回值大于255时会出错。为了返回大于255的数、浮点数和字符串值，最好用函数输出到变量。

如下面的例子所示：

	 #!/bin/sh
	  
	 fSum 3 2;
	 function fSum()
	 {
	     echo $1,$2;
	     return $(($1+$2));
	 }
	 fSum 5 7;
	 total=$(fSum 3 2);
	 echo $total,$?;
	                  
	sh testfun1.sh
	testfun1.sh: line 3: fSum: command not found
	5,7
	3,2
	1
	5

### 全局变量
在shell脚本内处处有效的变量。函数中定义全局变量，那么主代码中有效。主代码中定义全局，函数中也有效。
默认情况下，脚本中定义的变量都是全局变量。  
见下面的例子：

	#!/bin/bash
	# testing the script
	function myfun {
	  value=$[ $value * 2 ]
	}
	read -p "Enter a value:" value
	myfun
	echo "The new value is:$value"

（输入45，得到90，这里的value在函数中发生变化了，到脚本中一样可以使用，而且已经变化了）

### 局部变量
作用范围只在函数当中，关键字`local`.

	#!/bin/bash
	# testing the script
	function myfun {
	  local value=$[ $value * 2 ]
	}
	read -p "Enter a value:" value
	myfun
	echo "The new value is:$value"

（输入45，输出45。因为加上local关键字之后，函数中的value是局部变量，与外界无关）


## <a name='debugs'>脚本调试</a>

### echo打桩调试
这里仅仅是提供一些常规性的调试方法，最简单的就是使用echo函数打印出变量的值从而达到调试目的。

### shell 调试命令
shell脚本执行可以通过./shell-filename.sh的形式执行，另外的一种形式是通过bash ./shell-filename.sh的形式执行，同时在执行脚本时，可以使用-v或者是-x参数打印程序执行信息。

-v：默认打印shell读取的内容

-x：将命令“展开” 之后打印

例如对于下面的程序：

	!/bin/sh
	# for debug shell script
	#
	tot=`expr $1 + $2`
	echo $tot

如果使用-v选项，结果如下：

    root:~/shell$ sh -v ./debug.sh 2 5

	#!/bin/sh
	#
	# for debug shell script
	#
	tot=`expr $1 + $2`
	echo $tot
	7

如果是使用-x选项的话，结果：
 
    root:~/shell$ sh -x ./debug.sh 2 5
    
    expr 2 + 5
    tot=7 
    echo 7
    7

shell还有一个不执行脚本只检查语法的模式，命令如下：

    sh -n your_script

这个命令会返回所有语法错误。

## <a name='references'>参考</a>
《Linux Shell编程》  

http://www.gnu.org/software/sed/manual/sed.html  
http://www.gnu.org/software/gawk/manual/gawk.html  

