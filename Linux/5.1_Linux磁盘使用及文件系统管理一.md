## Linux磁盘使用及文件系统管理介绍

***

### 计算机组成

计算机由CPU,Memory(RAM),I/O组成，再非交互式的服务器上，I/O一般不考虑键盘和显示器，一般包括：磁盘Disks和以太网卡Ethercard

### Disks

Disks:持久存储数据

**接口类型**

```
   IDE(ata): 并口（并行总线），133MB/S    100iops
   SCSI:并口，UltraSCSI320,320MB/S,UltraSCSI640,640MB/S  150iops
   
   SATA: 串口（串行总线）速率比IED更快,6gbps    100+iops
   SAS: 串口，6gbps						150iops-200+iops
   USB: 串口，480MB/S

   PCI-E接口:（直接接到北桥总线）


   并口: 同一线缆可以接多块设备;
        IDE:两个，主，从

        SCSI:
        	宽带：16-1
        	窄带：8-1
   串口: 同一线缆只能接一个设备;
   
   
   每种接口的读写速度不一样，每秒读写次数用iops表示
   io：读写操作
   iops: io per second
   
```

### 硬盘

硬盘分为机械硬盘和固态硬盘

固态硬盘可达到400+iops，精心设计的PCI-E接口的固态硬盘可以达到10W+iops，深圳数十万次iops。

**机械硬盘的组成结构**

```
   		机械硬盘:
   		  track:磁道
   		  sector:扇区，512bytes
   		  cylinder:柱面 不同盘面上的同一编号上的磁道组合起来就叫柱面
   		  		分区划分基于柱面

   		  每个硬盘的平均寻道时间不一样，而平均寻道时间由硬盘的转速决定

   		  此即硬盘转速，转速越快，平均寻到时间越短，读写文件的越快，即硬盘的性能越好		
   		  常见的机械硬盘转速如下
   		  
   		  		5400rpm  5400转每分钟
   		  		7200rpm
   		  		10000rpm
   		  		15000rpm
```

### 硬件设备在Linux上的呈现形式

Linux的哲学思想之一，一切皆文件，因此，设备在Linux上也应该是文件的形式

设备文件类型分两种：

块(block):随机访问，数据交换单位是“块”;

字符(character):线性访问，数据交换单位是“字符”；

**设备文件**

Linux上的设备文件放在/dev目录下:

**/dev**

设备文件：关联至设备的驱动程序;是设备的访问入口;

**设备号:**
	

major: 主设备号，区分设备类型;用于表明设备所需要的驱动程序;
	

minor: 次设备号，区分同种类型下的不同的设备;是特定设备的访问入口;

#### mknod命令

用以创建设备文件的命令，mknod - make block or character special files

**用法：mknod [OPTION]... NAME TYPE [MAJOR MINOR]**

选项：	 	-m MODE: 创建后的设备文件的访问权限;

**example.**

```
[root@localhost ~]# mknod /dev/testdev c 111 1
[root@localhost ~]# ls -l /dev/testdev 
crw-r--r--. 1 root root 111, 1 Feb 19 01:53 /dev/testdev
```



### 磁盘

#### 磁盘命名

设备文件名命名规则由ICANN组织制定，ICANN（The Internet Corporation for Assigned Names and Numbers）互联网名称与数字地址分配机构。

```
IDE接口磁盘文件目录及命名规则:
/dev/hd[a-z]
					例如:/dev/hda,/dev/hdb
SCSI,SATA,USB,SAS等接口磁盘文件目录及命名规则:
/dev/sd[a-z]		
					例如sda,sdb

分区命名规则:
/dev/sda#:
					例如/dev/sda1,/dev/sda2...

注意：CentOS 6和7通通将硬盘设备文件表示为/dev/sd[a-z]#



```

引用设备的方式:
（1）设备文件名
（2）卷标
（3）UUID

### 机械磁盘原理

机械硬盘由坚硬金属材料制成的涂以磁性介质的盘片，盘片两面称为盘面或扇面，都可以记录信息，由磁头对盘面进行操作（如果你有坏的硬盘，可以动手拆开看。嗯？为什么用坏的？用好的可能费钱……）一般用磁头号区分。结构特性决定了机械硬盘如果受到剧烈冲击（摔在地上或是勤奋的你想拆开学习），磁头与盘面可能产生的哪怕是轻微撞击都有可能报废。

继续讲原理：假设磁头不动，硬盘旋转，那么磁头就会在磁盘表面画出一个圆形轨迹并将之磁化，数据就保存在这些磁化区中，称之为磁道，将每个磁道分段，一个弧段就是一个扇区。一个硬盘可以包含多个扇面，扇面同轴重叠放置，每个盘面磁道数相同，具有相同周长的磁道所形成的圆柱称之为柱面，柱面数与磁道数相等。如下图

![img](https://pic3.zhimg.com/80/v2-ae88b94d2f8d90176f0a10e8b35a180e_hd.jpg)



了解了这些，我们就可以对最初的硬盘地址管理方式作一个原理层面的了解：

最初的寻址方式称为CHS，在LBA（Logical Block Address）概念诞生之前，由他负责管理磁盘地址。所谓CHS即柱面（cylinder），磁头（header），扇区（sector），通过这三个变量描述磁盘地址，需要明白的是，这里表示的已不是物理地址而是逻辑地址了。这种方法也称作是LARGE寻址方式。该方法下：

硬盘容量=磁头数×柱面数×扇区数×扇区大小（一般为512byte）。

后来，人们通过为每个扇区分配逻辑地址，以扇区为单位进行寻址，也就有了LBA寻址方式。但是为了保持与CHS模式的兼容，通过逻辑变换算法，可以转换为磁头/柱面/扇区三种参数来表示，和 LARGE寻址模式一样，这里的地址也是逻辑地址了。（固态硬盘的存储原理虽然与机械硬盘不同，采用的是flash存储，但仍然使用LBA进行管理，此处不再详述。）

科普到这里，我们可以试图去理解MBR分区了。现在我们来看看MBR分区的技术原理。

#### 磁盘分区

磁盘分区表有两种：MBR和GTP

#### MBR

MBR在磁盘的0扇区上，磁盘上一个扇区有512个字节，也就是说MBR总共有512bytes。

MBR：Master Boot Record，主分区引导记录。最早在1983年在IBM PC DOS 2.0中提出。前面说过，每个扇区/区块都被分配了一个逻辑块地址，即LBA，而引导扇区则是每个分区的第一扇区，而主引导扇区则是整个硬盘的第一扇区（主分区的第一个扇区）。MBR就保存在主引导扇区中。另外，这个扇区里还包含了硬盘分区表DPT（Disk Partition Table），和结束标志字（Magic number）。扇区总计512字节，MBR占446字节（0000H - 01BDH），DPT占据64个字节（01BEH - 01FDH），最后的magic number占2字节（01FEH – 01FFH）。

![img](https://pic4.zhimg.com/80/v2-039771e933fc23590ca888821fd6903f_hd.png)

```
主引导扇区分为三部分:
前面446bytes: bootloader程序，引导启动操作系统的程序;
中间64bytes: FAT(File Allocation Table)分区表（文件系统分配表），每16bytes标识一个分区，一共只能有4个分区（主分区）
			4主分区
			3主分区，1扩展分区
						扩展分区中可以有n个逻辑分区

最后2bytes:MBR区域的有效表示;55AA为有效;否则为无效

Linux中主分区和扩展分区的表示:1-4，例如sda1,sda2,sda3,sda4
逻辑分区:5+,例如sdb6
```

**example.**

```
现在，我们来看一个MBR记录的实例：

80 01 01 00, 0B FE BF FC, 3F 00 00 00, 7E 86 BB 00

其中， “80”是一个分区的激活标志，表示系统可引导；“01 01 00”表示分区开始的磁头号为01，开始的扇区号为01，开始的柱面号为00；“0B”表示该分区的系统类型是FAT32，其他比较常用的有04(FAT16)、07(NTFS)；“FE BF FC”表示分区结束的磁头号为254，分区结束的扇区号为63、分区结束的柱面号为764；“3F 00 00 00”表示首扇区的相对扇区号为63；“7E 86 BB 00”表示总扇区数为12289622。

可以看到，在只分配64字节给DPT的情况下，每个分区项分别占用16个字节，因此只能记录四个分区信息，尽管后来为了支持更多的分区，引入了扩展分区及逻辑分区的概念。但每个分区项仍然用16个字节存储。能表示的最大扇区数为FF FF,FF FFH，因此可管理的最大空间=总扇区数*扇区大小（512byte），也就是2TB（由于硬盘制造商采用1:1000进行单位换算，因此也有2.2TB一说，别怪他们，他们不是程序员）。超过2TB以后的空间，不能分配地址，自然也就无法管理了。
```

### GPT

GTP结构图

![img](https://pic3.zhimg.com/80/v2-06027b517bf5e4ff73392b8ee09d9b76_hd.png)

**GUID分区表（简称GPT）**特性 	 	

```
		 	GUID分区表（简称GPT）
		 	支持2T以上的大硬盘
		 	每个磁盘的分区个数几乎没有限制
		 	分区大小几乎没有限制
		 	分区表自带备份
		 	每个分区可以有一个不同的名称
		 	UEFI的引导才能够读取GPT分区

		 	使用LBA（逻辑区块地址，预设大小为512字节,等于一个扇区的大小）逻辑区块地址(Logical Block Address, LBA)
		 	
		 	区块必须以硬盘上某个磁柱、磁头、扇区的硬件位置所合成的地址来指定

			LBA0与MBR相同的模块，包括446字节的开机管理程序+一个特殊的分区标识，表示这是个GPT格式。LBA1：GPT表头记录，记录了分区表的本身的位置和大小，同时记录了备份用GPT分区放置的位置，分区表的校验机制码。 LBA2~33：实际记录分区信息。每个LBA记录4个分区信息，因此可以有4*32=128个分区。每个LBA有512个字节，每个分区信息可以占用128字节，用64bit记载开始/结束的扇区号，因此GPT格式的每个分区容量大小限制：2^64*512字节=8ZB。

```

这里的P意为protective，PMBR存在的意义就是，当不支持GPT的分区工具试图对硬盘进行操作时（例如MS-DOS和Linux的fdisk程序），它可以根据这份PMBR以传统方式启动，过程和MBR+BIOS完全一致，极大地提高了兼容性。而支持GPT的系统在检测PMBR后会直接跳到GPT表头读取分区表。和MBR类似，分区表中存储了某个分区的起始和结束位置及其文件系统属性信息，而分区才是实际存在的物理磁盘的一部分。

GPT HDR：GPT表头，如下图，主要定义了分区表中项目数及每项大小，还包含硬盘的容量信息。在64位的Windows Server 2003的机器上，最多可以创建128个分区，即分区表中保留了128个项，其中每个都是128字节。（也是EFI标准中的最低要求：分区表最小要有16,384字节）分区表头还记录了这块硬盘的GUID，分区表头位置（总是LBA1）和大小，也包含了备份分区表头和分区表的位置和大小信息（LBA-1~LBA-34）。同时还储存着它本身和分区表的CRC32校验。固件、引导程序和操作系统在启动时可以根据这个校验值来判断分区表是否出错，如果出错，可以使用软件从硬盘最后的备份GPT中恢复整个分区表，如果备份GPT也校验错误，硬盘将不可使用。具体内容如下表：

![img](https://pic1.zhimg.com/80/v2-4db0c60b8341a818c4c5a17de0acc224_hd.jpg)



Partition Table：分区表，包含分区的类型GUID（如：EFI系统分区的GUID类型是{C12A7328-F81F-11D2-BA4B-00A0C93EC93B}），名称，起始终止位置，该分区的GUID以及分区属性。其内容如下：

![img](https://pic1.zhimg.com/80/v2-e117772cc52454292869e47a78f8b7f8_hd.png)

相较于MBR，GPT具有以下优点：

**（1）**得益于LBA提升至64位，以及分区表中每项128位设定，GPT可管理的空间近乎无限大，假设一个扇区大小仍为512字节，可表示扇区数为，算下来，可管理的硬盘容量=18EB(1EB=1024PB=1,048,576TB)，2T在它面前完全不在话下。按目前的硬盘技术来看，确实近乎无限，不过，以后的事谁知道呢。

**（2）**分区数量几乎没有限制，由于可在表头中设置分区数量的大小，如果愿意，设置个分区也可以（有人愿意管理这么多分区吗），不过，目前windows仅支持最大128个分区。

**（3）**自带保险，由于在磁盘的首尾部分各带一个GPT表头，任何一个受到破坏后都可以通过另一份恢复，极大地提高了磁盘的抗性（两个一起坏的请出门买彩票）。

**（4）**循环冗余检验值针对关键数据结构而计算，提高了数据崩溃的检测几率。

**（5）**尽管目前分区类型不超过百数（十数也没有吧。），GPT仍提供了16字节的GUID来标识分区类型，使其更不容易产生冲突。

**（6）**每个分区都可以拥有一个特别的名字，最长72字节，足够写一首七律了。满足你的各种奇葩起名需求。

完美支持UEFI，毕竟它就是UEFI规范的衍生品。在将来全行业UEFI的情境下，GPT必将更快淘汰MBR。

### Linux分区管理

#### fdisk命令

**fdisk - manipulate disk partition table**

两种用法：

1. 查看文件的分区信息

   **语法：fdisk -l [-u] [device...]**

   列出指定磁盘设备上的分区情况;

   **example**

   ```
   [root@localhost ~]# fdisk -l
   
   Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x00098069
   
      Device Boot      Start         End      Blocks   Id  System
   /dev/sda1   *        2048      616447      307200   83  Linux
   /dev/sda2          616448     4810751     2097152   82  Linux swap / Solaris
   /dev/sda3         4810752    41943039    18566144   83  Linux
   [root@localhost ~]# fdisk -l /dev/sda
   
   Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
   Units = sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   Disk label type: dos
   Disk identifier: 0x00098069
   
      Device Boot      Start         End      Blocks   Id  System
   /dev/sda1   *        2048      616447      307200   83  Linux
   /dev/sda2          616448     4810751     2097152   82  Linux swap / Solaris
   /dev/sda3         4810752    41943039    18566144   83  Linux
   
   ```

   

2. 管理分区

   **语法：fdisk device**

```
fdisk提供了一个交互式接口来管理分区，它有许多字敏玲，分别用于不同的管理功能;所有的操作均在内存中完成，没有直接同步到磁盘;直到使用w命令保存至磁盘上，它才会生效。
常用命令:
    n:创建新分区
    d:删除已有分区
    t:修改分区类型
    l:查看所有已知ID
    w:保存并退出
    q:不保存并退出
    m:查看帮助信息
    p:显示现有分区信息
```

**example.**

```
[root@localhost ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): m
Command action
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

Command (m for help): l

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary  
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
1e  Hidden W95 FAT1 80  Old Minix      
```

**读取磁盘分区操作**

	注意：在已经分区并且已经挂载其中某个分区的磁盘设备上创建的新分区 ,内核可能在创建完成后无法直接识别；
	
	查看: cat /proc/partitions
	通知内核强制重读磁盘分区表:
	语法：		  CentOS 5: partprobe [device]
		    	CentOS 6,7:partx,kpartx
						partx -a [device]
						kpartx -a [device]			有时候可能需要执行2次
其他分区创建工具:parted,sfdisk等。

### 创建文件系统

我们要使用磁盘，需要多其进行格式化，格式化分两种

（1）格式化:低级格式化（分区之前进行，划分磁道）

（2）高级格式化（分区之后对分区进行，创建文件系统）

一般的磁盘分为元数据区和数据区：

```
元数据区：
			文件元数据：inode（index node）索引节点  （inode表，inode编号）
			大小、权限、属主属组、时间戳、数据块指针...
			inode bit map index 位图索引
			block bit map index 位图索引
					
数据区：
			只有数据块（存数据）
			
	注意：元数据和数据都在块中存放，某些块存元数据，某些块存数据
```

**超级块**

磁盘也可以在内部划分成多个逻辑块组，每一个块组可以有自己的元数据区和数据区，并有一个超级块。

超级块定义我们有多少个块组，每一个块组开始和结束的位置。

超级块也属于块组，在块组中的某个块中存放

块组中的元数据区中有块组描述符

**符号链接文件和设备文件的区别**

符号链接文件：存储数据指针的空间当中存储的是真实文件的访问路径；

备文件：存储数据指针的空间当中存储的是设备号（major,minor）;

#### VFS

**VFS：Virtual File System**

				Linux的文件系统:ext2,ext3,ext4,xfs,reiserfs,btrfs
				光盘:ios9660
				网络文件系统：nfs,cifs
				集群文件系统：gfs2,ocfs2
				内核级分布式文件系统:ceph
				windows的文件系统:vfat,ntfs
				伪文件系统:proc,sysfs,tmpfs,hugepagefs
				Unix的文件系统:UFS,FFS,JFS
				交换文件系统：swap
				用户控件的分布式文件系统:mogiles,moosefs,glusterfs
#### Journal(日志)

文件系统根据有无日志可分为两类

1、有日志文件系统（例如ext3,ext4,xfs,reiserfs,btrfs）

磁盘中创建日志区，新文件在创建时元数据可以存在日志区，文件创建完毕（元数据创建完成，数据块分配好，存文件动作结束），元数据移回元数据区

2.无日志文件系统（例如ext2）

#### 链接文件

链接文件：访问一文件的不同路径;

**硬链接**

		硬链接：指向同一个inode的多个文件路径；
			特性：
			（1）目录不支持硬链接；（为避免循环链接）
			（2）硬链接不能垮文件系统；（不同文件系统inode独立管理）
			（3）创建硬链接会增加inode引用计数
**example.引用计数**

```
[root@localhost ~]# ll
total 12
-rw-------. 1 root root 2776 Feb 16 20:26 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Desktop
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Documents
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Downloads
-rw-r--r--. 1 root root 2874 Feb 16 21:16 initial-setup-ks.cfg
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Music
-rw-------. 1 root root 2056 Feb 16 20:26 original-ks.cfg
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Pictures
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Public
drwxr-xr-x. 2 root root    6 Feb 16 21:18 Templates

以上文件第二列的，1和2就是引用计数，表明文件inode被引用的次数
```

**创建硬链接命令**

	ln:创建硬链接命令
			ln src link_file
#### 软链接 （符号链接）

符号链接（软连接）：指向一个文件路径的另一个文件路径；

特性：

（1）符号链接与文件是两个各自独立的文件，各有自己的inode;对原文件创建符号链接不会增加引用计数；

（2）支持对目录创建符号链接，可以垮文件系统；

（3）删除符号链接文件不影响原文件;但删除原文件，符号链接指向的路径将不存在，此时会变成无效链接;

注意：符号链接文件的大小是其指向的文件的路径字符串的字节数；

权限:lrwxrwxrwx

**创建软链接命令**

			ln -s 创建软连接
			ln -s src link_file
	
			   -v:verbose显示过程
#### 磁盘如何实现删除、复制和移动文件

	删除文件:将此文件指向的所有data block标记为未使用状态;将此文件的inode标记为未使用;并修改inode位图和block位图
	
	复制和移动文件：
	      复制:相当于新建文件;
	      移动文件:
	          在同一文件系统：改变的仅是其路径;
	          在不同文件系统：复制数据至目标文件，并删除原文件;