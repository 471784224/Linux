## sudo的用法

***

Linux是一种多用户的操作系统，统一时间可以登录多个用户进行操作，同时在LiNux之上用户又分为系统用户和普通用户，不同的用户对系统（系统中的文件）拥有不同的操作权限。

​     root拥有所有的操作权限，其他普通用户只有定义的操作权限，这里的定义是通过一定的管理机制来进行限定，如果普通用户想执行一些超越自身权限的命令式就必须以拥有该命令权限的用户身份或者切换到拥有对应权限的用户当中去执行。

​     **用户身份切**

* su命令

  通过su命令可以切换到其他用户来进行一些操作，但是前提是我们必须拥有所切换用户的口令信息

* sudo命令

  假如我们想让一个普通用户拥有一些只有root可以执行的命令，但又不能讲root的密码信息告诉别人，此时可以使用sudo命令

  并非所有的用户都能够运行sudo命令，这由sudo的配置文件/etc/sudoers来进行定义

  1. 语法格式

     **sudo [-u USERNAME] COMMAND**

     通常是普通用户以root身份来执行命令，sudo不加-u参数默认就是以root身份执行COMMAND

     因此，可以直接使用 **sudo COMMAND**

  2. 配置文件：/etc/sudoers

     配置sudo必须通过编辑`/etc/sudoers`文件，而且只有root才可以修改它，还必须使用visudo命令编辑。之所以使用visudo有两个原因，一是它能够防止两个用户同时修改它；二是它也能进行有限的语法检查。

     `visudo`

     语法规则：

     ​		账号	登陆者的来源主机名（可切换的身份）  可以通过sudo执行的命令

     例如：jacky		ALL=(root)			/usr/bin/passwd

     这表示从任何主机访问过来的本地用户jacky允许切换到root用户来执行/usr/bin/passwd命令

      exmaple2： `jacky   ALL=(root)      ALL`

     表示允许任意主机的本地用户jacky切换到root用户执行所有命令

     1. ALL代表所有，必须大写

     2. 如果后面是命令必须为绝对路径,多个命令用逗号隔开

     3. 前面的使用者账号可以是一个组，这样表示：%GROUP_NAME，例如%dockeruser

     4. 如果在执行sudo的时候不需要输入密码，则可以在命令前面这样表示：

        NOPASSWD: COMMAND 

        强制密码验证则使用

        PASSWD: COMMAND

        example：

        %wheel  ALL=(ALL)  NOPASSWD: ALL

        表示wheel组内的所有用户可以通过任何主机切换到任何用户，可以免密执行任何命令

     5. 如果可以同时切换多个用户如何表示：

        `foobar    linux=(jimmy,rene)    /bin/kill`

        主机名为linux上的用户 foobar可以切换到jimmy,rene这两个用户下，允许执行/bin/kill命令

        切换的的时候需要使用-u选项

        如果不想使用-u选项，可以设置默认切换用户，如下

        `Defaults:foobar    runas_default=jimmy`

     6. 其他管理选项

        

        命令格式：sudo [options] COMMAND

         选项：

        ```
        -b：在后台执行指令；
        -h：显示帮助；
        -H：将HOME环境变量设为新身份的HOME环境变量；
        -k：结束密码的有效期限，也就是下次再执行sudo时便需要输入密码；。
        -l：列出目前用户可执行与无法执行的指令；
        -p：改变询问密码的提示符号；
        -s<shell>：执行指定的shell；
        -u<用户>：以指定的用户作为新的身份。若不加上此参数，则预设以root作为新的身份；
        -v：延长密码有效期限5分钟；
        -V ：显示版本信息。
        ```

     **日志与安全**

     sudo为安全考虑得很周到，不仅可以记录日志，还能在有必要时向系统管理员报告。但是，sudo的日志功能不是自动的，必须由管理员开启。这样来做：

     ```
     touch /var/log/sudo
     vi /etc/syslog.conf
     ```

     在syslog.conf最后面加一行（必须用tab分割开）并保存：

     ```
     local2.debug                    /var/log/sudo
     ```

     重启日志守候进程，

     ```
     ps aux grep syslogd
     ```

     把得到的syslogd进程的PID（输出的第二列是PID）填入下面：

     ```
     kill –HUP PID
     ```

     这样，sudo就可以写日志了：

     ```
     [foobar@localhost ~]$ sudo ls /rootanaconda-ks.cfg
     Desktop install.log
     install.log.syslog
     $cat /var/log/sudoJul 28 22:52:54 localhost sudo:   foobar :
     TTY=pts/1 ; pwd=/home/foobar ; USER=root ; command=/bin/ls /root
     ```

     不过，有一个小小的“缺陷”，sudo记录日志并不是很忠实：

     ```
     [foobar@localhost ~]$ sudo cat /etc/shadow > /dev/null
     cat /var/log/sudo...Jul 28 23:10:24 localhost sudo:   foobar : TTY=pts/1 ;
     PWD=/home/foobar ; USER=root ; COMMAND=/bin/cat /etc/shadow
     ```

     重定向没有被记录在案！为什么？因为在命令运行之前，shell把重定向的工作做完了，sudo根本就没看到重定向。这也有个好处，下面的手段不会得逞：

     ```
     [foobar@localhost ~]$ sudo ls /root > /etc/shadowbash: /etc/shadow: 权限不够
     ```

     sudo 有自己的方式来保护安全。以root的身份执行`sudo-V`，查看一下sudo的设置。因为考虑到安全问题，一部分环境变量并没有传递给sudo后面的命令，或者被检查后再传递的，比如：PATH，HOME，SHELL等。当然，你也可以通过sudoers来配置这些环境变量。



  

​    



