

**1、定义一个对所有用户都生效的命令别名**

定义一个对所有用户都生效的命令别名需要更改全局配置文件/etc/bashrc，

例如，我们以root用户编辑/etc/bashrc,在文件的最后一行增加alias like='ls'

当我们新启一个shell进程的时候，列出命令别名，会发现刚定义的别名like

```
[root@localhost ~]# tail /etc/bashrc
                . "$i" >/dev/null
            fi
        fi
    done

    unset i
    unset -f pathmunge
fi
# vim:ts=4:sw=4
alias like='ls'                        //此处显示了我追加在文件最后的内容，定义别名like
[root@localhost ~]# alias
alias cls='clear'
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias like='ls'                        //别名like
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
[root@localhost ~]# like -l
总用量 12
drwxr-xr-x. 2 root  root      6 4月   3 00:02 00-02-17
-rw-r-----. 1 root  root   1257 3月  19 19:05 anaconda-ks.cfg
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Desktop
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Documents
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Downloads
-rw-r--r--. 1 root  root    132 4月  30 16:44 grep.txt
-rwxr-xr-x. 1 jacky docker  511 4月  19 20:44 inittab
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Music
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Pictures
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Public
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Templates
drwxr-xr-x. 2 root  root      6 3月  19 14:47 Videos
drwxr-xr-x. 2 root  root      6 3月  19 14:47 桌面
```

**2、显示/etc/passwd文件中不以/bin/bash结尾的行。**

`~]# grep -v "/bin/bash$" /etc/passwd`



**3、找出/etc/passwd文件中，包含二位数子或三位数的行。**

    grep:    ~]# grep "\<[0-9]\{2,3\}\>" /etc/passwd
    
    egrep:   ~]# egrep "\<[0-9]{2,3}\>" /etc/passwd


**4、显示/proc/meminfo文件中以大写或小写s开头的行;至少以三种方式实现**

     A: ~]# grep "^[sS]" /proc/meminfo
     B: ~]# egrep "^(s|S)" /proc/meminfo
     C: ~]# grep "^s" /tmp/super.txt && grep "^S" /tmp/super.txt
     D: ~]# grep -i "^s" /proc/meminfo


**5、使用echo输出一个绝对路径，使用egrep取出路径名，类型执行dirname /etc/passwd的结果**

```
grep实现
[root@localhost ~]# echo /etc/rc.d/init.d/functions | grep -o "^/.*/"
/etc/rc.d/init.d/

egrep实现
[root@localhost ~]# echo /etc/rc.d/init.d/functions | grep -E -o "^/(.*)/"
/etc/rc.d/init.d/

```

**6、找出ifconfig中的Ip地址。要求结果只显示IP地址。**

      A: 解析：因为IP地址的区间为1.0.0.1 - 223.255.255.254，所以按照1-223.0-255.0-255.1-254去匹配
    	[root@localhost ~]# ifconfig | egrep -o "\<([1-9]|[1-9][0-9]|1[0-9]{2}|21[0-9]|22[0-3])\>\.(\<([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\>\.){2}\<([1-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-4])\>"
    	192.168.220.7
    	127.0.0.1
    	192.168.122.1
      B:取出网卡ens33的ip地址，先过滤到Ip的行再用cut命令，以空格为分隔符取第10列，刚好就是IP地址		 
    	 [root@localhost tmp]# ifconfig ens33 | grep "inet.*broad" | cut -d' ' -f10
    	 192.168.220.7    





**7、vim定制自动缩进四个字符。**

末行模式下

`:set tabstop=4`





**8、编写脚本，实现自动添加三个用户，并计算这三个用户的uid之和。**


	[root@localhost tmp]# cat test.sh 
	#!/bin/bash
	
	id neo || useradd neo
	id jerry || useradd jerry
	id louis || useradd louis
	UID1=`id -u neo`
	UID2=`id -u jerry`
	UID3=`id -u louis`
	echo "$[$UID1+$UID2+$UID3]"
	[root@localhost tmp]# . test.sh 
	id: neo: no such user
	id: jerry: no such user
	id: louis: no such user
	15033







**9、find用法以及常用用法的示例演示。**



**find:**
 实时查找工具，通过遍历指定其实路径下文件系统层级结构完成文件查找；

​	工作特性：
 	

​		查找速度略慢；
 				

​		精确查找；
 				

​		实时查找；

**用法：**

​	**find [OPTIONS] [查找起始路径] [查找条件] [处理动作]**		

​												处理动作可加ls 表明查找后结果按ls-l排列

**查找起始路径:** 指定具体搜索目标起始路径;默认为当前目录；

**查找条件:** 指定的查找标准，可以根据文件名，大小、类型、从属关系、权限等等标准进行；默认为找出指定路径下的所有文件；

**处理动作:** 对符合查找天剑的文件作出的操作，例如删除等操作；默认为输出至标准输出；

```


 查找条件：
 		表达式：选项和测试

 		测试：结果通常为布尔型（"true"or"false"）

 		根据文件名查找：
 			   -name "pattern"
 			   -iname "pattern"
 			    		(支持glob风格的通配符)；
 			    		*,?,[],[^]

 			   -regex pattern:基于正则表达式模式查找文件，匹配是整个路径，而非基名。

 		根据文件从属关系查找：
 			    -user USERNAME: 查找属主指定用户的所有文件；
 			    -group GRPNAME: 查找属组指定组的所有文件；

 			    -uid UID:查找属主指定UID的所有文件;
 			    -gid GID:查找属组指定GID的所有文件;

 			    -nouser: 查找没有属主的文件;
 			    -nogroup: 查找没有属组的文件；

 		根据文件的类型查找：
 			    -type TYPE:
 			    		f:普通文件
 			    		d:目录文件
 			    		l:符号链接文件
 			    		b:块设备文件
 			    		c:字符设备文件
 			    		p:管道文件
 			    		s:套接字文件

 	    组合测试：
 			   与：-a,默认组合逻辑；必须符合全部条件
 			   或：-o,符合其中之一条件即可
 			   非：-not,!
 			   
 			   !A -a !B = !(A -o B)
			   !A -o !B = !(A -a B)
```

example.

```
example1
[root@localhost tmp]# find /tmp -nouser -type f -ls
67654200    4 -rw-r--r--   1 5010     5010          541 2月 13 17:48 /tmp/fstab
[root@localhost tmp]# find /tmp -nouser -a -type f -ls
67654200    4 -rw-r--r--   1 5010     5010          541 2月 13 17:48 /tmp/fstab
examle2
1、找出/tmp目录下属主为非root的所有文件；
						
	]# find /tmp -not -user root -ls

2、找出/tmp目录下文件名中不包含fstab字符串的文件；
	]# find /tmp -not -iname "*fatab*"

3、找出/tmp目录下属主为非root,而且文件名不包含fstab字符串的文件；
	]# find /tmp -not -user root -a -not -iname "*fstab*" -ls

或者
	]# find /tmp -not \( -user root -o -iname "*fstab*" \) -ls
```

**根据文件大小查找**

```
-size [+|-]#UNIT
	      常用单位: k,M,G
	       #UNIT:(#-1,#]此处表示值大于-1的值 小于等于#值
	       -#UNIT:[0,#-1]
	       +#UNIT:(#,oo)

```

**根据时间戳查找:**

```
以“天”为单位：
	-atime [+|-]#										/访问时间
			#:[#,#-1)
			-#:(#,0]
			+#:(oo,#-1]	 此处的#在后方表达式中是一个负数 


	-mtime								/修改时间
    -ctime								/改动时间					

以“分钟”为单位：
	-amin
	-mmin
	-cmin
```

example.

```
找到/etc下 访问时间在一周之前的文件
#] find /etc -atime +7 -ls


24小时内/etc下修改过的文件
#] find /etc -mtime -1 -ls
```

**根据权限查找**

```
-perm [/|-] mode
		mode: 精确权限匹配
		/mode: 任何一类用户（u,g,o）的权限中的任何一位(r,d,x)符合条件即满足；
				9位权限之间存在“或”关系

		-mode: 每一类用户(u,g,o)的权限中的每一位(r,w,x)同时符合条件即满足；
				9位权限之间存在“与”关系

```

example.

```
[root@localhost test]# touch a b c d e f g
[root@localhost test]# 
[root@localhost test]# chmod 640 a 
[root@localhost test]# chmod 666 b
[root@localhost test]# chmod 440 c
[root@localhost test]# chmod 775 d
[root@localhost test]# chmod 777 e
[root@localhost test]# ll
总用量 0
-rw-r-----. 1 root root 0 2月  15 15:50 a
-rw-rw-rw-. 1 root root 0 2月  15 15:50 b
-r--r-----. 1 root root 0 2月  15 15:50 c
-rwxrwxr-x. 1 root root 0 2月  15 15:50 d
-rwxrwxrwx. 1 root root 0 2月  15 15:50 e
-rw-r--r--. 1 root root 0 2月  15 15:50 f
-rw-r--r--. 1 root root 0 2月  15 15:50 g
[root@localhost test]# find ./ -perm 644 -ls
372669    0 -rw-r--r--   1 root     root            0 2月 15 15:50 ./f
372670    0 -rw-r--r--   1 root     root            0 2月 15 15:50 ./g

[root@localhost test]# find ./ -perm /222 -ls
372648    0 drwxr-xr-x   2 root     root           69 2月 15 15:50 ./
372650    0 -rw-r-----   1 root     root            0 2月 15 15:50 ./a
372665    0 -rw-rw-rw-   1 root     root            0 2月 15 15:50 ./b
372667    0 -rwxrwxr-x   1 root     root            0 2月 15 15:50 ./d
372668    0 -rwxrwxrwx   1 root     root            0 2月 15 15:50 ./e
372669    0 -rw-r--r--   1 root     root            0 2月 15 15:50 ./f
372670    0 -rw-r--r--   1 root     root            0 2月 15 15:50 ./g
[root@localhost test]# find ./ -perm /111 -ls
372648    0 drwxr-xr-x   2 root     root           69 2月 15 15:50 ./
372667    0 -rwxrwxr-x   1 root     root            0 2月 15 15:50 ./d
372668    0 -rwxrwxrwx   1 root     root            0 2月 15 15:50 ./e
[root@localhost test]# find ./ -perm /001 -ls
372648    0 drwxr-xr-x   2 root     root           69 2月 15 15:50 ./
372667    0 -rwxrwxr-x   1 root     root            0 2月 15 15:50 ./d
372668    0 -rwxrwxrwx   1 root     root            0 2月 15 15:50 ./e
[root@localhost test]# find ./ -perm /002 -ls
372665    0 -rw-rw-rw-   1 root     root            0 2月 15 15:50 ./b
```

**处理动作**

```
	-print: 输出至标准输出;默认的动作；
	-ls: 类似于对查找到的文件执行“ls-l”命令，输出文件的详细信息；
	-delete: 删除查找到的文件；
	-fls /PATH/TO/SOMEFILE: 把查找到的所有文件的长格式信息保存至指定文件中（类似于把ls的输出结果保存到指定文件中);
	-ok COMMAND {} \; :对查找到的每个文件执行有COMMAND表示的命令；每次操作都由用户进行确认
	-exec COMMAND {} \;查找到的每个文件执行有COMMAND表示的命令；不用用户确认直接执行命令

	      example.
				[root@localhost tmp]# find ./ -nouser
				./fstab
				[root@localhost tmp]# find ./ -nouser -a -nogroup
				./fstab
				[root@localhost tmp]# find ./ -nouser -a -nogroup -ls
				67654200    4 -rw-r--r--   1 5010     5010          541 2月 13 17:48 ./fstab
				[root@localhost tmp]# find ./ -nouser -a -nogroup -ok chown root:root {} \;
				< chown ... ./fstab > ? y
				[root@localhost tmp]# ll
				总用量 12
				-rw-r--r--. 1 root  root  451 2月  13 17:48 crontab
				-rw-r--r--. 1 root  root  541 2月  13 17:48 fstab

				[root@localhost test]# find ./ -perm /002
				./b
				./e
				[root@localhost test]# find ./ -perm /002 -exec mv {} {}.danger \;
				[root@localhost test]# ll
				总用量 4
				-rw-r-----. 1 root root   0 2月  15 15:50 a
				-rw-r--r--. 1 root root 144 2月  15 16:13 abc.test
				-rw-rw-rw-. 1 root root   0 2月  15 15:50 b.danger
				-r--r-----. 1 root root   0 2月  15 15:50 c
				-rwxrwxr-x. 1 root root   0 2月  15 15:50 d
				-rwxrwxrwx. 1 root root   0 2月  15 15:50 e.danger
				-rw-r--r--. 1 root root   0 2月  15 15:50 f
				-rw-r--r--. 1 root root   0 2月  15 15:50 g


	注意： find传递查找到的文件路径至后面的命令时，是先查找出所有符合条件的文件路径，并一次性传递给后面的命令；
		但是有些命令不能接受过长的参数，此时命令执行会失败；另一种方式可规避此问题；

			find | xargs COMMAND
```

**参数代换命令：xargs**

**xargs [option] COMMAND**

```
   -o:如果输入的stdin含特殊字符，例如`,\,空格字符等，这个参数可以将它还原成一般字符，这个参数可以用于特殊状态。

   -e:这个是EOF(end of file)的意思，后面接一个字符串char，当xargs分析到这个字符串是，就会停止继续工作。
   -p: 在执行每个命令的参数时，都会询问用户的意思。
   -n:后面接次数，每次command命令执行时，要使用几个参数的意思。

   当xargs后面没有接任何命令时，默认是以echo来进行输出。
```

example

```
1、查找/var目录下属主为root,且属组为mail的所有文件或目录；

[root@localhost ~]# find /var -user root -group mail -ls
67161923    0 drwxrwxr-x   2 root     mail          219 2月 14 14:43 /var/spool/mail


2、查找/usr目录下不属于root，bin或hadoop的所有文件或目录；用两种方法；

A.
[root@localhost ~]# find /usr -not -user root -a -not -user bin -a -not -user hadoop -ls
 96954    0 drwx------   2 polkitd  root          287 3月 19  2019 /usr/share/polkit-1/rules.d
1000398   16 -rwsr-sr-x   1 abrt     abrt        15432 11月 14  2018 /usr/libexec/abrt-action-install-debuginfo-to-abrt-cache
B.
[root@localhost ~]# find /usr -not \( -user root -o -user bin -o -user hadoop \) -ls
 96954    0 drwx------   2 polkitd  root          287 3月 19  2019 /usr/share/polkit-1/rules.d
1000398   16 -rwsr-sr-x   1 abrt     abrt        15432 11月 14  2018 /usr/libexec/abrt-action-install-debuginfo-to-abrt-cache



3、查找/etc目录下最近一周内其内容修改过，且属主不是root用户也不是hadoop用户的文件或目录；

[root@localhost ~]# find /etc -mtime -7 -a -not \( -user root -o -user hadoop \) -ls





4、查找当前系统上没有属主或属组，且最近一周内曾被访问过的文件或目录；

[root@localhost ~]# find / \( -nouser -o -nogroup \) -atime -7 -ls
   105    0 drwx------   3 5010     5010           78 2月 14 14:43 /home/neo
67108939    0 drwxr-xr-x   4 5010     5010           39 3月 19  2019 /home/neo/.mozilla
134284490    0 drwxr-xr-x   2 5010     5010            6 6月 10  2014 /home/neo/.mozilla/extensions
201326666    0 drwxr-xr-x   2 5010     5010            6 6月 10  2014 /home/neo/.mozilla/plugins
find: ‘/proc/17459/task/17459/fd/5’: 没有那个文件或目录
find: ‘/proc/17459/task/17459/fdinfo/5’: 没有那个文件或目录
find: ‘/proc/17459/fd/6’: 没有那个文件或目录
find: ‘/proc/17459/fdinfo/6’: 没有那个文件或目录
67654182    0 -rw-rw----   1 5010     mail            0 2月 14 14:43 /var/spool/mail/neo




5、查找/etc目录下大于1M且类型为普通文件的所有文件；

[root@localhost ~]#  find /etc -size +1M -type f -exec ls -lh {} \;
-r--r--r--. 1 root root 7.2M 3月  19 2019 /etc/udev/hwdb.bin
-rw-r--r--. 1 root root 1.4M 3月  19 2019 /etc/selinux/targeted/contexts/files/file_contexts.bin
-rw-r--r--. 1 root root 3.7M 3月  19 2019 /etc/selinux/targeted/policy/policy.31
-rw-------. 1 root root 3.7M 3月  19 2019 /etc/selinux/targeted/active/policy.kern
-rw-------. 1 root root 3.7M 3月  19 2019 /etc/selinux/targeted/active/policy.linked
-rw-r--r--. 1 root root 1.4M 4月  11 2018 /etc/brltty/zh-tw.ctb




6、查找/etc目录下所有用户都没有写权限的文件；

[root@localhost ~]# find /etc -not -perm /222 -ls
33615593  180 -r--r--r--   1 root     root       183421 3月 19  2019 /etc/pki/ca-trust/extracted/java/cacerts
67181253  328 -r--r--r--   1 root     root       334001 3月 19  2019 /etc/pki/ca-trust/extracted/openssl/ca-bundle.trust.crt
100708941  248 -r--r--r--   1 root     root       251593 3月 19  2019 /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
100708942  200 -r--r--r--   1 root     root       201168 3月 19  2019 /etc/pki/ca-trust/extracted/pem/email-ca-bundle.pem
100708943  168 -r--r--r--   1 root     root       171863 3月 19  2019 /etc/pki/ca-trust/extracted/pem/objsign-ca-bundle.pem
235701    4 -r--r--r--   1 root     root          531 5月  3  2017 /etc/lvm/profile/cache-mq.profile
235702    4 -r--r--r--   1 root     root          339 5月  3  2017 /etc/lvm/profile/cache-smq.profile
235703    4 -r--r--r--   1 root     root         3020 8月  5  2017 /etc/lvm/profile/command_profile_template.profile
235704    4 -r--r--r--   1 root     root         2309 5月  3  2017 /etc/lvm/profile/lvmdbusd.profile
235705    4 -r--r--r--   1 root     root          828 8月  5  2017 /etc/lvm/profile/metadata_profile_template.profile
235706    4 -r--r--r--   1 root     root           76 5月  3  2017 /etc/lvm/profile/thin-generic.profile
235707    4 -r--r--r--   1 root     root           80 5月  3  2017 /etc/lvm/profile/thin-performance.profile
100769850    4 -r--------   1 root     root           45 3月 19  2019 /etc/openldap/certs/password
67183454    4 ----------   1 root     root          975 2月 16 11:25 /etc/gshadow
34025845    4 -r--r--r--   1 root     root          460 4月 11  2018 /etc/dbus-1/system.d/cups.conf
67183449    4 ----------   1 root     root         1608 2月 16 11:25 /etc/shadow
67294134    4 ----------   1 root     root         1579 2月 15 12:18 /etc/shadow-
100861801    4 -r--r--r--   1 root     root           63 8月 23  2017 /etc/ld.so.conf.d/kernel-3.10.0-693.el7.x86_64.conf
100821074    4 -r--r--r--   1 root     root           63 3月 18  2019 /etc/ld.so.conf.d/kernel-3.10.0-957.10.1.el7.x86_64.conf
102850752 7348 -r--r--r--   1 root     root      7522390 3月 19  2019 /etc/udev/hwdb.bin
67294386    4 -r--r--r--   1 root     root           33 3月 19  2019 /etc/machine-id
790438    4 -r--r--r--   1 root     root          146 4月 11  2018 /etc/pam.d/cups
67191422    4 ----------   1 root     root          964 2月 15 12:18 /etc/gshadow-
67513217    4 -r--r-----   1 root     root         3938 6月  7  2017 /etc/sudoers




7、查找/etc目录至少一类用户没有执行权限的文件；

[root@localhost ~]# find /etc -not -perm -111 -ls



8、查找/etc/init.d/目录下，所有用户都有执行权限，且其他用户有些权限的所有文件；

[root@localhost ~]# find /etc/init.d/ -perm -113 -ls
```

