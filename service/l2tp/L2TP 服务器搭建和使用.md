## L2TP 服务器搭建和使用

***

### 一、安装脚本

下载脚本一键安装l2tp vpn


wget --no-check-certificate https://raw.githubusercontent.com/teddysun/across/master/l2tp.sh  

chmod +x l2tp.sh      //添加可执行权限

. l2tp.sh     执行脚本  如果非root用户使用su sh

 根据提示进行安装



### 二、服务器启动命令

**启动服务**

`service ipsec start` 

`service xl2tpd start` 

**停止服务**

`service ipsec stop`

`service xl2tpd stop`

**重启服务**

`service ipsec restart`

`service xl2tpd restart`    

**分析ipsec的状态**

`ipsec verify`

**查看服务是否启动成功**

```
[root@centos7 download]# ps -ef | grep -E "(ipsec|l2tp)" 
root      4164     1  0 17:32 ?        00:00:00 /usr/sbin/xl2tpd -D
root      4461     1  0 17:32 ?        00:00:00 /usr/libexec/ipsec/pluto --leak-detective --config /etc/ipsec.conf --nofork
root      4513  3692  0 17:40 pts/4    00:00:00 grep --color=auto -E (ipsec|l2tp)
```

将启动命令写入到/etc/rc.local中，并查看/etc/rc.local是否为链接文件，如果无法执行需要检查源文件是否有执行权限，用chomod +x添加执行权限。添加完命令之后，服务器每次开机才会自动执行脚本命令将所需的服务开启。

### 三、管理账号

修改/etc/ppp/chap-secrets文件

```
[root@centos7 ~]# cat /etc/ppp/chap-secrets
# Secrets for authentication using CHAP
# client    server    secret    IP addresses
jacky    l2tpd    yourpassword       *

```

**

### 四、调整网络参数

修改 /etc/sysctl.conf 文件开启路由功能， vi /etc/sysctl.conf

将下面两项找到：

`net.ipv4.ip_forward = 0`

`net.ipv4.conf.default.rp_filter = 1`

改为：

`net.ipv4.ip_forward = 1`

`net.ipv4.conf.default.rp_filter = 0`

之后先让修改后的配置生效，
`sysctl -p`



**iptables修改**

```
root@centos7 ~]# iptables -t nat -A POSTROUTING -s 192.168.17.0/24 -o eth0 -j MASQUERADE
[root@centos7 ~]# iptables -I FORWARD -s 192.168.17.0/24 -j ACCEPT
[root@centos7 ~]# iptables -I FORWARD -d 192.168.17.0/24 -j ACCEPT
[root@centos7 ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             192.168.17.0/24     
ACCEPT     all  --  192.168.17.0/24      anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination  
```

将Iptables修改添加到配置文件中，/etc/rc.d/init.d/iptables

CentOS7默认没有这个文件，可以将iptables命令写到/etc/rc.local中开机执行。



UDP 500,1701是L2TP的默认通信端口,需要在防火墙，安全组策略这些地方允许其通过。





### 五、终端连接

#### 支持的终端

win7+win10自带vpn客户端，andriod自带vpn客户端等

#### 连接问题解决

1、win10系统 L2TP连接尝试失败:ERROR因为安全层在初始化与远程计算机的协商时遇到了一个处理错误

(1)创建 AllowL2TPWeakCrypto 注册表项，并将其设置为 1 的值。


        1. 单击开始，单击运行，键入regedit，然后单击确定
        2. 在注册表编辑器中，找到并单击以下注册表子项︰
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters

        3. 在编辑菜单上，指向新建，然后单击DWORD 值。
        4. 键入AllowL2TPWeakCrypto，然后按 enter 键。
        5. 在编辑菜单上，单击修改。
        6. 在数值数据框中，键入1，然后单击确定。
        7. 在文件菜单上，单击退出以退出注册表编辑器。
(2)ProhibitIpSec 注册表项，并将其设置为 0 的值
1.定位注册表HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Rasman\Parameters 
2. 在“编辑”菜单上，单击“新建”->“DWORD值” 
3. 在“名称”框中，键入“ProhibitIpSec” 
4. 在“数值数据”框中，键入“1”，然后单击“确定” 
  （ps: “ProhibitIpSec”=dword:00000000 ;使用RAS的L2TP功能[1=关闭]） 

重启服务：IPsec Policy Agent，Routing and Remote Access，Remote Access Connection Manager

2、L2TP 809错误,win10提示l2tp被防火墙等设备不允许通过

有三个服务需要启动（“Remote Access Auto Connection Manager”、“Remote Access Connection Manager”和“Secure Socket Tunneling Protocol Service”），
“Remote Access Auto Connection Manager”服务是手动的，如果未启动，改成自动启动并启动之，后再连接L2TP/IPSec的VPN



