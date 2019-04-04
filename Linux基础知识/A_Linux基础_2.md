#### （一）Linux基础（2）

***

- 1、**Linux上的文件管理类命令**

  Linux上的文件管理类命令有：cp，mv,rm.

   (1)、cp命令： copy
              源文件：目标文件；

  			cp - copy files and directories
  	
  		单源复制： cp [OPTION]... [-T] SOURCE DEST
  	   	 多源复制： cp [OPTION]... SOURCE... DIRECTORY
  		                    cp [OPTION]... -t DIRECTORY SOURCE...
  	
  	  A、单源复制： cp [OPTION]... [-T] SOURCE DEST
  	                 如果DEST不存在，则事先创建此文件，并复制源文件的数据流至DEST中；
  	                 如果DEST存在：
  	                 如果DEST是非目录文件；则覆盖目标文件；
  	                 如果DEST是目录文件：则现在DEST目录下创建一个与源文件同名的文件，并复制其数据流；
  	
  	  B、多源复制
  	                  cp [OPTION]... SOURCE... DIRECTORY
  	                  cp [OPTION]... -t DIRECTORY SOURCE...
  	
  	                如果DEST不存在：错误；
  	                如果DEST存在:
  	                     如果DEST是非目录文件： 错误；
  	                     如果DEST是目录文件： 分别复制每个文件至目标目录中，并保持原名;
  	
  	    常用选项
  	         -i: 交互式复制，即覆盖之前提醒用户去人；
  	         -f: --force 强制覆盖目标文件； 
  	         -r，-R: 递归复制目录；
  	         -d: 复制符号链接文件本身，而非其指向的源文件；
  	         -a：-dR --preserve=all,archive,用于实现归档；
  	         --preserv=
  	               mode:权限
  	               ownership:属主和属组
  	               timestamps：时间戳
  	               context:安全标签
  	               xattr:扩展属性
  	               links: 符号链接
  	               all: 上述所有属性
     

  (2)、mv命令

  ​	用法和cp命令类似

  ​		mv命令：move   mv - move (rename) files

     		命令格式：

     				mv [OPTION]... [-T] SOURCE DEST
     				mv [OPTION]... SOURCE... DIRECTORY
     				mv [OPTION]... -t DIRECTORY SOURCE...

    		源文件和目标文件再同一个目录下，即重命名
         		常用选项：
             		-i:交互式
             		-f:force

     

   (3)、rm命令： remove

   命令格式：rm [OPTION]... FILE...

  常用选项：

  ​     -i: interactive 交互式

  ​     -f: force  强制

  ​     -r： recursive 递归

  删除目录：rm -rf /PATH/TO/DIR

  危险操作： rm -rf /*

  注意：在生产环境中，所有不用的文件建议不要直接删除，而是移动至某个人专用目录；（模拟回收站）

  

- 2、**bash的基础特性：命令行展开**

  ~：自动展开为用户的家目录，或指定的用户的家目录；

  {}：可承载一个以逗号分隔的路径列表，并能够将其展开为多个路径；

  

  实战：

  使用命令行展开功能

  1、创建/tmp/a1, /tmp/a2, /tmp/a1/a, /tmp/a1/b，

  2、在/tmp目录下创建目录：x_y, x_z, q_y, q_z

  ```
  [root@bogon tmp]# mkdir -pv /tmp/{a1/{a,b},a2}
  mkdir: 已创建目录 "/tmp/a1"
  mkdir: 已创建目录 "/tmp/a1/a"
  mkdir: 已创建目录 "/tmp/a1/b"
  mkdir: 已创建目录 "/tmp/a2"
  ```

  ```
  [root@bogon trash]# mkdir -v /tmp/{x,q}_{y,z}
  mkdir: 已创建目录 "/tmp/x_y"
  mkdir: 已创建目录 "/tmp/x_z"
  mkdir: 已创建目录 "/tmp/q_y"
  mkdir: 已创建目录 "/tmp/q_z"
  ```

  

- 3、**什么是文件的元数据**

  对Linux文件系统来讲，每个文件都由两类数据组成：

  ​         元数据：metadata

  ​         数据：data

  数据是是文件的内容，而元数据是文件的属性信息，我们可以用stat命令来查看文件的元数据

  (1)、stat命令：
           stat - display file or file system status

   命令格式： stat FILE...

  ```
  [root@bogon ~]# stat /tmp/hi.txt 
    File: ‘/tmp/hi.txt’
    Size: 23        	Blocks: 8          IO Block: 4096   regular file
  Device: fd00h/64768d	Inode: 67309026    Links: 1
  Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
  Context: unconfined_u:object_r:user_tmp_t:s0
  Access: 2019-04-04 17:53:48.707013568 +0800
  Modify: 2019-04-04 17:53:48.707013568 +0800
  Change: 2019-04-04 17:53:48.707013568 +0800
   Birth: -
  ```

| 元数据信息   | 解释说明                   |
| ------------ | -------------------------- |
| File         | 文件名                     |
| size         | 文件的大小                 |
| blocks       | 占用block的数量            |
| io block     | block总大小为4096          |
| regular file | 定义文件类型-常规文件      |
| Device       | 设备编号的十六进制和十进制 |
| Inode        | 文件的Inode值              |
| Links        | 文件的硬链接数             |
| 第一个Access | 文件的权限                 |
| Context      | 注释信息                   |
| Access       | 访问时间                   |
| Modify       | 修改时间                   |
| Change       | 改动时间                   |

​		状态信息有三个时间戳：

​                access time：访问时间，atime即：通过cat,more等读取其内容的最近一次时间

​                modifiy time: 修改时间，mtime即：改变文件内容的时间

​                change time: 改变时间，ctime：即元数据发生改变的时间

​		修改文件的时间戳信息使用touch命令

​           (2)、touch命令

   		touch - change file timestamps   

  		 touch [OPTION]... FILE...         (如果touch的文件不存在默认会创建一个空文件)
        		-c: 指定的文件路径不存在是不予创建
       		 -a:仅修改access time
        		-m:仅修改modify time
        		-t STAMP
          		use [[CC]YY]MMDDhhmm[.ss]   instead of current time

​                PS:change time是无法指定修改的,他随元数据发生改变而随之改变

- 4
- 5
- 6
- 7