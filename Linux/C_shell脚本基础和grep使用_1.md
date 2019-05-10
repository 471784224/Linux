# shell脚本基础和grep使用

***



## 一、shell脚本基础

### （1）、命令hash

**hash的作用：**缓存此前命令的查找结果：key-value数据格式（键值），key:搜索键；value:值

**hash的使用：**

1. 直接使用hash，列出缓存的所有命令
2. -d选项，hash -d COMMAND：表示清空指定命令的缓存,如hash -d ls
3. -r选项，表示清空所有命令的缓存，hash -r

### （2）、编程语言

 	强类型变量编程语言
                   

​	 弱类型变量编程语言：
​                                 

​				（1）bash把所有变量统统视作字符型;
​                                      

​					  在指明做算数加减运算时自动转换为数字类型
​                                      

​                                          不支持浮点数据，除非借助于外在工具
​                                  

​				（2）bash中的变量无需事先声明;相当于，把声明和赋值过程同时实现；

​							变量声明：类型，变量

### （3）、变量

**程序是由指令加数据组成**，指令:由程序文件提供；数据（来源）：常量，IO设备，文件，管道、变量等。

​	文件在被程序处理时，将文件加载到内存中进行修改，当文件要被修改多次的时候，存放文件的内存空间会依次序对文件进行处理，处理完成后存放到内存中的另一个位置
，文件第一阶段处理完成后，第二阶段和接下来的后续步骤要完成第一阶段相同的步骤，文件加载到内存中暂存的是同一个内存空间，这个内存空间我们取名叫做变量，变量指向一个内存空间。



**程序的另一种组成表示方法：程序是由算法加数据结构组成。**

​		算法：设计解决问题的路径
​              

​		数据结构：数据以特定的结构组织起来

**变量是由变量名加指向的内存空间组成。**

 **变量赋值**:shell中定义为name=value

**变量类型**可以以数据的存储格式、表示范围、参与的运算三个方面进行分类。

**(1)、以数据的存储格式分**：

                字符型
                数值型
                      整型
                      浮点型
                          单精度浮点型
                          双精度浮点型
                
                布尔型(true,false)
**(2)、以作用范围分，bash变量类型：**

本地变量:     作用域为当前shell进程
                

环境变量:     作用域为当前shell进程及其子进程
                

局部变量:     作用域仅为shell进程中的某一个代码片段（函数上下文）
                

位置参数变量:  向执行脚本的shell进程传递的参数;
            

特殊变量:  shell内置的有特殊公用的变量;

```
                 前面讲过一个特殊变量 $?:
                     $?:
                         0:成功
                         1-255：失败
```

**变量替换**

把变量名出现的位置替换为其所指向的内存空间中的数据的过程；
           

变量替换的表示方式叫变量引用
       

bash变量引用：${var_name}或$var_name

**变量名**

变量名只能包含数字、字母和下划线，而且不能以数字开头:

变量名：见名知义，命名机制遵循某种法则;不能够使用程序(编程语言)的保留字，例如if,else,then,while..

**定义变量**

**（1）定义本地变量**

```
本地变量：
                    本地变量赋值:  name=value
                    变量引用:  ${var_name}，$var_name
                        "":变量名会替换为其直
                        '':变量名不会替换为其值
                    查看变量: set命令
                    撤销变量：unset name（注意此处非变量引用，不能使用$符）
```

**（2）定义环境变量**

```
环境变量：
                     变量赋值：
                          (1) export name=value
                          (2) name=value
                              export name
                          (3) declare -x name=value 
                          (4) name=value
                              declare -x name 
                     变量引用:${name},$name

                     注意:bash内嵌了许多环境变量（通常为全大写字符），用于定义bash的工作环境
PATH,HISTFILE,HISTSIZE,HISTFILESIZE,HISTCONTORL,SHELL,HOME,UID,PWD,OLDPWD
                
                     查看环境变量：export,declare -x,printenv,env
                     撤销环境变量：unset name 
                     
                       declare命令 shell内嵌命令
                          Set variable values and attributes.设置变量值和属性。
						      -a	to make NAMEs indexed arrays (if supported)
						      -A	to make NAMEs associative arrays (if supported)
						      -i	to make NAMEs have the `integer' attribute
						      -l	to convert NAMEs to lower case on assignment
						      -r	to make NAMEs readonly
						      -t	to make NAMEs have the `trace' attribute
						      -u	to convert NAMEs to upper case on assignment
						      -x	to make NAMEs export
```

**（3）只读变量**

    只读变量：
                       (1) declare -r name
                       (2) readonly name
    
                       只读变量无法重新复制，并且不支持撤销;存活时间为当前shell进程的生命周期，随shell进程终止而终止;


### （4）、逻辑运算

~]# COMMAND1; COMMAND2; COMMAND3; ...

我们用这样的方式可以表示命令顺序执行



**（1）逻辑运算符**

运算数： 真(true, yes, on, 1)
                     假(false, no, off, 0)

    逻辑运算符
    
    与：&&
                  1 && 1 = 1
                  1 && 0 = 0
                  0 && 1 = 0
                  0 && 0 = 0
             
    或：||
                  1 || 1 = 1
                  1 || 0 = 1
                  0 || 1 = 1
                  0 || 0 = 0
             
    非：！ 
                 ！ 1 = 0
                 ！ 0 = 1
             
             异或:运算数相同则为0，不同则为1
**（2）短路法则**

          ~]# COMMAND1 && COMMAND2
               COMMAND1为"假"，则COMMAND2不会再执行；（为真表示命令执行成功，为假表示命令执行失败）
               否则，COMMAND1为"真"，则COMMAND2必须执行
    
          ~]# COMMAND1 || COMMAND2
               COMMAND1为"真"，则COMMAND2不会再执行；
               否则，COMMAND1为"假"，则COMMAND2必须执行
    
               示例：
                  ~]# id $username || useradd $username
example.

				[root@bogon ~]# touch /tmp/test.etc1 && ls /etc > /tmp/test.etc1
				[root@bogon ~]# cat /tmp/test.etc1
				abrt
				adjtime
				aliases
				aliases.db
				alsa
				alternatives
				anacrontab
				asound.conf
				at.deny
				.
				.
				.
	
				[root@bogon ~]# touch123 /tmp/test.etc2 && ls /etc > /tmp/test.etc2
				bash: touch123: 未找到命令...
				[root@bogon ~]# cat /tmp/test.etc2
				cat: /tmp/test.etc2: 没有那个文件或目录            
				[root@bogon jacky]# id user1
				id: user1: no such user
				[root@bogon jacky]# id user1 || useradd user1
				id: user1: no such user
				[root@bogon jacky]# id user1
				uid=5014(user1) gid=5015(user1) 组=5015(user1)
				[root@bogon jacky]# id user1 || useradd user1
				uid=5014(user1) gid=5015(user1) 组=5015(user1)




### （5）、shell脚本编程

#### 1.编程语言的分类：

A、编程语言根据运行方式可以分为下列2种

​       1、编译运行:源代码需要由编译器转换为程序文件，而后才能运行，源代码-->编辑器(编译)-->程序文件
​           C语言：

​       2、解释运行:源代码-->运行时启动解释器，由解释器边解释边运行；（源代码本身不可运行，由解释器将源代码的数据处理运行）

B、根据其编程过程中功能的实现是调用库还是调用外部的程序文件分类

​        1、脚本编程（此处为shell脚本编程）：利用系统上的命令及编程组件进行编程;
​       

​	2、完整编程：利用库或编程组件进行编程;

C、据编程模型分类：

程序=指令+数据

1、过程式编程语言

过程式：以指令为中心来组织代码，数据是服务于代码；

                 顺序执行
                 选择执行
                 循环执行
                 代表：C,bash
2、 面向对象的编程语言

 对象式：以数据位中心来组织代码，围绕数据来组织指令；

                 类（class）:同一类对象的抽象集合，可以实例化对象，
                 方法（method）:能够对对象额外施加的操作：顺序，选择，循环
                 对象
                 代表：Java,C++,Python
#### 2.shell脚本编程特性

1、shell脚本编程是过程式编程

2、解释运行

3、依赖于外部程序文件

#### 3.如何写shell脚本

格式：

脚本文件的第一行，顶格：给出shebang,解释器路径，用以指明解释执行当前脚本的解释器程序文件
                 

常见的解释器:
                    

#!/bin/bash
                     

#!/usr/bin/python
                    

#!/usr/bin/perl

**文本编辑器：nano**

在这里我们可以用nano进行shell文件的创建和编辑。

     知识延伸：
     行编辑器：sed
     全屏幕编辑器：nano,vi,vim
**shell脚本是什么**

简单的来讲，shell脚本就是命令的堆砌。但是，很多命令不具有幂等性，需要用程序逻辑来判断运行条件是否满足，以避免其运行中发生错误。

**如何运行脚本**

 (1) 赋予执行权限，并直接运行此程序文件即可；
                

​	chmod +x /PATH/TO/SCRIPT_FILE
​               

​	/PATH/TO/SCRIPT_FILE
​          

 (2) 直接运行解释器，将脚本以命令行参数传递给解释器程序；
               

 	bash /PATH/TO/SCRIPT_FILE
               

    注意：
    脚本中的空白行会被解释器忽略；
    脚本中，除了shebang,余下所有以#开头的行，都被视作注释行而被忽略；此即为注释行；
    shell脚本的运行是通过运行一个子shell进程实现的；
    example：
    		写一个脚本，实现如下功能;
         （1）显示/etc/目录下所有以大写P或小写p开头的文件或目录本身; 
         （2）显示/var目录下的所有文件或目录本身，并将显示结果中的小写字母转换为大写后显示；
         （3）创建临时文件/tmp/myfile.XXXX;  
    
    		~]# nano test_script.sh


​    		
​    		
    		#!/bin/bash
    		
    		echo "Show some under /etc"
    		ls -d /etc/[pP]*
    
    		echo "Translate lower to upper"
    		ls -d /var/* | tr 'a-z' 'A-Z'
    
    		echo "create temp file"
    		mktemp /tmp/myfile.XXXX

#### 4.bash的配置文件

**bash配置文件分两类**

1、profile类：为交互式登录的shell进程提供配置

2、bashrc类：为非交互式登录的shell进程提供配置

**登录类型**

1、交互式登录shell进程：
            

​		直接通过某终端输入账号和密码后登录打开的shell进程
​            

​		使用su命令：su - USERNAME,或者使用su -l USERNAME执行的登录切换

2、非交互式登录shell进程：
            

​		su USERNAME执行的登录切换；
​            

​		图形界面下打开的终端；
​            

​		运行脚本时起的shell；

**登录shell进程之后查找应用配置文件的顺序**

    交互式登录shell进程：
               /etc/profil e--> /etc/profile.d/*.sh --> ~/.bash_profile --> ~/.bashrc--> /etc/bashrc
    
    非交互式登录shell进程：
               ~/.bashrc --> /etc/bashrc --> /etc/profile.d/*.sh
**profile类：**

1、全局配置文件：对所有用户都生效；
     （注意：仅管理员可修改全局配置文件)              

​	/etc/profile
​                  

​	/etc/profile.d/*.sh  (开机运行脚本放到这里方便管理，单独定义环境变量也可以放在这里例如定义JAVA_HOME环境变量)
​                   

2、用户个人配置文件
                     

​	~/.bash_profile

3、profile类配置文件的功用：
                  

​	A、用于定义环境变量；
​                   

​	B、开机运行命令或脚本；

**bashrc类：**

 1、 全局：

​	 /etc/bashrc

 2、用户个人：

​	 ~/.bashrc

 3、功用：
                   

​	1、定义本地变量；
​                   

​	2、定义命令别名；

**生命周期**

命令行中定义的特性，例如变量和别名作用域为当前shell进程的生命周期；
          

配置文件定义的特性，只对随后新的shell进程有效；用以下方法让通过配置文件定义的特性立即生效；

    （1）通过命令行重复定义一次；
    （2）让shell进程重读配置文件；
        ~]# source /PATH/FROM/CONF_FILE
    （3）~]# . /PATH/FROM/CONF_FILE


## 二、文本处理工具grep



### （1）、Linux文本处理三剑客

​     **grep，egrep,fgrep**:文本过滤工具（模式：pattern）工具；

​       			grep:支持基本正则表达式，-E，-F

​       			egrep:扩展正则表达式，-G，-F

​                               fgrep:不支持正则表达式
​      

​     **sed**:stream editor,流编辑器；文本编辑工具；
​       

​     **awk**:Linux上的实现为gawk（GNU awk）,文本报告生成器(格式化文本);

### （2）、正则表达式

正则表达式：Regual Expression,REGEXP

由一类特殊字符及文本字符所编写的模式，其中有些字符不表示其字面意义，而用于表示控制或通配的功能;

分为两类：

​                    基本正则表达式：BRE(basic regual expression)

​                    扩展正则表达式：ERE(extended regual expression)

​         元字符:

​			举例：` \(hello[[:space:]]\+\)\+`

### （3）、grep

    grep：Gloabal search REgulaer expression an Print out the line.全局搜索正则表达式并输出相应行
           作用：文本搜索工具，genuine用户指定的“模式（过滤条件）”对目标文本逐行进行匹配检查；打印匹配到的行；
    
           模式：由正则表达式的元字符及文本字符所编写出的过滤条件；（grep默认使用基本正则表达式，加选项-E使用扩展正则表达式）
    
           正则表达式引擎：
    
           grep [OPTIONS] PATTERN [FILE...]
           grep [OPTIONS] [-e PATTERN | -f FILE] [FILE...]
    
                OPTIONS:
                  --color=auto:对匹配到的文本着色后高亮显示；
                  -i: ignorecase，忽略字符的大小写
                  -o: 仅显示匹配到的字符串本身；
                  -v,--invert-match: 显示不能被模式匹配到的行
                  -E: 支持使用扩展的正则表达式元字符
                  -q,--quiet,--slient:静默模式，即不输出任何信息；
    
                  -A #：after,同时显示匹配到的行的后#行
                  -B #：before,前#行
                  -C #：context,前后各#行
选项参考

		-a 不要忽略二进制数据。
		-A<显示列数> 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
		-b 在显示符合范本样式的那一行之外，并显示该行之前的内容。
		-c 计算符合范本样式的列数。
		-C<显示列数>或-<显示列数>  除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
		-d<进行动作> 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
		-e<范本样式> 指定字符串作为查找文件内容的范本样式。
		-E 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
		-f<范本文件> 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
		-F 将范本样式视为固定字符串的列表。
		-G 将范本样式视为普通的表示法来使用。使用基本正则表达式
		-h 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
		-H 在显示符合范本样式的那一列之前，标示该列的文件名称。
		-i 忽略字符大小写的差别。
		-l 列出文件内容符合指定的范本样式的文件名称。
		-L 列出文件内容不符合指定的范本样式的文件名称。
		-n 在显示符合范本样式的那一列之前，标示出该列的编号。
		-q 不显示任何信息。
		-R/-r 此参数的效果和指定“-d recurse”参数相同。
		-s 不显示错误信息。
		-v 反转查找。
		-w 只显示全字符合的列。
		-x 只显示全列符合的列。
		-y 此参数效果跟“-i”相同。
		-o 只输出文件中匹配到的部分。
### （4）、 基本正则表达式元字符

#### 1.字符匹配

​            .： 匹配任意单个字符；

 	  []：匹配指定范围内的任意单个字符

​		  有几种特殊格式如下：

                 [a-z]或[A-Z]：所有字母（范围匹配时不区分字母大小写）
                 [0-9]:    0-9所有数字
                 [a-z0-9]:a到z,0到9(所有数字和字母)
                 [abcxyz]: a，b，c，x，y，z中的任意单个字符，区分大小写
                 [[:upper:]]: 所有大写字母
                 [[:lower:]]: 所有小写字母
                 [[:alpha:]]: 所有字母
                 [[:digit:]]: 所有数字
                 [[:alnum:]]: 所有的字母和数字
                 [[:space:]]: 所有空白字符
                 [[:punct:]]: 所有标点符号
​	   [^]：匹配指定范围外的任意单个字符

             	 [^[:upper:]]   匹配非大写字母以外的任意单个字符
              	 [^0-9]         匹配数字以外的任意单个字符（非数字字符）
                 [^[:alnum:]]   匹配字母和数字以外的任意单个字符（例如空白符合标点）
#### 2.匹配次数

用在要指定其出现的次数的字符的后面，用于限制其前面字符出现的次数；默认工作于贪婪模式(能匹配多少个字符就会匹配多少)；

         *:匹配其前面的字符任意此；0,1,多次;
                        例如:grep "x*y"
                            abxy
                            aby
                            xxxxxy
                            yab
         .*: 匹配任意长度的任意字符
         \?: 匹配其前面的字符0次或1次;即其前面的字符可有可无;
         \+: 匹配其前面的字符1次或多次;即其前面的字符要出现至少1次;
         \{m\}: 匹配其前面的字符m次;
         \{m,n\}: 匹配其前面的字符至少m次，至多n次;
         \{0,n\}:至多n次
         \{m,\}:至少m次

example.

	[root@bogon ~]# nano grep.txt
	[root@bogon ~]# grep "x*y" grep.txt 
	                           abxy
	                            aby
	                            xxxxxy
	                            yab
	[root@bogon ~]# grep "x\?y" grep.txt 
	                           abxy
	                            aby
	                            xxxxxy
	                            yab
	[root@bogon ~]# grep "x\+y" grep.txt 
	                           abxy
	                            xxxxxy
	[root@bogon ~]# grep "x\1y" grep.txt 
	grep: 无效的向后引用
	[root@bogon ~]# grep "x\{1}y" grep.txt 
	grep: 不匹配的 \{
	[root@bogon ~]# grep "x\{1\}y" grep.txt 
	                           abxy
	                            xxxxxy
	[root@bogon ~]# grep "x\{2\}y" grep.txt 
	                            xxxxxy
	[root@bogon ~]# grep "x\{2,5\}y" grep.txt 
	                            xxxxxy
	[root@bogon ~]# grep "x\{1,5\}y" grep.txt 
	                       	   abxy
	                            xxxxxy
#### 3.位置锚定

^: **行首锚定**; 用于模式的最左侧

$: **行尾锚定**; 用于模式的最右侧

^PATTERN$: 用于 PATTERN来匹配整行;

​		^$: 空白行；

​		^[[:space:]]*: 空行或包含空白字符的行;

	    example.
								[root@bogon ~]# grep root /etc/passwd
								root:x:0:0:root:/root:/bin/bash
								operator:x:11:0:operator:/root:/sbin/nologin
								rootkit:x:5016:5017::/home/rootkit:/bin/bash
								user7:x:5017:5018::/home/user7:/bin/chroot
								chrooter:x:5018:5019::/home/chrooter:/bin/bash
								[root@bogon ~]# grep "^root" /etc/passwd
								root:x:0:0:root:/root:/bin/bash
								rootkit:x:5016:5017::/home/rootkit:/bin/bash
								[root@bogon ~]# grep "root$" /etc/passwd
								user7:x:5017:5018::/home/user7:/bin/chroot
**单词**: 非特殊字符组成的连续字符(字符串)都称为单词；

    \<或\b: 词首锚定，用于单词模式的左侧;
    \>或\b: 词尾锚定，用于单词模式的右侧;
    \<PATTERN\>:匹配完整单词；
    	example.
    					[root@bogon ~]# grep "\<root" /etc/passwd
    					root:x:0:0:root:/root:/bin/bash
    					operator:x:11:0:operator:/root:/sbin/nologin
    					rootkit:x:5016:5017::/home/rootkit:/bin/bash
    					[root@bogon ~]# grep "\broot" /etc/passwd
    					root:x:0:0:root:/root:/bin/bash
    					operator:x:11:0:operator:/root:/sbin/nologin
    					rootkit:x:5016:5017::/home/rootkit:/bin/bash
    					[root@bogon ~]# grep "root\>" /etc/passwd
    					root:x:0:0:root:/root:/bin/bash
    					operator:x:11:0:operator:/root:/sbin/nologin
    					user7:x:5017:5018::/home/user7:/bin/chroot                        
    					[root@bogon ~]# grep "\<root\>" /etc/passwd
    					root:x:0:0:root:/root:/bin/bash
    					operator:x:11:0:operator:/root:/sbin/nologin
#### 4.分组及引用

    \(\): 将一个或多个字符捆绑在一起，当作一个整体进行处理;
          例如:
          		\(xy\)*ab
    
    Note: 分组括号中的模式匹配到的内容会被正则表达式引擎自动记录与内部的变量中，这些变量为:
    \1: 模式从左侧起，第一个左括号以及与之匹配的右括号之间的模式所匹配到的字符;
    \2: 模式从左侧起，第二个左括号以及与之匹配的右括号之间的模式所匹配到的字符;
    \3:
    ...
**后向引用**: 引用前面的分组括号中的模式所匹配到的字符;

    		example.
    				He likes his lover.
    				He loves his lover.
    				She likes her liker.
    				She love her liker.
                     
    			[root@localhost tmp]# grep "\(l..e\).*\1" loves.txt
    			He loves his lover.
    			She likes her liker.
### （5）、egrep

egrep支持扩展的正则表达式实现类似于grep文本过滤功能；grep -E 实现同样的功能.

         命令：egrep [OPTIONS] PATTERN [FILE...]
    
              OPTIONS:
                  --color=auto:对匹配到的文本着色后高亮显示；
                  -i: ignorecase，忽略字符的大小写
                  -o: 仅显示匹配到的字符串本身；
                  -v,--invert-match: 显示不能被模式匹配到的行
                  -E: 支持使用扩展的正则表达式元字符
                  -q,--quiet,--slient:静默模式，即不输出任何信息；
    
                  -A #：after,同时显示匹配到的行的后#行
                  -B #：before,前#行
                  -C #：context,前后各#行
              扩展正则表达式的元字符:
                字符匹配
                      .:任意单个字符
                     []:指定范围内的任意单个字符
                     [^]指定范围外的任意单个字符
    
                次数匹配
                     *：任意次，0,1或多次；
                     ?:0次或1次，其前的字符是可以可恶的;
                     +:其前字符至少1次；
                     {m}:其前的字符m次；
                     {m,n}:至少每次，至多n次;
                         {0,n}
                         {m,}
    
                位置锚定
                    ^: 行首锚定;
                    $: 行尾锚定;
                    \<,\b: 词首锚定;
                    \>,\b: 词尾锚定;
                
                分组及引用;
                    (): 分组; 括号内的模式匹配到的字符会被记录与正则表达式引擎的内部变量中;
                    向后引用: \1,\2,...         (只能在正则表示中引用)
                
                或：
                     a|b: a或者b;
                     C|cat: C或cat
                     (C|c)at: cat或Cat

### （6）、fgrep

​     fgrep: 不支持正则表达式元字符;

​     当无需要用到元字符去编写模式时，使用fgrep性能更好；

## 三、其他文本查看及处理工具

### （1）、wc

  wc: word count 单词统计， print newline, word, and byte counts for each file

    命令：wc [OPTION]... [FILE]...
    	   -l: lines，行数
    	   -w: words，单词数
    	   -c: bytes,字节数
    	example.
    		        [root@bogon ~]# wc /etc/fstab 
    				 12  60 541 /etc/fstab
    				[root@bogon ~]# wc -l /etc/fstab 
    				12 /etc/fstab
    				[root@bogon ~]# wc -w /etc/fstab 
    				60 /etc/fstab
    				[root@bogon ~]# wc -c /etc/fstab 
    				541 /etc/fstab
### （2）、CUT

   cut:文本截取工具

​	cut - remove sections from each line of files

    命令：cut OPTION... [FILE]...
             OPTION:
                -d CHAR: 以指定的字符为分隔符；-d后面可以不跟空格（-d'' 表示以空格为分隔符）
                -f FIELDS: 挑选出的字段；
                    #: 指定的单个字段;
                    #-#: 连续的多个字段;
                    #,#: 离散的多个字段;
       example. 
             ~]# cut -d: -f1,3-5,7 /etc/passwd
### （3）、sort

sort:排序命令

    sort - sort lines of text files
         命令： sort [OPTION]... [FILE]...
              		-n:基于数值大小进行排序，而非字符
          	  		-t CHAR: 指定字段分隔符;
          	  		-k #: 用于排序比较的字段
          	  		-r: 逆序排序;
          	  		-f: 忽略字符大小写
          	  		-u, --unique: 重复的行只保留一份; 
          	       			重复:连续且相同
    example.
          	   A:
          	    sort -t: -k3 -n /etc/passwd      //将用户信息以UID的数值大小排序
          	   B：
    			[root@bogon ~]# cut -d: -f7 /etc/passwd | sort -u
    			/bin/bash
    			/bin/chroot
    			/bin/csh
    			/bin/sync
    			/bin/tcsh
    			/bin/zsh
    			/sbin/halt
    			/sbin/nologin
    			/sbin/shutdown
    			[root@bogon ~]# cut -d: -f7 /etc/passwd | sort -u | wc -l         
    			9
    				//表示当前系统的用户有几种在使用的shell(Linux哲学思想之一组合简单命令完成复杂任务)
### （4）、uniq

   uniq:报告或移除重复的行

    命令： uniq [OPTION]... [INPUT [OUTPUT]]
           			-c, 显示每一行重复出现的次数
           			-u, 仅显示未曾重复过的行
           			-d, 仅显示存在重复的行
    example.
    			[root@localhost ~]# cut -d: -f7 /etc/passwd | sort | uniq
    			/bin/bash
    			/bin/chroot
    			/bin/csh
    			/bin/sync
    			/bin/tcsh
    			/bin/zsh
    			/sbin/halt
    			/sbin/nologin
    			/sbin/shutdown
    			[root@localhost ~]# cut -d: -f7 /etc/passwd | sort | uniq -u
    			/bin/chroot
    			/bin/csh
    			/bin/sync
    			/bin/tcsh
    			/bin/zsh
    			/sbin/halt
    			/sbin/shutdown
    			[root@localhost ~]# cut -d: -f7 /etc/passwd | sort | uniq -c
    			     21 /bin/bash
    			      1 /bin/chroot
    			      1 /bin/csh
    			      1 /bin/sync
    			      1 /bin/tcsh
    			      1 /bin/zsh
    			      1 /sbin/halt
    			     41 /sbin/nologin
    			      1 /sbin/shutdown
    			[root@localhost ~]# cut -d: -f7 /etc/passwd | sort | uniq -d
    			/bin/bash
    			/sbin/nologin
### （5）、diff

   diff - compare files line by line,逐行比较文件的不同之处

    命令： diff [OPTION]... FILES
    
          diff /PATH/TO/OLDFILE /PATH/TO/NEWFIEL > /PATH/TO/PATCH_FILE
               -U: 使用unified机制，即显示要修改的行的上下文，默认为3行
     patch:向文件打补丁
          patch [OPTIONS] -i /PATH/TO/PATCH_FILE /PATH/TO/OLDFILE
    
          patch /PATH/TO/OLDFILE < /PATH/TO/PATCH_FILE             以输入重定向实现
    
    PS:diff可以将2个文件夹下的文件逐一比较，可以以逐一生成补丁文件，逐一打补丁，变得和新文件一模一样
    example.
    
    			[root@localhost tmp]# diff fstab fstab.new 
    			2c2												//原文件的第二行change为新文件的第二行
    			< #             								//原文件#删除
    			---             								//分割线
    			> ##											//新文件的第二行为## 
    			8c8                                 			//原文件的第8行change为新文件的第8行
    			< #                                 			//原文件#删除
    			---                                 			//分割线
    			> #jacky                            			//线文件的第8为#jacky
    			[root@localhost tmp]# diff fstab fstab.new > fstab.patch          //比较新旧文件，重定向到文件fstab.patch，达成生成补丁的目的
    			[root@localhost tmp]# cat fstab.patch 						//查看补丁文件
    			2c2
    			< #
    			---
    			> ##
    			8c8
    			< #
    			---
    			> #jacky
    			[root@localhost tmp]# patch -i fstab.patch fstab        //给fstab文件打补丁
    			patching file fstab
    			[root@localhost tmp]# diff fstab fstab.new             //再次比较，因为fstab文件打了补丁和fstab.new文件内容一样了，比较没有不同，不会有结果输出
    			[root@localhost tmp]# 
    			[root@localhost tmp]# patch -R -i fstab.patch fstab     //逆向补丁（取消补丁的效果）
    			patching file fstab
    			[root@localhost tmp]# diff -u fstab fstab.new          //加-u选项统一上下文的输出行数（默认为3行），这是另一种风格的比较结果输出方式
    			--- fstab	2019-05-08 22:03:29.007621543 +0800
    			+++ fstab.new	2019-05-08 21:30:43.782458462 +0800
    			@@ -1,11 +1,11 @@
    			 
    			-#
    			+##
    			 # /etc/fstab
    			 # Created by anaconda on Tue Mar 19 19:01:41 2019
    			 #
    			 # Accessible filesystems, by reference, are maintained under '/dev/disk'
    			 # See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
    			-#
    			+#jacky
    			 /dev/mapper/centos-root /                       xfs     defaults        0 0
    			 UUID=ded54529-49da-4e0f-b847-efeb43908dad /boot                   xfs     defaults        0 0
    			 /dev/mapper/centos-home /home                   xfs     defaults        0 0