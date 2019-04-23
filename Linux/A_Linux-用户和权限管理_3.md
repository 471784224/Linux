Linux-用户和权限管理

***

#### 一、用户和组

早期的计算机的使用场景，一般都是多用户，多任务下（ Multi-task,Multi-Users）。为了区别不同的使用者，隔离每个人对计算机资源的访问，引入了用户的概念。

不同的使用者通过用户标识和密码加以区别，

 认证：Authentication

 授权：Authorization

 审计：Audition(Account)

这就是3A认证

Linux下的用户标识就是：用户

组，即用户组，用户容器。

##### 1、用户类别

用户分为管理用户和普通用户，其中普通用户又包括系统用户和登录用户

系统用户：为了能够让后台进程或服务类进程以非管理员身份运行，通常需要为此创建多个普通用户，这类用户从不登录系统

          1、管理用户
          2、普通用户
              a.系统用户
              b.登录用户
###### 2、用户标识

      用户标识：UserID,UID
            16bits二进制数字：0+65535
                  管理员：0
                  普通用户：1-65535
                        系统用户：1-499（CentOS6）,1-999(CentOS7)
                        登录用户：500-60000（CentOS6）,10000-60000(CentOS7)
    
            名称解析：名称转换
                Username <--> UID 
    
                根据名称解析库进行: /etc/passwd

###### 3、组类别

和用户相对应的，组分为管理员组和普通用户组，普通用户组又包括系统组和登录组

          1、管理员组
          2、普通用户组
              a、系统组
              b、登录组
##### 4、组标识

       组标识：GroupID,GID
                  管理员组：0
                  普通用户组：1-65535
                        系统用户组：1-499（CentOS6）,1-999(CentOS7)
                        登录用户组：500-60000（CentOS6）,10000-60000(CentOS7)   
    
            名称解析：groupname <--> GID
                解析库： /etc/group
##### 5、组的另外两种分类方法

1. A、用户的基本组:primary group,也叫主组

   B、用户的附加组

2. A、私有组：组名同用户名且包含一个用户：privite group

   B、公共组：组内包含了多个用户：public group

##### 6、计算机是如何认证用户所提供的信息是否真实的？

认证信息：通过比对事先存储的，与登录时提供的信息是否一致；

password的存储位置：

                /etc/shadow     用户密码
                /etc/gshadow    组密码
##### 7、密码

           密码的使用策略
               1：使用随机密码：
               2：最短长度不要低于8位
               3：应该使用大写字母，小写字母、数字和标点符号四类字符中至少三类
               4：定期更换
          
          密码存储格式：单向加密，并借助于salt完成
           加密算法：
               1、对称加密：加密和解密使用同一个秘钥
               2、非对称加密：加密和解密使用的一对儿秘钥；
                    秘钥对儿：  
                      公钥:public key
                      私钥:privita key
               3、单向加密：只能加密，不能解密;提取数据特征码；
                    定长输出
                    雪崩效应
                    
                    单向加密常用算法：
                         md5: message digest, 128bits
                         sha: secure hash algorithm,160bits 安全的哈希算法
                         sha224
                         sha256
                         sha384
                         sha512
    
                  在计算之时加salt，添加的随机数

##### 8、/etc/passwd,/etc/shadow,etc/group,/etc/gshadow 文件内容


              /etc/passwd:    用户信息库
                     name:password:UID:GID:GECOS:directory:shell
    
                     name: 用户名
                     password: 可以是加密的密码，也可是占位符
                     UID：
                     GID： 用户所属的主组的ID号；
                     GECOS： 用户的注释信息，可选
                     directory: 用户的家目录；
                     shell: 用户的默认shell,登录时默认shell程序
    
              /etc/shadow: 用户密码
                    用户名:加密的密码:最近一次修改密码的时间:密码最短使用期限:最长使用期限:警告期段:过期期限:保留字段
    
                    查看帮助文档：man 5 shadow
              
              etc/group:组的信息库
                   group_name:password:GID:user_list
    
                           user_list:该组的用户成员；以此组为附加组的用户的用户列表：
              etc/gshadow
                 group_name:password:admin_list:user_list
                           组名:密码:管理员列表:组成员列表
                      
                    查看帮助文档：man 5 gshadow      



#### 二、用户和组管理

##### 1、组管理命令

###### （1）groupadd命令： 添加组

    groupadd [选项] group_name
            
            -g GID: 指定GID；如不指定会默认，上一个组的GID+1
            -r:创建系统组；                
###### （2）groupmod命令：修改组属性

```
groupmod [选项] GROUP
    
        -g GID, 修改GID
        -n new_name: 修改组名
```

###### （3）groupdel命令： 删除组

​       `groupdel [选项] GROUP`

###### （4）gpasswd命令：修改组密码

组密码文件： /etc/gshadow

    gpasswd [选项] group
    
              （1）gpasswd group: 修改指定组的密码，默认情况下组创建之后没有密码，设定密码主要用途在用newgrp命令临时切换用户到某个组作为自己的基本组时使用，如果用户实现不属于该组，切该组没有设定密码，用户将无法切换到此没有设定密码的组
                -a USERNAME:向组中添加用户
                -d USERNAME:从组中移除用户
                -A：指定组管理员；
                -M：指定组成员和-A的用途差不多；
                -r：删除密码；
                -R：限制用户登入组，只有组中的成员才可以用newgrp加入该组。
###### （5）newgrp命令：临时切换指定的组为基本组

    newgrp [-] [group]
             -：会模拟用户重新登录以实现重新初始化其工作环境
##### 2、用户管理命令

###### （1）useradd命令：添加用户

          useradd [选项] 登录名
              -u, --uid UID: 指定UID；如不指定默认为上一个UID+1
              -g, --gid GROUP：指定用户基本组ID或组名，此组得事先存在
    		  -G, --groups GROUP1[,GROUP2,...[,GROUPN]]]: 指定用户所属的附加组（可以是组ID也可以是组名），多个组之间用逗号分隔；
    		  -c, --comment COMMENT：指明注释信息
    			           任何字符串。通常是关于登录的简短描述，当前用于用户全名。
              -d, --home HOME_DIR:以指定的路径作为用户的家目录：通过复制/etc/skel此目录并重命名实现；指定的家目录路径如果事先存在，则不会为用户复制环境配置文件
              -s, --shell SHELL: 指定用户的默认shell,可用的所有shell列表存储在/etc/shells文件中；
              -r，创建一个系统用户
              -M, --no-create-home    不创建用户主目录，即使系统在 /etc/login.defs中的设置(CREATE_HOME)为yes
                    Do not create the user's home directory, even if the system wide setting from /etc/login.defs (CREATE_HOME) is set to yes.
              -f, --inactive INACTIVE
                       密码过期到账户被禁用之前的天数。0表示立即禁用，-1表示禁用这个功能
                                      如果未指定，useradd将使用在 /etc/default/useradd 中 INACTIVE 变量指定的默认禁用周期，或者默认为-1.
    
    		注意：创建用户是的诸多默认设定配置文件为/etc/login.defs
    		      -d选项指定家目录时，家目录的父目录需要事先存在，例如指定家目录为/users/username   父目录/users需事先存在（被创建）
    
    		useradd -D:显示创建用户的默认配置；
    		useradd -D 选项：修改默认选项的值；
    
    		        修改的结果保存于/etc/default/useradd文件中；也可以通过直接编辑此文件实现
###### （2）usermod命令：修改用户属性

         usermod [选项] 登录名
                 -u，--uid UID: 修改用户的ID为此处指定的新的UID;
                 -g, --gidGROUP:  修改用户所属的基本组；此组必须存在;
                 -G, --groupsGROUP1[,GROUP2,...[,GROUPN]]]: 修改用户所属的附加组；原来的附加组会被覆盖；
                 -a, --append: 与-G一同使用，用于为用户追加新的附加组;  -aG
                 -c, --comment COMMENT: 修改注释信息；
                 -d, --home HOME_DIR: 修改用户的家目录: 用户原有的文件不会被转移至新位置;
                 -m, --move-home： 只能与-d选项一同使用，用于将原来的家目录里面的所有文件移动到新的家目录，目前的家目录路径必须存在，否则新家目录不会被创建;
                 -l, --login NEW_LOGIN:修改用户名;
                 -s, --shell SHELL：修改用户的默认shell;
                 -L，--lock:锁定用户密码；即在用户原来的密码字符串之前添加一个"!";
                 -U，--unlock:解锁用户的密码;
###### （3）userdel命令：删除用户

         userdel [选项] 登录名
                 -r： 删除用户是一并删除其家目录
###### （4）passwd命令：修改用户密码

     passwd [-k] [-l] [-u [-f]] [-d] [-e] [-n mindays] [-x maxdays] [-w warndays] [-i inactivedays] [-S] [--stdin] [username]
    
              （1）passwd:修改用户自己的密码;
              （2）passwd USERNAME:修改指定用户的密码，但默认情况下仅root有此权限;
    
                 -l,-u： 锁定和解锁用户密码
                 -d：删除用户密码
                 -e DATE: 过期期限，日期
                 -i DAYS: 非活动期限;密码过期之后还能活动几天
                 -n DAYS: 密码的最短使用期限
                 -x DAYS: 密码的最长使用期限
                 -w DAYS: 警告期限
    
                 --stdin:
                     echo "PASSWORD" | passwd --stdin USERNAME
###### （5）chage命令：更改用户密码过期信息

             chage [选项] 登录名
    
                 -d
                 -E
                 -W
                 -M
                 -m
###### （6）id命令：显示用户的真是和有效ID

          id - print real and effective user and group IDs
               
               id [OPTION]... [USER]
                 -u: 仅显示有效的UID
                 -g: 仅显示用户的基本组ID
                 -G: 显示用户所属的所有组的ID，包括基本组和附加组
                 -n: 显示名字而非ID
###### （7）su命令：切换用户

    a.登录式切换：会通过读取目标用户的配置文件来重新初始化
    su - USERNAME
    su -l USERNAME
    
    b.非登陆式切换：不会读取目标用户的配置文件进行初始化
    su USERNAME
    
    注意：管理员（root）可无密码切换至其他任何用户；
    
          -c 'COMMAND': 仅以指定用户的身份运行此处指定的命令；运行一次之后直接退回
###### （8）其他命令

       chsh，  更改默认shell
       chfn,finger  更改finger信息，显示finger信息
       whoami
       pwck,检查用户密码
       grpck,检查组
#### 三、权限管理

##### 1.概念

**权限**
linux系统中，每个文件，每个目录都有一组权限！ 使用ls -l命令即可看到！

第一段内容，就是我们的权限位，这个权限位包含了十个字符！ 例如

-rw-r-----. 1 root root 1370 3月 1 12:17 /etc/passwd

-rwsr-xr-x. 1 root root 27832 6月 10 2014 /bin/passwd

**第一个字段**，第一个字符表示该文件的类型，-表示普通文件，d表示目录文件，c表示字符设备文件，b表示块设备文件，l表示链接文件，s表示套接字文件，p表示管道文件……

而后的九个字符，每三个字符表示一组权限，他们分别是属主，属组，和其他人的权限，而每一组中，r（第一位）表示读，（第二位）w表示写，x（第三位）表示执行，每个对应的位置上，如果有字符，则表示有该权限，-表示无权限！ 上面例子中，就表示，属主有rw权限，属组有r权限，其他人没有任何权限！

**第二个字段**，一个数字，表示该文件被链接的次数

**第三个字段和第四个字段** 表示该文件的属主和属组

**第五个字段** 表示该文件的大小，单位为字节！

**第六个字段** 表示该文件的最近更改时间（mtime）

**第七个字段** 文件完整路径


**权限的概念：rwx**

    r:readable，可读
    w:writeable,可写
    x:excuteable,可执行
**linux权限的表示结构**

                rwxrwxrwx:
                    左三位：定义user(owner)的权限
                    中三位：定义group的权限
                    右三位：定义other的权限
**进程安全上下文（Security Context）**


    进程对文件的访问权限应用模型:
                  
    前提：进程有属主和属组，文件有属主和属组
    1、任何一个可执行程序文件能不能运行，取决于发起者对程序文件是否有执行权限。
    2、启动为进程之后，其进程的属主为发起者，属组为进程发起者所属的属组。
    3、进程访问文件的权限，取决于进程的发起者：
    （1）进程的发起者同文件的属主，则应用文件属主权限
    （2）进程的发起者属于文件属组，则应用文件属组权限
    （3）应用其它权限。
**rwx对文件和目录的含义**

                 文件：
                     r:可获取文件的数据
                     w:可修改文件的数据
                     x:可将此文件发起运行为进程  
    
                 目录：
                     r:可使用ls命令获取其下的所有文件列表;(不能获取详细信息，即不包括ls -l)
                     w:可修改此目录下的文件列表：即创建或删除文件；
                     x:可cd至此目录中，且可使用ls -l来获取所有文件的详细属性信息；
**权限的两种表现形式**

1. mode(模型)： rwxrwxrwx
2. ownership(从属关系):user,group

**权限组合机制和八进制表示法**

权限的rwx由于在每个位置上只有2种形式，-和其他（r,w,x）即有或没有权限

因此rwx可以用3位二进制来表示，3位二进制又可以转换为8进制，组合关系如下

| 权限 | 二进制表示 | 8进制表示 | 注释           |
| :--- | ---------- | --------- | -------------- |
| ---  | 000        | 0         | **没有权限**   |
| --x  | 001        | 1         | **可执行**     |
| -w-  | 010        | 2         | **可写**       |
| -wx  | 011        | 3         | 可写可执行     |
| r--  | 100        | 4         | **可读**       |
| r-x  | 101        | 5         | 可读可执行     |
| rw-  | 110        | 6         | 可读可写       |
| rwx  | 111        | 7         | 可读可写可执行 |

用八位进制权限表示法来表示：如rxwrwxrwx可表示为777，表示属主属组和其他用户都具有读写执行的权限

##### 2. 权限管理命令

###### （1）chmod命令：修改文件权限

                   chmod - change file mode bits
    
    				chmod [OPTION]... MODE[,MODE]... FILE...
    				chmod [OPTION]... OCTAL-MODE FILE...
    				chmod [OPTION]... --reference=RFILE FILE...
                 
                 三类用户：
                      u:属主
                      g:属组
                      o:其他
                      a:所有
    
    			  chmod [OPTION]... MODE[,MODE]... FILE...
    
    			  (1)MODE表示法：
    
    			               A、赋权表示法：直接赋予用户对应权限，可以操作一类用户的所有权限位，
    			                  u=
    			                  g=
    			                  o=
    			                  a=
    			                  两种用户可以合并书写：ug=，uo=，go=,
    			                  权限可合并书写，r,w,x,rw,rwx,rx,wx,
    
    			               B、授权表示法：直接增加或删除一类用户的现有权限r,w,x;
    			                  u+,u-
    			                  g+,g-
    			                  o+,o-
    			                  a+,a-                 (有时候可省略a,直接写成+,-)
    			                  
    			                  两种用户可以合并书写：ug=，uo=，go=,
    			                  权限也可合并书写，r,w,x,rw,rwx,rx,wx,
    			                  尽量符合逻辑书写，不符合逻辑也不会报错（例如其他用户没有w权限用o-x也不会报错）
    			                  分别加不同权限时可以用逗号隔开，如：chmod u+w,g+r FILE
    			                  
                                  ps:全局+w操作只会匹配到u（属主用户）
    
    			  (2)八进制表示法
    			      chmod [OPTION]... OCTAL-MODE FILE...
    
    			      example:
    			            chmod 660 /etc/shadow
    
    			  (3)引用权限命令
    			       chmod [OPTION]... --reference=RFILE FILE...      
    			     修改文件FILE的权限与被引用的文件RFILE的权限相同
    
    			  选项 ：
    			      -R,--recurisive:递归修改，会将目录以及目录下的子目录和所有文件一起修改  （此选项只建议授权表示法使用，避免影响目录内部的文件拥有其他不需要的权限，例如错误的将x权限给不可执行的文件）
    
    		         注意：用户仅能修改属主为自己的那些文件的权限       
    	exmaple.				赋权表示
    							[root@bogon ~]# ls
    							00-02-17  a_c  a_d  anaconda-ks.cfg  b_c  b_d  Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos  桌面
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rw-------. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod g=rw anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rw-rw----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod ug=r anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-r--r-----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod a=rwx anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rwxrwxrwx. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod u=rwx,g=rw,o= anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rwxrw----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    
    							授权表示
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							----------. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod u+w anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							--w-------. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod ug+r anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rw-r-----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod a+x anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rwxr-x--x. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod -x anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rw-r-----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod u-w anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-r--r-----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg
    							[root@bogon ~]# chmod +w anaconda-ks.cfg 
    							[root@bogon ~]# ll anaconda-ks.cfg 
    							-rw-r-----. 1 root root 1257 3月  19 19:05 anaconda-ks.cfg            //全局+w操作只会匹配到u（属主用户），因为全局+w很危险
    
    							权限引用
    							[root@bogon tmp]# ll | head -3
    							总用量 276
    							-rw-r--r--. 1 root root      0 4月  19 19:16 a
    							-rw-r--r--. 1 root root      0 4月  19 19:16 b
    							[root@bogon tmp]# chmod 766 a
    							[root@bogon tmp]# ll | head -3
    							总用量 276
    							-rwxrw-rw-. 1 root root      0 4月  19 19:16 a
    							-rw-r--r--. 1 root root      0 4月  19 19:16 b
    							[root@bogon tmp]# chmod --reference=a b
    							[root@bogon tmp]# ll | head -3
    							总用量 276
    							-rwxrw-rw-. 1 root root      0 4月  19 19:16 a
    							-rwxrw-rw-. 1 root root      0 4月  19 19:16 b
    
    							-R选项	
    							root@bogon tmp]# ls -la skel
    							总用量 16
    							drwxr-xr-x.  3 root root   78 4月  19 19:43 .
    							drwxrwxrwt. 24 root root 4096 4月  19 19:43 ..
    							-rw-r--r--.  1 root root   18 4月  19 19:43 .bash_logout
    							-rw-r--r--.  1 root root  193 4月  19 19:43 .bash_profile
    							-rw-r--r--.  1 root root  231 4月  19 19:43 .bashrc
    							drwxr-xr-x.  4 root root   39 4月  19 19:43 .mozilla
    							[root@bogon tmp]# chmod -R go= skel
    							[root@bogon tmp]# ls -la skel
    							总用量 16
    							drwx------.  3 root root   78 4月  19 19:43 .
    							drwxrwxrwt. 24 root root 4096 4月  19 19:43 ..
    							-rw-------.  1 root root   18 4月  19 19:43 .bash_logout
    							-rw-------.  1 root root  193 4月  19 19:43 .bash_profile
    							-rw-------.  1 root root  231 4月  19 19:43 .bashrc
    							drwx------.  4 root root   39 4月  19 19:43 .mozilla
    							[root@bogon tmp]# chmod -R g+r skel/
    							[root@bogon tmp]# ls -la skel
    							总用量 16
    							drwxr-----.  3 root root   78 4月  19 19:43 .
    							drwxrwxrwt. 24 root root 4096 4月  19 19:45 ..
    							-rw-r-----.  1 root root   18 4月  19 19:43 .bash_logout
    							-rw-r-----.  1 root root  193 4月  19 19:43 .bash_profile
    							-rw-r-----.  1 root root  231 4月  19 19:43 .bashrc
    							drwxr-----.  4 root root   39 4月  19 19:43 .mozilla
###### （2）从属关系管理命令：chown,chgrp

仅管理员可修改文件的属主和属组，

**chown命令：    既能修改属主也能修改属组**

		chown [OPTION]... [OWNER][:[GROUP]] FILE...         //修改属主
		chown [OPTION]... --reference=RFILE FILE...         //引用（参考修改）
	        选项：
	          -R:递归修改     
**chgrp命令：    只能用于修改属组**

		chgrp [OPTION]... GROUP FILE...
		chgrp [OPTION]... --reference=RFILE FILE...
	                exmple.     修改属主
								root@bogon tmp]# ls -la skel
								总用量 16
								drwxr-----.  3 root root   78 4月  19 19:43 .
								drwxrwxrwt. 24 root root 4096 4月  19 19:45 ..
								-rw-r-----.  1 root root   18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 root root  193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 root root  231 4月  19 19:43 .bashrc
								drwxr-----.  4 root root   39 4月  19 19:43 .mozilla
								[root@bogon tmp]# chown -R docker skel          
								[root@bogon tmp]# ll -d skel
								drwxr-----. 3 docker root 78 4月  19 19:43 skel
								[root@bogon tmp]# ll -a skel
								总用量 16
								drwxr-----.  3 docker root   78 4月  19 19:43 .
								drwxrwxrwt. 24 root   root 4096 4月  19 19:45 ..
								-rw-r-----.  1 docker root   18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 docker root  193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 docker root  231 4月  19 19:43 .bashrc
								drwxr-----.  4 docker root   39 4月  19 19:43 .mozilla
	
								修改属主和属组
								[root@bogon tmp]# ll -a skel/
								总用量 16
								drwxr-----.  3 docker root   78 4月  19 19:43 .
								drwxrwxrwt. 24 root   root 4096 4月  19 19:55 ..
								-rw-r-----.  1 docker root   18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 docker root  193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 docker root  231 4月  19 19:43 .bashrc
								drwxr-----.  4 docker root   39 4月  19 19:43 .mozilla
								[root@bogon tmp]# chown -R jacky:docker skel/
								[root@bogon tmp]# ll -a skel/
								总用量 16
								drwxr-----.  3 jacky docker   78 4月  19 19:43 .
								drwxrwxrwt. 24 root  root   4096 4月  19 19:57 ..
								-rw-r-----.  1 jacky docker   18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 jacky docker  193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 jacky docker  231 4月  19 19:43 .bashrc
								drwxr-----.  4 jacky docker   39 4月  19 19:43 .mozilla
								[root@bogon tmp]# chown -R root.root skel/
								[root@bogon tmp]# ll -a skel/
								总用量 16
								drwxr-----.  3 root root   78 4月  19 19:43 .
								drwxrwxrwt. 24 root root 4096 4月  19 19:57 ..
								-rw-r-----.  1 root root   18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 root root  193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 root root  231 4月  19 19:43 .bashrc
								drwxr-----.  4 root root   39 4月  19 19:43 .mozilla
	
	                            引用（参考）修改
								[root@bogon tmp]# ll a
								-rwxrw-rw-. 1 tom tom 0 4月  19 19:16 a
								[root@bogon tmp]# chown -R --reference=a /tmp/skel/
								[root@bogon tmp]# ll -a skel/
								总用量 16
								drwxr-----.  3 tom  tom    78 4月  19 19:43 .
								drwxrwxrwt. 24 root root 4096 4月  19 20:02 ..
								-rw-r-----.  1 tom  tom    18 4月  19 19:43 .bash_logout
								-rw-r-----.  1 tom  tom   193 4月  19 19:43 .bash_profile
								-rw-r-----.  1 tom  tom   231 4月  19 19:43 .bashrc
								drwxr-----.  4 tom  tom    39 4月  19 19:43 .mozilla
 **   **思考：用户对目录有写权限，但对目录下的文件没有写权限时，**
**    

（1）能否修改目录下的文件？   不能。
    

（2）能否删除目录下的文件？   可以

 **umask:文件权限反向掩码，遮罩码**

用户拥有自己的umask，用户创建文件后，文件的默认权限如下：

            文件的权限：
                666-umask
            目录的权限：
                777-umask
            注意：之所以文件用666去减，表示文件默认不能拥有执行权限；如果减得的结果中有执行权限，则需要将其加1；
                umask:023
                    666-023=643+1=644
                    777-023=754
​    **umask命令：
**        

`umask:   查看当前umask `       

`umask  MASK:设置umask`

​    注意：此类设定仅对当前shell进程有效；

###### （3）install命令：复制文件并设置权限属性

          install - copy files and set attributes--复制文件并设置属性
       
       单源复制：
          install [OPTION]... [-T] SOURCE DEST
       多源复制：
          install [OPTION]... SOURCE... DIRECTORY...
          install [OPTION]... -t DIRECTORY SOURCE...
       创建目录：   
          install [OPTION]... -d DIRECTORY...


​     
​       常用选项：
​           -m, --mode=MODE: 设定目标文件权限，默认为755
​           -o, --owner=OWNER: 设定目标文件属主;
​           -g, --group=GROUP: 设定目标文件属组;
​    example.
​    				[root@bogon tmp]# install /etc/inittab /tmp
​    				[root@bogon tmp]# ls -l inittab 
​    				-rwxr-xr-x. 1 root root 511 4月  19 20:37 inittab
​    				[root@bogon tmp]# ls -l /etc/inittab 
​    				-rw-r--r--. 1 root root 511 8月   4 2017 /etc/inittab
​    				[root@bogon tmp]# rm inittab 
​    				rm：是否删除普通文件 "inittab"？y
​    				[root@bogon tmp]# install -m 640 /etc/init
​    				init.d/  inittab  
​    				[root@bogon tmp]# install -m 640 /etc/inittab /tmp
​    				[root@bogon tmp]# ls -l inittab 
​    				-rw-r-----. 1 root root 511 4月  19 20:42 inittab
​    				[root@bogon tmp]# install -o jacky -g docker /etc/inittab /root
​    				[root@bogon tmp]# ls -l /root/inittab 
​    				-rwxr-xr-x. 1 jacky docker 511 4月  19 20:44 /root/inittab
​    
​    				-d选项创建目录
​    				[root@bogon tmp]# install -d hello
​    				[root@bogon tmp]# ll -d hello
​    				drwxr-xr-x. 2 root root 6 4月  19 20:45 hello
###### （4）mktemp命令：创建临时文件

    mktemp - create a temporary file or directory 创建临时文件或临时目录
    
    命令格式：mktemp [OPTION]... [TEMPLATE]
           TEMPLATE must contain at least 3 consecutive 'X's in last component（模板文件名中至少包含3个X,(自定义名称.xxx))
    
                -d  --directory 创建临时目录
                -u  --dry-run   仅做测试，不会实际创建文件
                -q  --执行若发生错误，不会显示任何信息
    
                 注意：mktemp会将创建的临时文件名直接返回，因此，可直接通过命令引用保存起来；
                 使用mktemp 命令生成临时文件时，文件名参数应当以"文件名.XXXX"的形式给出，mktemp 会根据文件名参数建立一个临时文件。
                 例如：mktemp tmp.xxxx #生成临时文件 
