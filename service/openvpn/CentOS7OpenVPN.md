# CentOS7 搭建OpenVPN

***



### 一、环境

我在云主机上租用了一台服务器，尝试搭建openvpn服务

**eth0:公网IPx.x.x.x**

**lo:127.0.0.1**

**操作系统：centOS 7.4**

**openvpn 2.4.7**

**安装EPEL仓库**

```
wget http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
```

### 二、安装

#### （一）、安装openvpn

**安装openvpn**

`yum install openvpn`

**安装openvpn最新的easy-rsa**

该包用来制作ca证书，服务端证书，客户端证书。
最新的为easy-rsa3。

```
wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
unzip master.zip
```

将解压得到的文件夹easy-rsa-master重命名为easy-rsa

```
mv easy-rsa-mater/ easy-rsa/
然后将easy-ras文件夹复制到/etc/openvpn/目录下
cp -R easy-rsa/ /etc/openvpn/
```

#### （二）、编辑vars

进入/etc/openvpn/easy-rsa/easyrsa3目录，复制vars.example为vars

`d /etc/openvpn/easy-rsa/easyrsa3/`

`cp vars.example vars`

修改下面字段，命令：vim vars，然后修改，最后wq保存

```
//以下字段根据自己实际情况更改，这些信息前面是有#注释的，去掉#号

set_var EASYRSA_REQ_COUNTRY “CN” #国家
set_var EASYRSA_REQ_PROVINCE “GuangDong” #省份
set_var EASYRSA_REQ_CITY “GuangZhou” #城市
set_var EASYRSA_REQ_ORG “tielemao” #非盈利组织，此处可填公司之类
set_var EASYRSA_REQ_EMAIL “wwz@tielemao.com” #邮箱地址
set_var EASYRSA_REQ_OU “My OpenVPN” #组织单元
```

这个vars文件似乎也不是很重要，不过填上一些信息也无不可？这是我填的example(这东西其实可以随便填)

```
set_var EASYRSA_REQ_COUNTRY     "CN"
set_var EASYRSA_REQ_PROVINCE    "Miramar"
set_var EASYRSA_REQ_CITY        "Pecado"
set_var EASYRSA_REQ_ORG         "4AM"
set_var EASYRSA_REQ_EMAIL       "jacky@pubg.com"
set_var EASYRSA_REQ_OU          "My OpenVPN"
```

#### （三）、创建服务器端证书及key

进入`/etc/openvpn/easy-rsa/easyrsa3/`目录初始化`./easyrsa init-pki`

```
[root@yunwei_OpenVPN easyrsa3]# ./easyrsa init-pki

Note: using Easy-RSA configuration from: ./vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/easyrsa3/pki
```

**创建根证书**

`./easyrsa build-ca`

如下图：

![img](https://www.tielemao.com/wp-content/uploads/2018/01/easyrsa_build-ca.jpg)

注意：在上述部分需要输入PEM密码 PEM pass phrase，输入两次，此密码必须记住，不然以后不能为证书签名。

还需要输入common name 通用名，自定义一个好记的。

生成的根证书文件为：`/etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt`

**创建服务器端证书**

`./easyrsa gen-req server nopass`

如下图：

![img](https://www.tielemao.com/wp-content/uploads/2018/01/%E5%88%9B%E5%BB%BAOpenVPN%E6%9C%8D%E5%8A%A1%E7%AB%AF%E8%AF%81%E4%B9%A6.jpg)

同样起个好记的通用名字，不过就不能和前面根证书的一样。

生成的文件有两个，注意这个时候这两个文件还不是服务端证书：

```
req: /etc/openvpn/easy-rsa/easyrsa3/pki/reqs/server.req
key: /etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key
```

**签约服务端证书：**

`./easyrsa sign server server`

注，这里前一个server是命令表示注册的是server端，后一个server是可以自行定义的名字，

但是要和前面命令起的名字一致，我这里一致都是server。

如下图：

![img](https://www.tielemao.com/wp-content/uploads/2018/01/%E7%AD%BE%E7%BA%A6OpenVPN%E6%9C%8D%E5%8A%A1%E7%AB%AF%E8%AF%81%E4%B9%A6.jpg)

要输入yes确认才能继续操作下去，
输入之前创建根证书的时候输入的PEM密码，如果忘记了就得从创建根证书重新做起了。
最终生成服务端的证书，crt格式：

`/etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt`

**创建Diffie-Hellman，确保key穿越不安全网络的命令：**

`./easyrsa gen-dh`

如下图：

![img](https://www.tielemao.com/wp-content/uploads/2018/01/%E5%88%9B%E5%BB%BAdh_pem%E7%A1%AE%E4%BF%9Dkey%E7%A9%BF%E8%B6%8A%E4%B8%8D%E5%AE%89%E5%85%A8%E7%BD%91%E7%BB%9C.jpg)

生成dh.pem文件：
`/etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem`

#### （四）、创建客户端证书

**新建client文件夹**

进入root目录新建client文件夹，文件夹可随意命名，然后拷贝前面解压得到的easy-ras文件夹到client文件夹,进入下列目录:

```
cd /root/
mkdir client && cd client
cp -R /root/easy-rsa/ client/
```

**注**：这里我是将之前下载的master.zip解压和命令了放置在root下了，所以路径是`/root/easy-rsa`。实验的时候要根据自己的实际情况操作。

`cd client/easy-rsa/easyrsa3/`

**初始化**

`./easyrsa init-pki`

**注：**其实和之前创建服务端证书前的操作无二，不同的是这次是将easy-rsa的目录放用户家目录下。

**创建客户端key及生成证书（记住和之前的操作一样生成时是自己输入的密码）**

./easyrsa gen-req client-wwz    //名字自己定义

```
[root@yunwei_OpenVPN easyrsa3]# ./easyrsa gen-req client-wwz
Generating a 2048 bit RSA private key
...........................+++...............................................................................+++
writing new private key to '/root/client/easy-rsa/easyrsa3/pki/private/client-wwz.key.fXcHCDk8k1'
Enter PEM pass phrase:

## Verifying - Enter PEM pass phrase:

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,

## If you enter '.', the field will be left blank.

Common Name (eg: your user, host, or server name) [client-wwz]:

Keypair and certificate request completed. Your files are:
req: /root/client/easy-rsa/easyrsa3/pki/reqs/client-wwz.req
key: /root/client/easy-rsa/easyrsa3/pki/private/client-wwz.key
```

将生成的client-wwz.req导入然后签约客户端证书

返回到/etc/openvpn/easy-rsa/easyrsa3/

`cd /etc/openvpn/easy-rsa/easyrsa3/`

导入req

`./root/client/easy-rsa/easyrsa3/pki/reqs/client-wwz.req client-wwz`

```
[root@yunwei_OpenVPN easyrsa3]# cd /etc/openvpn/easy-rsa/easyrsa3/
[root@yunwei_OpenVPN easyrsa3]# pwd
/etc/openvpn/easy-rsa/easyrsa3
[root@yunwei_OpenVPN easyrsa3]# ./easyrsa import-req /root/client/easy-rsa/easyrsa3/pki/reqs/client-wwz.req client-wwz

Note: using Easy-RSA configuration from: ./vars

The request has been successfully imported with a short name of: client-wwz

## You may now use this name to perform signing operations on this request.

签约证书
./easyrsa sign client client-wwz
//这里生成client所以必须为client，client-wwz要与之前导入名字一致
上面签约证书跟server类似，就不截图了，但是期间还是要输入CA的密码

------

[root@yunwei_OpenVPN easyrsa3]# ./easyrsa sign client client-wwz

Note: using Easy-RSA configuration from: ./vars

You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 3650 days:

subject=
commonName = client-wwz

Type the word 'yes' to continue, or any other input to abort.
Confirm request details: yes
Using configuration from ./openssl-easyrsa.cnf
Enter pass phrase for /etc/openvpn/easy-rsa/easyrsa3/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName :ASN.1 12:'client-wwz'
Certificate is to be certified until Nov 9 07:10:14 2027 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated

## Certificate created at: /etc/openvpn/easy-rsa/easyrsa3/pki/issued/client-wwz.crt
```

客户端的证书client-wwz.crt路径在`/etc/openvpn/easy-rsa/easyrsa3/pki/issued/client-wwz.crt`

**现在梳理一下上面都生成了些什么东西：**

服务端：`etc/openvpn/easy-rsa/`文件夹

```
/etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/reqs/server.req
/etc/openvpn/easy-rsa/easyrsa3/pki/reqs/client.req
/etc/openvpn/easy-rsa/easyrsa3/pki/private/ca.key
/etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key
/etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/issued/client-ww.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem
```

重要的是`pki/issued`目录下的服务端和客户端这两个证书

客户端：`root/client/easy-rsa`文件夹

```
/root/client/easy-rsa/easyrsa3/pki/private/clinet-wwz.key
/root/client/easy-rsa/easyrsa3/pki/reqs/client.req
//这个文件被我们导入到了服务端文件所以那里也有
```

#### （五）、将生成的证书放置到openvpn目录

拷贝证书等文件放入到相应位置。

将下列文件放到`/etc/openvpn/` 目录：

```
cp /etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem /etc/openvpn
```

将下列文件放到/root/client 目录下：

```
cp /etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt /root/client
cp /etc/openvpn/easy-rsa/easyrsa3/pki/issued/client-wwz.crt /root/client
cp /root/client/easy-rsa/easyrsa3/pki/private/client-wwz.key /root/client
```

![img](https://www.tielemao.com/wp-content/uploads/2018/09/openvpn.jpg)

#### （六）、编辑服务器端配置文件

由于我们是yum安装的openvpn。

所以相应在`/usr/share/doc/`下会有对应openvpn版本`/sample/sample-config-files`目录下会有一个`server.conf`文件，

将这个文件复制到`/etc/openvpn`：

`cp /usr/share/doc/openvpn-2.4.7/sample/sample-config-files/server.conf /etc/openvpn/`

然后修改配置`vi server.conf`如下：

```
local x.x.x.x            # 填自己openvpn服务器的 IP，默认侦听服务器上的所有ip，我这里填写服务器的公网IP
port 1194                     # 侦听端口，默认1194，可以根据自己的情况修改
proto udp                     # 端口协议，默认udp，也可以开启tcp方便映射转发。
dev tun                       # 默认创建一个路由IP隧道
ca /etc/openvpn/ca.crt        # 根证书,这里都要填写文件的绝对路径
cert /etc/openvpn/server.crt  # 证书
key /etc/openvpn/server.key   # 私钥文件/重要保密
dh /etc/openvpn/dh.pem

server 10.8.0.0 255.255.255.0 
# 设置服务器端模式，并提供一个VPN子网，以便于从中为客户端分配IP地址。
# 服务器自身会使用10.8.0.1这个ip。

ifconfig-pool-persist ipp.txt 
# 指定用于记录客户端和虚拟IP地址的关联关系的文件。
# 当重启OpenVPN时，再次连接的客户端将分配到与上一次分配相同的虚拟IP地址

push “route 192.168.0.0 255.255.0.0” 
# 推送路由信息到客户端，以允许客户端能够连接到服务器背后的其他私有子网。
# (简而言之，就是允许客户端访问VPN服务器自身所在的其他局域网)
# 记住，这些私有子网也要将OpenVPN客户端的地址池(10.66.72.0/255.255.255.0)反馈回OpenVPN服务器。

push "redirect-gateway def1 bypass-dhcp" 
# 启用该指令，所有客户端的默认网关都将重定向到VPN，这将导致诸如web浏览器、DNS查询等所有客户端流量都经过VPN。
# (为确保能正常工作，OpenVPN服务器所在计算机可能需要在TUN/TAP接口与以太网之间使用NAT或桥接技术进行连接，调整服务器端iptables设置)
# 如果需要客户端通过服务器端代理上网，需要配置此项

push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"
# 某些具体的Windows网络设置可以被推送到客户端，例如DNS或WINS服务器地址。
# 实际上可用于推送内网dns或阿里内网dns等。

keepalive 10 120 
# keepalive指令将导致类似于ping命令的消息被来回发送，以便于服务器端和客户端知道对方何时被关闭。
# 默认每10秒钟ping一次，如果120秒内都没有收到对方的回复，则表示远程连接已经关闭。比较频繁，建议改成30 240？

;comp-lzo 
# 在VPN连接上启用压缩。如果你在此处启用了该指令，那么也应该在每个客户端配置文件中启用它,不建议启用，用;注释掉他

max-clients 100 #默认最大客户端连接100，为安全可限到1或2。
# 持久化选项可以尽量避免访问那些在重启之后由于用户权限降低而无法访问的某些资源。

persist-key
persist-tun

status /etc/openvpn/openvpn-status.log # 状态日志,此处的#代表数字
# 为日志文件设置适当的冗余级别(0~9)。冗余级别越高，输出的信息越详细。
# 0 表示静默运行，只记录致命错误。
# 4 表示合理的常规用法。
# 5 和 6 可以帮助调试连接错误。
# 9 表示极度冗余，输出非常详细的日志信息。

;verb 3


explicit-exit-notify 1
#当服务器端重启后，通知客户端，以便其能够自动连接，推荐配置


```

**缓冲区大小设置**

编辑配置文件server.conf

```
sndbuf 262144  
rcvbuf 262144
;sndbuf 0
;rcvbuf 0
# OpenVPN还在用64KB的默认缓冲区大小,缓冲区过小也有可能降低速度。一般将大小设置为0，系统会自动调整缓冲区大小，如果你把缓冲区调成0后还是慢，你应该要么在系统层面增加缓冲区大小，例如设置为262144
# 在服务器端设置了缓冲区大小后，在客户端的配置文件上需要进行相同的设置，如果客户端不支持设置，需要从服务器端推送设置如下：
push "sndbuf 262144"
push "rcvbuf 262144”

```

**mtu设置**

默认的mtu值为1500，调整了缓冲区大小后，有些udp数据包过大，可能在传输过程中产生分割，导致数据包无法识别报错，如果发生这类问题，可能是由于mtu原因引起，需要在配置文件增加如下设置：

```
mssfix 0
tun-mtu 1500    
txqueuelen 1000
```

**注意：**如需要检测链路中的最小mtu，可在配置文件配置mtu-test选项，直接增加一行mtu-test即可

**多客户端设置**

如果你需要多个客户端使用相同的证书进行认证的话需要在配置文件添加额外的配置

```
duplicate-cn

```

**Tls加密**

如果需要额外的Tls加密，复制ta.key文件到openvpn目录下并在配置文件增加如下配置

```
tls-auth /etc/openvpn/ta.key 0
#客户端和服务器端都需要有对应的配置
cipher AES-256-CBC
# 选择加密算法
```

**log配置**

需要保存日志，同样，将日志文件复制到openvpn目录下，并在配置文件指明其绝对路径

```
log        /etc/openvpn/openvpn.log
log-append  /etc/openvpn/openvpn.log

```

### 三、运行

**启用转发：**

`echo 1 > /proc/sys/net/ipv4/ip_forward` 或直接vim /proc/sys/net/ipv4/ip_forward 修改值为1

**调整iptables:**

```
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables -I FORWARD -s 10.8.0.0/24 -j ACCEPT
iptables -I FORWARD -d 10.8.0.0/24 -j ACCEPT
```

将vpn子网ip设置为允许通过eth0上网，nat伪装。这里的ip地址根据自己的设置进行修改

允许vpn子网的源和目的数据包通过iptables

**启动openvpn**

前台启动（当前shell退出后，进程终止）：

`openvpn --config /etc/openvpn/server.conf`

**注意：**如果为了方便实时监测openvpn连接和启动状态的时候，可以使用前台启动运行，但是当前shell结束后，进程会被终止

**后台启动（推荐使用）**：

`openvpn --daemon --config /etc/openvpn/server.conf`

服务调试运行稳定后，建议使用此命令；

**注意：**将上述命令都加入到etc/rc.local中开机运行，如发现运行不了，请检查rc.local所指向的文件是否具有可执行权限

```
 ~]# cat /etc/rc.local 
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables -I FORWARD -s 10.8.0.0/24 -j ACCEPT
iptables -I FORWARD -d 10.8.0.0/24 -j ACCEPT
openvpn --daemon --config /etc/openvpn/server.conf
echo 1 > /proc/sys/net/ipv4/ip_forward

```

### 四、客户端设置

#### （一）、windows客户端

将在openvpn服务器生成的根证书，客户端证书和key下载到客户端电脑。

`ca.crt `

`client-wwz.crt `

`client-wwz.key `		//示例中为这三个文件

如果使用了tls,ta.key文件也要复制过去

去官网下载openvpn客户端进行安装，然后安装目录找到`simple-config`

`D:\program\OpenVPN\sample-config\client.ovpn`

安装路径最好不要有空格字符，这样可能导致后面在配置文件中的路径无法被客户端识别

将client.ovpn 复制到`D:\program\OpenVPN\ToCloud`（文件夹可以自己新建定义）下，根据自己实际安装情况选择.
将下载到的三个文件放入`D:\program\OpenVPN\ToCloud`下然后编辑`client.ovpn`配置文件：

编辑配置文件：

```
client
dev tun
proto udp
remote 172.16.1.128 1194 //主要这里修改成openvpn服务器的ip
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt 
// 这里需要证书，之前和配置文件放同一文件夹下了，理论上不需要敲绝对路径也能找到，
// 然而后面发现客户端导入配置文件后目录又是在另外路径下，以致还是要敲绝对路径。
cert client-wwz.crt
key client-wwz.key
comp-lzo
verb 3
```

简单应用的话我们只需要以上项目每行一个，复杂些应用的话可以参照官方具体配置文档。根据服务器端配置文件进行相应调整

**注意：**上面的路径都要写绝对路径，

```
ca D:\\program\\OpenVPN\\ToCloud\\ca.crt
cert D:\\programs\\OpenVPN\\ToCloud\\client-wwz.crt
key D:\\programs\\OpenVPN\\ToCloud\\client-wwz.key
```

如果配置了Tls，也要相应进行修改

打开openvpn客户端，右键弹出菜单，选择导入配置文件

![img](https://www.tielemao.com/wp-content/uploads/2018/01/openvpn%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%AF%BC%E5%85%A5%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.jpg)

选中D:\program\OpenVPN\ToCloud下的配置文件导入，

![img](https://www.tielemao.com/wp-content/uploads/2018/01/openvpn%E5%AE%A2%E6%88%B7%E7%AB%AF%E8%BF%9E%E6%8E%A5%E6%9C%8D%E5%8A%A1%E7%AB%AF.jpg)

连接成功后需要输入之前我们设置的客户端证书密码，进行连接。

![img](https://www.tielemao.com/wp-content/uploads/2018/01/openvpn%E8%BF%9E%E6%8E%A5%E4%B8%8A%E4%B8%94%E8%A6%81%E6%B1%82%E8%BE%93%E5%85%A5%E8%AF%81%E4%B9%A6%E5%AF%86%E7%A0%81.jpg)

输完之后连接成功。

**注意：**

可以通过tail -f /etc/openvpn/openvpn.log 监控日志排错

也可以根据需要查看客户端的日志。

#### （二）、移动客户端

**安卓**

1、安卓客户端可以在googleplay商店中下载openvpn程序

2、将电脑端的配置文件和证书直接复制到手机中，如果使用了tls,ta.key文件也要复制过去

3、安装客户端，打开openvpn，通过导入找到配置文件和根证书，客户端证书和key文件导入

4、验证密码后，可以进行正常连接了

**IOS**

1、在苹果商店安装openvpn客户端

2、通过itunes导入openvpn所需的配置文件和证书及key

3、或者通过邮件发送

4、根据客户端提示简单修改配置即可连接

