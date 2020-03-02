## Linu系统上的特殊权限SUID,SGID,STICKY和facl

***

### 安全上下文

1、进程以某用户的身份运行；进程是发起此进程用户的代理，因此以此用户的身份和权限完成所有操作；

2、权限匹配模型：
	

（1）判断进程的属主（发起者），是否为被访问文件的属主，如果是，则应用属主权限；否则进入第2步；
	

（2）判断进程的属主（发起者），是否属于被访问的属组；如果是，则应用属组的权限；否则进入第3步；
	

（3）应用other的权限；

### SUID

 默认情况下：用户发起的进程，进程的属主是其发起者；因此，其以发起者的身份运行；

 SUID的功用：用户运行某程序时，如果此程序拥有SUID权限，那此程序运行为进程时，进程的属主不是发起者，而是程序文件自己的属主；

```
管理文件的SUID权限：

  chmod u+|-s FILE...

展示位置：属主的执行权限位

如果属主原本有执行权限，显示为小写s;
否则，显示为大写S;
 				例如 rws,rwS

```

**examle.**

```
[root@localhost tmp]# cp /bin/cat /tmp
[root@localhost tmp]# chmod u+s ./cat
[root@localhost tmp]# ll
total 752
-rw-r--r--. 1 root root   2729 Feb 16 21:16 anaconda.log
-rwsr-xr-x. 1 root root  54080 Feb 17 18:54 cat
.
.
.
[root@localhost tmp]# su - jacky
Last login: Mon Feb 17 18:56:20 PST 2020 on pts/0
[jacky@localhost ~]$ whoami
jacky
[jacky@localhost ~]$ cat /etc/shadow
cat: /etc/shadow: Permission denied
[jacky@localhost ~]$ /tmp/cat /etc/shadow
root:$1$fHBNu9YM$AY78uh5gbldoPLdWP6cis.::0:99999:7:::
bin:*:17110:0:99999:7:::
daemon:*:17110:0:99999:7:::
adm:*:17110:0:99999:7:::
.
.
.
```

### SGID

功用：当目录属组有写权限，且有SGID权限时，那么所有属于此目录的属组，且以属组身份在此目录中新建文件或目录时，新文件的属组不是用户的基本组，而是此目录的属组；

```
管理文件的SGID权限：
 	chmod g+|-s FILE...

 	如果属组原本有执行权限，显示为小写s;
 	否则，显示为大写S;
 				  例如 rws,rwS

```

**example.**

```
root@bogon tmp]# mkdir /tmp/test
[root@bogon test]# ls -ld /tmp/test/
drwxr-xr-x. 2 root root 6 Feb 17 22:36 /tmp/test/
[root@bogon test]# chown :mygrp /tmp/test
[root@bogon test]# ls -ld /tmp/test/
drwxr-xr-x. 2 root mygrp 6 Feb 17 22:36 /tmp/test/
[root@bogon test]# chmod g+w /tmp/test
[root@bogon test]# ll -d /tmp/test
drwxrwxr-x. 2 root mygrp 6 Feb 17 22:36 /tmp/test
[root@bogon test]# usermod -aG mygrp fedora
[root@bogon test]# usermod -aG mygrp centos
[root@bogon test]# id fedora
uid=1002(fedora) gid=1003(fedora) groups=1003(fedora),1002(mygrp)
[root@bogon test]# id centos
uid=1003(centos) gid=1004(centos) groups=1004(centos),1002(mygrp)
[root@bogon test]# su - fedora
Last login: Mon Feb 17 22:48:30 PST 2020 on pts/1
[fedora@bogon ~]$ touch /tmp/test/a.fedora
[fedora@bogon ~]$ ll /tmp/test/
total 0
-rw-rw-r--. 1 fedora fedora 0 Feb 17 22:52 a.fedora
[root@bogon test]# su - centos
[centos@bogon ~]$ 
[centos@bogon ~]$ 
[centos@bogon ~]$ cd /tmp/test
[centos@bogon test]$ touch a.centos
[centos@bogon test]$ ll
total 0
-rw-rw-r--. 1 centos centos 0 Feb 17 22:52 a.centos
-rw-rw-r--. 1 fedora fedora 0 Feb 17 22:52 a.fedora
centos@bogon test]$ exit
logout
[root@bogon test]# 
[root@bogon test]# chmod g+s /tmp/test
[root@bogon test]# ls -ld /tmp/test
drwxrwsr-x. 2 root mygrp 38 Feb 17 23:02 /tmp/test
[root@bogon test]# su - fedora
Last login: Mon Feb 17 22:50:44 PST 2020 on pts/1
[fedora@bogon ~]$ touch /tmp/test/b.fedora
[fedora@bogon ~]$ ll /tmp/test/
total 0
-rw-rw-r--. 1 centos centos 0 Feb 17 22:52 a.centos
-rw-rw-r--. 1 fedora fedora 0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 fedora mygrp  0 Feb 17 23:04 b.fedora
[fedora@bogon ~]$ exit
logout
[root@bogon test]# su - centos
Last login: Mon Feb 17 22:52:21 PST 2020 on pts/1
[centos@bogon ~]$ touch /tmp/test/b.centos
[centos@bogon ~]$ ll /tmp/test
total 0
-rw-rw-r--. 1 centos centos 0 Feb 17 22:52 a.centos
-rw-rw-r--. 1 fedora fedora 0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 centos mygrp  0 Feb 17 23:06 b.centos
-rw-rw-r--. 1 fedora mygrp  0 Feb 17 23:04 b.fedora

```

### sticky

功用：对于属组或全局可写的目录，组内的所有用户或系统上的所有用户在此目录中都能创建新文件或删除所有的已有文件；如果为此类目录设置Sticky权限，则每个用户能创建新文件，但只能删除自己的文件；

```
管理文件的Sticky权限:
 		  chmod o+|-t FILE...

 		  展示位置：其他用户的执行权限位
 		  		如果其他用户原本有执行权限，显示为小写t;
 		  		   否则，显示为大写T;

 		系统上的/tmp和/var/tmp目录默认均有sticky权限；

```

**example.**

```
[root@bogon ~]# su - fedora
Last login: Mon Feb 17 23:04:29 PST 2020 on pts/1
[fedora@bogon ~]$ cd /tmp/test/
[fedora@bogon test]$ ll
total 4
-rw-rw-r--. 1 centos centos  0 Feb 17 22:52 a.centos
-rw-rw-r--. 1 fedora fedora  0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:06 b.centos
-rw-rw-r--. 1 fedora mygrp  21 Feb 17 23:40 b.fedora
-rw-rw-r--. 1 centos centos  0 Feb 17 23:41 c.centos
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:45 d.centos
[fedora@bogon test]$ rm b.centos 
[fedora@bogon test]$ ll
total 4
-rw-rw-r--. 1 centos centos  0 Feb 17 22:52 a.centos
-rw-rw-r--. 1 fedora fedora  0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 fedora mygrp  21 Feb 17 23:40 b.fedora
-rw-rw-r--. 1 centos centos  0 Feb 17 23:41 c.centos
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:45 d.centos
[fedora@bogon test]$ rm a.centos 
rm: remove write-protected regular empty file ‘a.centos’? y
[fedora@bogon test]$ ll
total 4
-rw-rw-r--. 1 fedora fedora  0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 fedora mygrp  21 Feb 17 23:40 b.fedora
-rw-rw-r--. 1 centos centos  0 Feb 17 23:41 c.centos
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:45 d.centos
[fedora@bogon test]$ exit
logout
[root@bogon ~]# chmod o+t /tmp/test
[root@bogon ~]# ls -ld /tmp/test/
drwxrwsr-t. 2 root mygrp 70 Feb 17 23:50 /tmp/test/
[root@bogon ~]# su - fedora 
Last login: Mon Feb 17 23:46:20 PST 2020 on pts/1
[fedora@bogon ~]$ cd /tmp/test/
[fedora@bogon test]$ ll
total 4
-rw-rw-r--. 1 fedora fedora  0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 fedora mygrp  21 Feb 17 23:40 b.fedora
-rw-rw-r--. 1 centos centos  0 Feb 17 23:41 c.centos
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:45 d.centos
[fedora@bogon test]$ rm c.centos 
rm: remove write-protected regular empty file ‘c.centos’? y
rm: cannot remove ‘c.centos’: Operation not permitted
[fedora@bogon test]$ rm d.centos 
rm: cannot remove ‘d.centos’: Operation not permitted

```

### 管理特殊权限的另一种方式：

```
		suid sgid sticky  		八进制权限

					0	0	0		0
					0	0	1		1			sticky
					0	1	0		2			sgid
					0	1	1		3			sgid+sticky	
					1	0	0		4			suid
					1	0	1		5			suid+sticky
					1	1	0		6			suid+suid
					1	1	1		7			suid+sgid+sticky	

		基于八进制方式赋权是，可于默认的三位八进制数字左侧再加一位八进制数字；

		例如：chomd 1777 最左侧的1是特殊权限位，此处表示拥有sticky权限 
```

### facl

**file access control lists  文件访问控制列表**

文件的额外赋权机制：

在原有的u,g,o之外，另一层让普通用户能控制赋权给另外指定的用户或组的赋权机制；

```
	   getfacl命令

	   		getfacl FILE...
	   			user:USERNAME:MODE
	   			group:GROUPNAME:MODE

	   setfacl命令
	   		赋权给用户:
	        	setfacl -m u:USERNAME:MODE FILE...
	        赋权给组:
	        	setfacl -m g:GROUPNAME:MODE FILE...

	        撤销赋权:
	        	setfacl -x u:USERNAME FILE...
	        	setfacl -x g:GROUPNAME FILE...


	   当用户发起的进程访问某一问件时，首先会检查进程的属主是不是被访问文件的属主相同，如果相同，则应用属主权限，否则将检查此文件的facl中有没有包含该进程发起者对应的权限定义，如果有，则应用facl中的权限；如果没有，检查进程的属组是不是文件的属组，如果是应用属组权限，否则，再检查此用户（进程发起者）的属组有没有在文件的facl中定义相关权限，如果有则应用facl中的权限，否则应用其他用户的权限。 
```

**example.**

```
[fedora@bogon tmp]$ cd /tmp/test/
[fedora@bogon test]$ touch touch.fedora
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
group::rw-
other::r--

[fedora@bogon test]$ setfacl -m u:centos:rw touch.fedora 
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
user:centos:rw-
group::rw-
mask::rw-
other::r--

[fedora@bogon test]$ setfacl -m u:jacky:rw touch.fedora 
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
user:jacky:rw-
user:centos:rw-
group::rw-
mask::rw-
other::r--

[fedora@bogon test]$ setfacl -m g:mygrp:rw touch.fedora 
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
user:jacky:rw-
user:centos:rw-
group::rw-
group:mygrp:rw-
mask::rw-
other::r--

[fedora@bogon test]$ setfacl -x u:centos touch.fedora 
[fedora@bogon test]$ setfacl -x g:mygrp touch.fedora 
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
user:jacky:rw-
group::rw-
mask::rw-
other::r--
```

**example2.在facl中单独禁止某用户对文件的权限**

```
[arvin@bogon test]$ ll
total 4
-rw-rw-r--. 1 fedora fedora  0 Feb 17 22:52 a.fedora
-rw-rw-r--. 1 fedora mygrp  21 Feb 17 23:40 b.fedora
-rw-rw-r--. 1 centos centos  0 Feb 17 23:41 c.centos
-rw-rw-r--. 1 centos mygrp   0 Feb 17 23:45 d.centos
-rw-rw-r--+ 1 fedora mygrp   0 Feb 18 00:38 touch.fedora
[arvin@bogon test]$ cat /tmp/test/touch.fedora 
hello world

[fedora@bogon test]$ setfacl -m u:arvin:--- touch.fedora 
[fedora@bogon test]$ getfacl touch.fedora 
# file: touch.fedora
# owner: fedora
# group: mygrp
user::rw-
user:jacky:rw-
user:arvin:---
group::rw-
mask::rw-
other::r--

[fedora@bogon test]$ 

[arvin@bogon test]$ cat /tmp/test/touch.fedora 
cat: /tmp/test/touch.fedora: Permission denied
```

