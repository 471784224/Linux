## Linux磁盘使用及文件系统提供管理使用进阶

***

题外话：

      查看系统版本：
      		lsb_release
      		cat /etc/issue
    
      查看内核版本：
      		uname-r
### 一、文件系统管理相关概念

#### 1. 文件系统的组织结构中的术语

block groups，块组

block，磁盘块

inode table,索引表

inode,索引

inode bitmap,索引位图

block bitmap,块位图

superblock,超级块

#### 2. 文件系统管理工具

创建文件系统的工具

**mkfs：**	

​	**mkfs.ext2,mkfs.ext3,mkfs,ext4,mkfs,xfs,mkfs,vfat,..**

检测及修复文件系统的工具

**fsck**
**:**	

​	**fsck.ext2,fsck.ext3,...**

查看其属性的工具
：	

​	**dumpe2fs,tune2fs**

调整文件系统特性:
	

​	**tune2fs**

#### 3. 内核级文件系统的组成部分

文件系统驱动：由内核提供

文件系统管理工具：由用户空间的应用程序提供

#### 4. 查看文件系统

**blkid命令**

	 blkid device
	 blkid -L LABEL: 根据指定LABEL定位设备
	 blkid -U UUID: 根据UUID定位设备
example.

```
[root@localhost ~]# blkid
/dev/sda3: UUID="8068cc2b-a5fe-4a18-9c5a-a51914615c6f" TYPE="ext4" 
/dev/sda1: UUID="0254349a-2c68-4546-a4f9-0156a1aeceb7" TYPE="ext4" 
/dev/sda2: UUID="6440d340-ef6e-4c07-9396-e38274edce09" TYPE="swap" 
You have new mail in /var/spool/mail/root

```

### 二、ext系列文件系统的管理工具

#### 1. 创建文件系统的工具

```
mkfs.ext2,mkfs.ext3,mkfs.ext4

mkfs -t ext2 = mkfs.ext2
```

#### 1.1  ext系列文件系统提供专用管理工具:mke2fs

	mke2fs [OPTIONS] device
			-t {ext2|ext3|ext4}:指明要创建的文件系统类型
			  mkfs.ext4 = mkfs -t ext4 = mke2fs -t ext4
			-b{1024|2048|4096}:指明文件系统的块大小
			-L LABEL:指明卷标;
			-j:创建有日志功能的文件系统ext3;
			  mke2fs -j = mke2fs -ext3 = mkfs -t ext3 =mkfs.ext3
			-i #:bytes-per-inode,指明inode于字节的比率;即每多少字节创建一个inode
			-N #：直接要给此文件系统创建的inode的数量；
			-m #:指定预留的空间，百分比;#%
			-O [^]FEATURE: 以指定的特性来创建目标文件系统;如加^表示关闭此特性
#### 1.2 e2label命令

卷标的查看与设定
	  
	  查看: e2label device
	  设定：e2label device LABEL
#### 1.3 tune2fs命令

```
查看或修改ext系列文件系统的某些属性
tune2fs - adjust tunable filesystem parameters on ext2/ext3/ext4 filesystems
	注意:块大小创建后不可修改;

	tune2fs [OPTIONS] device
 	    -l:查看超级块的内容；


 	    修改指定文件系统的属性:
 	 	  -j:将ext2-->ext3  将ext2升级为ext3
 	 	  -L LABEL:修改卷标；
 	 	  -m #:调整预留给管理员的空间百分比;#%
 	 	  -O [^]FEATURE:开启或关闭某种特性;

 	 	  -o [^]mount_options: 开启或关闭某种默认挂载选项
 	 			acl  设置支持facl的挂载选项
 	 			^acl 设置不支持facl的挂载选项
```

#### 1.4 dumpe2fs命令

```
显示ext系列文件系统的属性信息
		dumpe2fs [-h] device
```

#### 1.5 CentOS 6如何使用xfs文件系统

	#yum install xfsprogs
	
	创建：mkfs.xfs
	检测: fsck.xfs
	
	其他：
	修改yum安装源操作：
	
	[root@localhost ~]# cd /etc/yum.repos.d/
	[root@localhost yum.repos.d]# ll
	total 28
	-rw-r--r--. 1 root root 1664 Aug 30  2017 CentOS-Base.repo
	-rw-r--r--. 1 root root 1309 Aug 30  2017 CentOS-CR.repo
	-rw-r--r--. 1 root root  649 Aug 30  2017 CentOS-Debuginfo.repo
	-rw-r--r--. 1 root root  314 Aug 30  2017 CentOS-fasttrack.repo
	-rw-r--r--. 1 root root  630 Aug 30  2017 CentOS-Media.repo
	-rw-r--r--. 1 root root 1331 Aug 30  2017 CentOS-Sources.repo
	-rw-r--r--. 1 root root 3830 Aug 30  2017 CentOS-Vault.repo
	[root@localhost yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak     //备份原文件
	后面再上传或wget命令下载新文件，或者在原文件上修改yum源

#### 2. 用于实现文件系统检测的工具

因进程意外终止或系统崩溃等原因，导致写入操作非正常终止时，可能会造成文件顺怀；此时，应该检测并修复文件系统；建议，离线进行。

#### 

#### 2.1 fsck命令

	fsck:
		check and repair a Linux filesystem
	
	  fsck [OPTION] device
			-t fstype: 指明文件系统类型;
				例如:fsck -t ext4 = fsck.ext4
			-a: 无须交互而自动修复所有的错误;
			-r: 交互式修复;
#### 2.2  ext系列文件系统的专用工具

**e2fsck**

```
e2fsck: 
  	 check a Linux ext2/ext3/ext4 file syste
  	 
  e2fsck[OPTION] device
  	  -y:对所有问题自动回答yes;
  	  -f:即便文件系统处于clean状态，也要强制进行检测；
```

#### 2.3 example.

```
[root@localhost ~]# mkfs.ext4 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=                                                 //卷标位空
OS type: Linux                                                    //操作系统类型为Linux
Block size=4096 (log=2)											  //块大小为4096字节 4K
Fragment size=4096 (log=2)						//分块大小为4K
Stride=0 blocks, Stripe width=0 blocks
327680 inodes, 1310720 blocks			//为分区创建了327680个索引，1310720个磁盘块
65536 blocks (5.00%) reserved for the super user         、、有多少块预留下来给超级管理员
First data block=0									//第一个数据块编号从0开始	
Maximum filesystem blocks=1342177280	//最大文件系统块编号 （编号和个数并不完全一样）
40 block groups                        //一共多少个块组
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 						//超级块备份在哪些块上
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done               //正在写入inode表：完成              
Writing inode tables: done                            
Creating journal (32768 blocks): done				//创建日志：完成
Writing superblocks and filesystem accounting information: done 

[root@localhost ~]# blkid /dev/sdb1
/dev/sdb1: UUID="0eb7970a-e0fd-4916-bee6-d4f2f335a1ef" TYPE="ext4"  //UUID全局唯一标识符

[root@localhost yum.repos.d]# mkfs -t xfs /dev/sdb1
mkfs.xfs: /dev/sdb1 appears to contain an existing filesystem (ext4).
mkfs.xfs: Use the -f option to force overwrite.
[root@localhost yum.repos.d]# mkfs -t xfs -f /dev/sdb1	//重新格式化为xfs			
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=327680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=1310720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost yum.repos.d]# blkid /dev/sdb1
/dev/sdb1: UUID="627ae4e1-a35c-46d5-89be-7995c4b3d499" TYPE="xfs" 

[root@localhost yum.repos.d]# mke2fs -t ext4 -L MYDATA -b 2048 /dev/sdb1    //指定卷标和块大小
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=MYDATA
OS type: Linux
Block size=2048 (log=1)
Fragment size=2048 (log=1)
Stride=0 blocks, Stripe width=0 blocks
327680 inodes, 2621440 blocks
131072 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=271056896
160 block groups
16384 blocks per group, 16384 fragments per group
2048 inodes per group
Superblock backups stored on blocks: 
	16384, 49152, 81920, 114688, 147456, 409600, 442368, 802816, 1327104, 
	2048000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   

[root@localhost yum.repos.d]# blkid /dev/sdb1
/dev/sdb1: LABEL="MYDATA" UUID="25f83709-6673-4550-b36f-dcd7df304408" TYPE="ext4" 




[root@localhost yum.repos.d]# blkid /dev/sdb1
/dev/sdb1: UUID="068a0ab6-0232-41f6-ac8c-504a79c64eb2" TYPE="ext2" 
[root@localhost yum.repos.d]# tune2fs -j /dev/sdb1
tune2fs 1.42.9 (28-Dec-2013)
Creating journal inode: done
[root@localhost yum.repos.d]# blkid /dev/sdb1
/dev/sdb1: UUID="068a0ab6-0232-41f6-ac8c-504a79c64eb2" SEC_TYPE="ext2" TYPE="ext3" 
[root@localhost yum.repos.d]# tune2fs -m 2 /dev/sdb1
tune2fs 1.42.9 (28-Dec-2013)
Setting reserved blocks percentage to 2% (26214 blocks)
[root@localhost yum.repos.d]# tune2fs -O ^has_journal /dev/sdb1
tune2fs 1.42.9 (28-Dec-2013)
[root@localhost yum.repos.d]# blkid /dev/sdb1
/dev/sdb1: UUID="068a0ab6-0232-41f6-ac8c-504a79c64eb2" TYPE="ext2" 


[root@localhost yum.repos.d]# e2fsck /dev/sdb1
e2fsck 1.42.9 (28-Dec-2013)
/dev/sdb1: clean, 11/327680 files, 23134/1310720 blocks
[root@localhost yum.repos.d]# e2fsck -f /dev/sdb1
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/sdb1: 11/327680 files (0.0% non-contiguous), 23134/1310720 blocks
```

#### 2.4 swap文件系统

Linux上的交换分区必须使用独立的文件系统;

且文件系统的system ID 必须为82;

**mkswap命令**

创建swap设备的命令

```
mkswap [OPTIONS] device
   -L LABEL: 指明卷标
   -f: 强制

		example.
		[root@localhost yum.repos.d]# mkswap /dev/sdb5
		Setting up swapspace version 1, size = 5240828 KiB
		no label, UUID=40729c5c-cc30-41a8-a8b1-cdfe1969c0eb
```

**交换分区当前启用和禁用**

```
启用: swapon
        	swapon [OPTION] [DEVICE]
        		 -a:定义在/etc/fstab文件中的所有swap设备;

禁用: swapoff
        	swapoff DEVICE
```

example.

```
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3774         304        3109           9         360        3191
Swap:          2047           0        2047
[root@localhost ~]# swapon /dev/sdb5
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3774         308        3105           9         360        3188
[root@localhost ~]# swapoff /dev/sdb5
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3774         304        3108   

```



### 3. 文件系统的使用

#### 1. 挂载

根文件系统之外的其他文件系统要想能够被访问，都必须通过“关联”至跟文件系统上的某个目录来实现；此关联操作即为“挂载”；此目录即为“挂载点”。

**挂载点**mount_point,用于作为另一个文件系统的访问入口;

        			（1）事先存在;
        			（2）应该使用未被或不会被其他进程使用到的目录;
        			（3）挂载点下原有的文件将会被隐藏;
#### 2. mount命令

```
mount [-fnrsvw] [-t vfstype] [-o options] device dir

			命令选项:
			 -r:readonly,只读挂载;
			 -w:read and write,读写挂载;（默认读写挂载）
			 -n:默认情况下，设备挂载或卸载的操作会同步更新至/etc/mtab文件中；-n用于禁止此特性;

			 -t vfstype:指明要挂载的设备上的文件系统的类型;多数情况下可省略，此时mount会通过blkid来判断要挂载的设备的文件系统类型;
			 -L LABEL:挂载时以卷标的方式指明设备；
			 	 mount -L LABEL dir
			 -U UUID:挂载时以UUID的方式指明设备；
			     mount -U UUID dir

		-o options: 挂载选项
			sync/async: 同步/异步操作;
			atime/noatime: 文件或目录在被访问时是否更新其访问时间戳;
			diratime/nodiratime: 目录在被访问时是否更新其访问时间戳；
			remount: 重新挂载;
			acl:支持使用facl功能；
				# mount -o -acl device dir
				# tune2fs -o acl device
				       	example. ~]# mount -o remount,acl /dev/sdb1 /mnt

			ro: 只读
			rw: 读写
			dev/nodev: 此设备上是否允许创建设备文件;
			exec/noexec: 是否允许运行此设备上的程序文件;(noexec可以防止未经授权的程序自动运行)
			auto/noauto: 是否支持自动挂载
			user/nouser: 是否允许普通用户挂载此文件系统;
			suid/nosuid: 是否允许程序文件上的suid和sgid特殊权限生效
			realtime/norealtime: Update inode access times relative to modify or change time.

			defaults:  使用默认选项 Use default options: rw, suid, dev, exec, auto, nouser, async, and relatime.
			
			指明多个选下可以用逗号隔开


		一个使用技巧:
				可以实现将目录绑定至另一个目录上，作为其临时访问入口;


	查看当前设备所有已挂载的设备:

		# mount
		# cat /etc/mtab
		# cat /proc/mounts

		挂载光盘:

			mount -r /dev/cdrom mount_point

			光盘设备文件: /dev/cdrom,/dev/dvd

		挂载U盘:
		    事先识别U盘的设备文件;


		 挂载本地的回环设备:

		 	# mount -o loop /PATH/TO/SOME_LOOP_FILE MOUNT_POINT

		 	     多见于挂载ISO镜像文件
```

#### 3. umount命令

```
   umount device|dir

 注意:正在被进程访问到的挂载点无法被卸载;
   查看被哪个或哪些进程所占用：

     # lsof MOUNT_POINT
 或  # fuser -v MOUNT_POINT

     终止所有正在访问某挂载点的进程:
     # fuser -km MOUNT_POINT
```

#### 4. /etc/fstab文件

设定除根文件系统以外的其他文件系统能够开机时自动挂载,需要将挂载编辑到 /etc/fstab文件中

**语法规则：**

```
每行定义一个要挂载的文件系统及其相关属性:
    6个字段:
      (1)要挂载的设备;
            设备文件;
            LABEL
            UUID
            伪文件系统:如sysfs,proc,tmpfs等
                伪文件系统，只存在内存当中，而不占用外存空间。它以文件系统的方式为访问系统内数据的操作提供接口。用户和应用程序可以通过proc得到系统的信息，并可以改变内核的某些参数。

      (2)挂载点
           swap类型的设备的挂载点为swap;

      (3)文件系统类型；
      (4)挂载选项
          defaults:使用默认挂载选项;
          如果要同时使用多个挂载选项，彼此间以逗号分隔;
             defaults,acl,noatime,noexec
      (5)转储频率
            0：从不备份；
            1：每天备份；
            2：每隔一天备份；

     （6)自检次序
            0：不自检
            1：首先自检，通常只能是根文件系统可用1；
            2：次级自检
            3；
            .
            .
            .
            9；
```

**mount -a**

mount -a:此命令可自动挂载指定在/etc/fstab文件中的所支持自动挂载的设备。

example.

```
[root@localhost ~]# vi /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Mon Feb 17 04:21:43 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=e6cabb97-1faf-4bab-b38f-28dcee50565f /                       xfs     defaults        0 0
UUID=c9975e53-188b-49ff-8b61-ecdd5b27d58a /boot                   xfs     defaults        0 0
UUID=4a99158e-d94a-4441-b572-44a3cbdb8ed5 swap                    swap    defaults        0 0
/dev/sdb1                                 /mydata                 ext4    defaults,acl    0 0

[root@localhost ~]# mount -a
```

#### 5. df和du命令

```
df命令（disk free）
      report file system disk space usage

      df [OPTION]... [FILE]...
                  -l:仅显示本地文件的相关信息;
                  -h:humen-readable
                  -i:显示inode的使用状态而非block

du命令
      du - estimate file space usage 评估文件空间占用大小

      du [OPTION]... [FILE]...
      -s:sumary 显示目录时，可显示目录下所有文件大小之和
      -h:human-readable
```



#### 其他

Windwos无法识别Linux的文件系统;因此，存储设备需要两种系统之间交叉使用时，应该使用windows和Linux同时支持的文件系统：fat32(vfat);

    # mkfs.vfat device