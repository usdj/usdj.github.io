---
layout: post
title:  "Linux中网络那些事"
categories: Linux
tags: Linux Network
author: DJY
---

* content
{:toc}
## Linux中的网络
### 网元概念

**Bridge**

bridge 设备负责交换, Bridge（桥）是 Linux 上用来做 TCP/IP 二层协议交换的设备，与现实世界中的交换机功能相似。Bridge 设备实例可以和 Linux 上其他网络设备实例连接，既 attach 一个从设备，类似于在现实世界中的交换机和一个用户终端之间连接一根网线。当有数据到达时，Bridge 会根据报文中的 MAC 信息进行广播、转发、丢弃处理。

bridge 其中有一个区别是，linux 下的 bridge 设备本身有一个 IP，这个 IP 原本是这个 bridge 设备对外连接的网络接口的 IP（如 eth0），但是这个接口连接到 bridge 设备之后，其本身的 IP 将失去作用，通过这个 IP 访问到的 bridge 设备。

需要注意的是，bridge 对外连接的接口只能有一个，如果有多个，将会导致概率性丢高和极高的时延。 

**VLAN**
vlan 设备负责隔离，用来划分网络和网络上的主机，不同 vlan 中的主机无法直接通信。

vlan 设备分子母，如 eth0 为母设备，eth0.666 是子设备，母设备只能接收数据，子设备只能发送数据。

VLAN 又称虚拟网络，是一个被广泛使用的概念，有些应用程序把自己的内部网络也称为 VLAN。此处主要说的是在物理世界中存在的，需要协议支持的 VLAN。它的种类很多，按照协议原理一般分为：MACVLAN、802.1.q VLAN、802.1.qbg VLAN、802.1.qbh VLAN。其中出现较早，应用广泛并且比较成熟的是 802.1.q VLAN，其基本原理是在二层协议里插入额外的 VLAN 协议数据（称为 802.1.q VLAN Tag)，同时保持和传统二层设备的兼容性。Linux 里的 VLAN 设备是对 802.1.q 协议的一种内部软件实现，模拟现实世界中的 802.1.q 交换机。

vlan 的报文有两种形式，有标签的和无标签的。无标签的 vlan 报文就是一个普通报文，这些无标签的 vlan 报文由交换机决定它们属于哪些 vlan，交换机可以将接口配置给指定的 vlan。 
     
如果交换机收到一个有标签的报文并且接受该报文的端口被配置为允许有标签的报文通过，交换机可以知道需要将这些报文送到哪些端口。 
     
每个 vlan 都会给配置一个 id，这个 id 可以是 1-4094 中的任意数字。vlan 1 一般被用来管理所以在使用的时候不应该使用这个 id。

ps: 
只能使用物理接口来使用 vlan，虚拟接口（如 eth0:1）不可用。

**tap/tun**

针对 TAP 设备的一个形象的比喻是：使用 TAP 设备的应用程序相当于另外一台计算机，TAP 设备是本机的一个网卡，他们之间相互连接。应用程序通过 read()/write()操作，和本机网络核心进行通讯。

TUN/TAP 设备是一种让用户态程序向内核协议栈注入数据的设备，一个工作在三层，一个工作在二层，使用较多的是 TAP 设备。

**veth**
VETH 设备总是成对出现，送到一端请求发送的数据总是从另一端以请求接受的形式出现。该设备不能被用户程序直接操作，但使用起来比较简单。创建并配置正确后，向其一端输入数据，VETH 会改变数据的方向并将其送入内核网络核心，完成数据的注入。在另一端能读到此数据。可以将两个 VETH 设备理解为一条网线的两端，事实上系统中两个 bridge 可以使用 VETH 进行连接。

VETH 设备出现较早，它的作用是反转通讯数据的方向，需要发送的数据会被转换成需要收到的数据重新送入内核网络层进行处理，从而间接的完成数据的注入。

### 用法
**Bridge** 
```
brctl show
brctl addbr br2
    -- 创建 bridge 设备 br2
brctl delbr br2
    -- 删除 bridge 设备 br2
brctl addif br2 em1
    -- 将 em1 接入到 br2
brctl delif br2 em1
    -- 将 em1 从 br2 移除
brctl showmacs br1
    -- 查看 mac 表
brctl setportprio br2 vnet2 1
    -- 设置 port 优先级
```

**VLAN**
ubuntu/debian中：
```
apt-get install vlan
modprobe 8021q
vconfig add eth1 10
ip addr add 10.0.0.1/24 dev eth1.10
ip link set up eth1.10
```
配置持久化
```
设置开机加载 8021q 模块
su -c 'echo "8021q" >> /etc/modules'
在系统启动时创建接口并使其可用，向 /etc/network/interfaces 中添加：
auto eth1.10
iface eth1.10 inet static
    address 10.0.0.1
    netmask 255.255.255.0
    vlan-raw-device eth1
```
其他相关操作：
```
ip link delete eth1.10
vconfig rem br2.0
    -- 删除 vlan br2.0
cat /proc/net/vlan/xxx
    -- 查看 vlan 当前配置
```
**TAP** 
安装 tap 设备管理工具
```
apt-get install uml-utilities
```
手动添加/删除 tap 设备
```
tunctl -t tap0 -u root
     -- 创建一个 root 所有的虚拟网卡 tap0
tunctl -d tap0
     -- 删除虚拟网卡 tap0
```
自动添加 tap 设备
     开机自动创建一个 ip 为 11.11.11.10/24 的 tap0 虚拟网卡
```
 vi /etc/network/interfaces
auto tap0
iface tap0 inet manual
        up ifconfig $IFACE 11.11.11.10/24 up
        down ifconfig $IFACE down
        tunctl_user root
```
具体工作原理查看 /etc/network/if-*

**VETH**
管理命令
```
ip link add veth0 type veth peer name veth1
    -- 创建一对 veth 设备，一端是 veth0，另一端是 veth1
ip link set veth1 netns net0
    -- 将 veth1 连接到 namespace net0，veth1 的另一端 veth0 默认连接在创建 veth 时的 namespace，也可以用来连接到其它设备，如 bridge。
ethtool -S veth0
    -- 查看 veth2 的对端是哪个端口，端口可以通过 ip link show 查询
ip link add veth-a type veth peer name veth-b
ovs-vsctl add-port br0 veth-a
ovs-vsctl add-port br1 veth-b
    -- 创建一对 veth 接口 veth-a 和 veth-b，使用它们来连接网桥 br0 和 br1
```

**Namespace**
一个网络命名空间 namespace 内可以有多个各种各样的接口，包括物理接口，loopback 回环接口，桥接口（bridge），vlan 接口，虚拟接口，tap 接口，tun 接口，veth 接口等。
在同一个网络命名空间内的各个接口是可以经由内核直接通信的。
Linux 操作系统有一个默认的不具名网络命名空间，直接用 ifconfig 或 ip link show 查看到的那些网络设备就是这个命名空间下的设备，可以通过创建多个具名网络命名空间来起到网络隔离的效果。

### 三张表
- MAC表：负责映射端口和 mac 地址，只进行本交换机的信息管理
- ARP表：负责 mac 地址和 ip 地址的映射
- 路由表：负责维护局域网和局域网之间的关系映射，网关是局域网的入口，路由表中维护了网关（局域网入口）和网段（局域网）的映射关系

**MAC表**
mac 表中保存了 mac 地址和 端口的对应关系，在二层网络设备（如交换机）中使用。
交换机根据 mac 地址表转发数据帧，在交换机中，维护一张记录了局域网主机的 mac 地址和交换机接口的对应关系的 mac 表，交换机根据这张表负责将数据帧传输到指定的主机。
  mac表更新
  当有数据帧到达的时候，交换机会查询当前的 mac 表，如果在当前的表中已经。保存了 src mac 的端口，则不更新 mac 表，否则更新 mac 表，将 src mac 和该数据帧进来时的端口进行映射。
  mac 表使用
  当有数据帧到达的时候，交换机会看数据帧中的 dst mac 地址，然后查询 mac 表，如果能找到，则将该数据帧从 dst mac 所对应的端口发送出去；如果找不到，则将该数据帧广播出去。

**APR表**
arp 表中保存了 mac 地址和 ip 地址的映射关系，在第三层网络中应用。

主机在发送数据帧的时候，根据 dst ip 地址查询 mac 地址，如果能查到，则将 dst ip 对应的 mac 地址 写入数据帧并发送，否则在发送数据帧之前先发送 arp 报文以获取 dst ip 的 mac 地址。

在每台主机中都有一张ARP表，它记录着主机的IP地址和MAC地址的对应关系。

ARP协议：ARP协议是工作在网络层的协议，它负责将IP地址解析为MAC地址。

**路由表**
路由表应用在网络层，路由器根据路由表确定报文的下一跳。 
当需要发送网络报文的时候，查找路由表，确定网络报文中的 dst ip 所在的网段的下一跳，如果有，则直接发送给下一跳，如果没有，则发送给默认网关。

### 配置工具
ifconfig
     网络设备管理软件
eg:
```
列出活跃的网络接口
ifconfig
列出所有的网络接口
ifconfig -a
设置接口的 ip 和广播
ifconfig em1 172.0.10.155/24 broadcast 172.0.10.255
设置 ether 设备硬件地址（mac 地址）
ifconfig em1 hw ether f8:bc:12:0d:17:40
添加 ipv6 地址
ifconfig eth0 add fc00::2/8
删除接口上的 IP
ifconfig eth1 0.0.0.0
修改接口的 mac 地址（临时修改，重启失效）
ifconfig eth0 hw ether 00:1b:21:ba:ae:81
```

配置文件
ubuntu/debian
```
cat /etc/network/interfaces
auto em1
iface em1 inet static
        address 172.0.10.155
        netmask 255.255.255.0
        network 172.0.10.0
        broadcast 172.0.10.255
        gateway 172.0.10.1
        # dns-* options are implemented by the resolvconf package, if installed
        dns-nameservers 114.114.114.114
auto wlan1
iface wlan1 inet static
    wpa-ssid Router
    wpa-psk 9012345678
    address 192.168.0.7
    netmask 255.255.255.0
    gateway 192.168.0.1
    dns-nameservers 114.114.114.114
```

rhel
```
cat /etc/sysconfig/network-scripts/ifcfg-em1
TYPE=Ethernet
BOOTPROTO=static
default via 172.0.1.1 dev em1
IPADDR=172.0.1.14
IP_GATEWAY=172.0.1.1
NETMASK=255.255.255.0
DNS1=114.114.114.114
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=em1
UUID=4f150533-b098-4ffb-b382-ea10869186ad
DEVICE=em1
ONBOOT=yes
```
centos 6.5
```
[root@centos ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
HWADDR=52:54:00:7c:01:39
TYPE=Ethernet
UUID=b9a4a8a9-2094-4bd9-a1c3-f73b7ca39b94
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=static
IPADDR=172.0.11.91
NETMASK=255.255.255.0
DNS1=8.8.8.8
GATEWAY=172.0.11.1
```

ifup / ifdown
根据系统配置启用 / 关闭网络接口，ubuntu 下为 /etc/network/interfaces，rhel 下为 /etc/sysconfig/network-scripts/ifcfg-em1
```
ifup em1
ifdown em1
```

ethtool
```
ethtool em1
    -- 查看 em1 的链接信息
ethtool -i em1
    -- 查看 em1 的接口信息，包括网口驱动（通过驱动可以区分网口类型，比如说 bridge，tun ），firmware 等
ethtool -S veth2
    -- 查看 veth2 的对端是哪个端口，端口可以通过 ip link show 查询
```
ping/ping6
ping6 是 ping 的 ipv6 版本。
```
ping 172.0.5.75
发起 flood ping
ping -f 172.0.5.75
设置 ping 包大小为 1450
ping -s 1450 172.0.5.75
指定接口 ping（可以 ping ipv6 local link）
ping6 -I eth0 fe80::5054:ff:fee4:6342
local-link 如果不指定端口的话，需要在 IP 后面用 % 指定设备（local-link 无法添加路由）
ping6 fe80::5054:ff:fee4:6342%eth0
```
route
     路由管理
```
route -n
route add -net default gw 172.0.5.1
route add -net 10.8.0.0/16 gw 172.1.1.123
route add 172.2.1.120/32 gw 172.1.1.120 -- netmask位32的时候不需要加-net选项，其余都要加
route add -net default dev eth0
```

ipv6
```
route -A inet6 -n
route -A inet6 add default dev eth0
```
ip
 网络管理工具

help
```
ip help
ip addr help
ip route help
```
ip link
```
ip link show
ip link set interface up
ip link set eth0 up
ip link delete eth0 
ip link add link eth5 eth5.80 type vlan id 80
    -- 根据 eth5 为母设备创建一个 eth5.80 的子设备，vlan id 设置为 80
ip link add veth0 type veth peer name veth_red
    -- 创建一对 veth 设备，一端是 veth0，另一端是 veth_red
ip link set veth0 netns red
    -- 将 veth0 连接到 namespace red，veth0 的另一端 veth_red 可以用来连接到其它设备，如 bridge。
ip -s link show eth0
    -- 显示 eth0 上的收发统计信息
ip -s -s link show eth0
    -- 显示 eth0 上面更详细的收发统计信息
```
eg
```
ip -s link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:ca:1d:08 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    347857468  1338388  7281    0       0       0      
    TX: bytes  packets  errors  dropped carrier collsns 
    130933755  308234   0       0       0       0 
```
```
ip -s -s link show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:ca:1d:08 brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    347916130  1338856  7283    0       0       0      
    RX errors: length  crc     frame   fifo    missed
               7283     0       0       0       0      
    TX: bytes  packets  errors  dropped carrier collsns 
    130944351  308288   0       0       0       0      
    TX errors: aborted fifo    window  heartbeat
               0        0       0       0  
```
ip addr
```
ip addr add 192.168.1.6/24 dev wlan0
    -- 设置 wlan0 的 ip
ip addr del 172.1.10.159/16 dev eth1
    -- 删除 eth1 上的指定 ip
```
ip route
```
ip route add default via 172.0.1.1 dev em1
ip route list 
ip route list table main
ip route add 0/0 via 192.168.0.4 table main
ip route add 192.168.3.0/24 via192.168.0.3 table 1
```
ipv6:
```
ip -6 route list
ip -6 route add default dev em1
```
ip rule
　　策略路由是一种比基于目标网络进行路由更加灵活的数据包路由转发机制。

　　Linux最多可以支持255张路由表，其中有3张表是内置的：存在文件/etc/iproute2/rt_tables中

　　表255 本地路由表(Local table) 本地接口地址，广播地址，已及NAT地址都放在这个表。该路由表由系统自动维护，管理员不能直接修改。

　　表254 主路由表(Main table) 如果没有指明路由所属的表，所有的路由都默认都放在这个表里，一般来说，旧的路由工具(如route)所添加的路由都会加到这个表。一般是普通的路由。

　　表253 默认路由表(Default table)一般来说默认的路由都放在这张表，但是如果特别指明放的也可以是所有的网关路由。

　　表0 保留 
在Linux里，总共可以定义232个优先级的规则，一个优先级别只能有一条规则，即理论上总共可以有232条规则。其中有3个规则是默认的。路由规则数字越小，优先级越高
```
ip rule
ip rule add [from 0/0] table 1 pref 32800
    -- 将向规则链增加一条规则，规则匹配的对象是所有的数据包，动作是选用路由表1的路由，这条规则的优先级是32800
ip rule add from 192.168.3.112/32 [tos 0x10] table 2 pref 1500 prohibit
    -- 向规则链增加一条规则，规则匹配的对象是IP为192.168.3.112, tos等于0x10的包，使用路由表2,这条规则的优先级是1500,动作是丢弃
```
　　上面的规则是以源地址为关键字，作为是否匹配的依据的。除了源地址外，还可以用以下的信息：

　　From – 源地址

　　To – 目的地址(这里是选择规则时使用，查找路由表时也使用)

　　Tos – IP包头的TOS(type of sevice)域

　　Dev – 物理接口

　　Fwmark – 防火墙参数

　　采取的动作除了指定表，还可以指定下面的动作：

　　Table 指明所使用的表

　　Nat 透明网关

　　Action：

　　prohibit 丢弃该包，并发送 COMM.ADM.PROHIITED的ICMP信息

　　Reject 单纯丢弃该包

　　Unreachable丢弃该包，并发送 NET UNREACHABLE的ICMP信息 
　　

ip netns
netns：network namespace，网络命名空间管理，可以创建完全隔离的网络命名空间，以此实现 vrf。
```
ip netns add net0
    -- 创建一个名为 net0 的网络命名空间
ip netns list
    -- 查看创建的网络命名空间
ip netns exec net0 bash
    -- 在 net0 的网络命名空间中执行 bash
ip netns exec net0 ifconfig
    -- 在 net0 的网络命名空间中执行 ifconfig
```
ip tunnel
tunnel 设备管理
`ip tunnel show`
ip tuntap
tap 设备管理

`ip tuntap show`
iptables
基本操作
```
iptables -F
     -- 清空当前所有的 iptables 规则（默认规则，也即 policy 除外）
iptables -nL
     -- 查看当前配置的 iptables 规则
iptables -nL --line-number
     -- 显示每条规则的序号（在删除规则的时候有用到）
iptables -A INPUT -s 172.0.5.75 -p tcp --dport 46989 -j ACCEPT
     -- 允许 172.0.5.75 这个 IP 通过 tcp 协议的 46989 端口连接到本地
iptables -A OUTPUT -p tcp -j ACCEPT
     -- 开放所有的 tcp 出口端口
iptables -P INPUT DROP    
     -- 设置入口默认为 DROP
iptables -P OUTPUT DROP
     -- 设置出口默认为 DROP
iptables -P FORWARD DROP
     -- 设置转发默认为 DROP
iptables -D INPUT 1
     -- 删除 INPUT 链中的第一条规则
iptables -t nat -D POSTROUTING 1
     -- 删除 nat 的 POSTROUTING 链中的第一条规则
```
保存 iptables 规则以便重启生效
```
iptables-save > /etc/iptables-rules
echo "pre-up iptables-restore < /etc/iptables-rules" >> /etc/network/interfaces
```
清空 iptabels 配置
```
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
```
NAT
SNAT
SNAT 是常见的 NAT 配置方式，主要用来实现多个内网用户使用同一个公网 IP 上网。SNAT 会修改发出去的包的源地址，不修改目的地址，并且建立表项，当收到回来的包时，根据表项分发到真正发包的 IP。

使用一个内网的 IP 给一个外网的 IP 发包，外网 IP 即使可以收到这个 IP 并正确处理，往内网 IP 发了一个包，但由于外网 IP 根本不知道怎么到达这个内网 IP，所以无法正确将包发到这个内网 IP。而使用 SNAT 之后，则相当于内网的那些 IP 可以使用一个共同的外网 IP 作为代理，实现正确的收发包流程。

SNAT 是在 iptables 的 postroutine 上实现的。

配置： 
确认 linux 内核的 IP 转发已开启，输出为 1 则为已开启，否则需要开启
`cat /proc/sys/net/ipv4/ip_forward`
开启 ip_forward，下面的操作重启失效，如果要永久有效，将 “net.ipv4.ip_forward=1” 选项添加到 /etc/sysctl.d/10-network-security.conf。
`echo 1 > /proc/sys/net/ipv4/ip_forward`
配置一条 SNAT 规则，所有源地址为 10.20.0.0/16 的包，发出去之前都将源地址修改为 172.0.11.55
`iptables -t nat -A POSTROUTING -s 10.20.0.0/16 -j SNAT --to-source 172.0.11.55`
查看配置结果
`iptables -nL -t nat`
DNAT
DNAT 和 SNAT 相反，DNAT 会修改包的目的地址，但不修改源地址，一般用来实现在一个 IP 上运行不同的服务。

在 DNAT 的基础上，还可以修改目的端口，称之为 PNAT。

DNAT 是在 iptables 的 preroutine 链上实现的。

几个示例
```
iptables -t nat -A PREROUTING -d 10.0.0.1 -j DNAT –-to-destination 172.0.5.75
    -- 如果收到目的地址为 10.0.0.1 的 IP 包，则将其目的地址修改为 172.0.5.75
iptables -t nat -A PREROUTING -d 10.0.0.1 -p tcp –-dport 80 -j DNAT –-to-destination 172.0.5.75
    -- 如果收到目的地址为 10.0.0.1，目的端口为 80 的 tcp 包，则将其目的地址修改 172.0.5.75
iptables -t nat -A PREROUTING -d 10.0.0.1 -p tcp –-dport 80 -j DNAT –-to-destination 172.16.93.1:8080
    -- 如果收到目的地址为 10.0.0.1，目的端口为 80 的 tcp 包，则将其目的地址修改 172.16.93.1，目的端口修改为 8080
```
brctl
     bridge 设备管理
```
brctl show
brctl addbr br2
    -- 创建 bridge 设备 br2
brctl delbr br2
    -- 删除 bridge 设备 br2
brctl addif br2 em1
    -- 将 em1 接入到 br2
brctl delif br2 em1
    -- 将 em1 从 br2 移除
brctl showmacs br1
    -- 查看 mac 表
brctl setportprio br2 vnet2 1
    -- 设置 port 优先级
```
bridge 上的 mac 表说明
```
port no mac addr is local? ageing timer
1 00:1b:21:26:02:88 no 0.89
1 86:d3:02:9d:37:5c no 0.68
1 a0:36:9f:81:27:11 no 0.74
1 f8:bc:12:0d:17:40 yes 0.00
1 f8:bc:12:0d:17:40 yes 0.00
2 fe:54:00:18:5e:88 yes 0.00
2 fe:54:00:18:5e:88 yes 0.00
```
说明： 
     当 src mac 为 86:d3:02:9d:37:5c 的报文从 port 1（port 1 只有 f8:bc:12:0d:17:40 是本地连接的 - is local）到达 bridge 的时候，arp 协议会自动记录，以后只要 dst mac 为 86:d3:02:9d:37:5c 的报文到达，就将报文从 port 1 发出去。



netstat
     netstat 可以查看本地 socket 相关的信息。

```
netstat
    -- 显示所有连接（connected）的 socket
netstat -p
    -- 显示 socket 的进程号
netstat -a
    -- 显示所有的 socket，默认显示的是 connected 的 socket
netstat -n
    -- 不解析主机名
netstat -l
    -- 显示所有监听的 socket
netstat -s
    -- 显示网络统计信息，按照类似 snmp 的方式显示，以协议分类
netstat -r
    显示默认路由表，效果类似于 route -n
netstat -i
    显示所有的接口信息
```
vconfig
     vlan 设备管理工具

命令说明
- 创建 VLAN 设备：vconfig add [PARENT DEVICE NAME] [VLAN ID]
- 删除 VLAN 设备：vconfig rem [VLAN DEVICE NAME]
- 设置 VLAN 设备 flag：vconfig set_flag [VLAN DEVICE NAME] [FLAG] [VALUE]
- 设置 VLAN 设备 qos： 
vconfig set_egress_map [VLAN DEVICE NAME] [SKB_PRIORITY] [VLAN_QOS] 
vconfig set_ingress_map [VLAN DEVICE NAME] [SKB_PRIORITY] [VLAN_QOS]
e.g
```
vconfig add em1 666
    -- 向 id 为 666 的 vlan 中添加 em1
vconfig rem em1.666
    -- 删除 id 为 666 的 vlan 上的 em1.666 子设备
```
arp
     三层的 arp 表管理，也即 ip 和 mac 地址的映射管理工具。
```
arp -n
    -- 查看 arp 表，并且不要解析主机名，可以加快速度
arp -s -i em1 172.0.5.5 f8:bc:12:0d:17:60
    -- 设置静态 arp 表，指定 ip 172.0.5.5 的 mac 地址并指定从 em1 口出去
```
nmblookup
nmblookup 在 samba 这个包中
`apt-get install -y samba`
常用命令
```
nmblookup -A IP
     -- 查看该IP的主机名
```
nslookup
     nslookup 用来查询域名与 IP 之间的对应关系

```
root@vm1:~# nslookup 111.13.100.92
Server:     114.114.114.114
Address:    114.114.114.114#53
Non-authoritative answer:
*** Can\'t find 92.100.13.111.in-addr.arpa.: No answer
Authoritative answers can be found from:
100.13.111.in-addr.arpa nameserver = dns01.bj.chinamobile.com.
100.13.111.in-addr.arpa nameserver = dns02.bj.chinamobile.com.
dns01.bj.chinamobile.com    internet address = 211.136.25.4
dns02.bj.chinamobile.com    internet address = 221.130.33.132
```
host
     host命令用来做DNS查询。如果命令参数是域名，命令会输出关联的IP;如果命令参数是IP，命令则输出关联的域名。
```
root@ltepc7:/public# host 172.0.10.155
155.10.0.172.in-addr.arpa domain name pointer 172-0-10-155.lightspeed.wlfrct.sbcglobal.net.
root@ltepc7:/public# host www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 111.13.100.92
root@ltepc7:/public# host 8.8.8.8
8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.
```
whois
     whois命令输出指定站点的whois记录，可以查看到更多如谁注册和持有这个站点这样的信息。
`whois www.taobao.com`
wpa_supplicant
     wpa_supplicant 是命令行下的 wifi 工具，可以用来配置和连接 wifi，其中 wpa_supplicant 是服务器端，wpa_cli 是客户端。可以直接用 wpa_supplicant 来连接 wifi，也可以通过 wpa_cli 来进入一个交互界面进行配置。

     在 wpa_cli 中，需要先通过 add_network 来添加一个网络，然后用 set_network id_num 来进行配置，最后通过 enable_network id_num 来启用网络，启用网络成功之后，还需要用 dhclient wlan1 来获取动态 IP 或手动配置静态 IP。这里的配置是全局性的而非针对某个特定接口。

通过 wpa_cli 来配置连接 wpa-psk 或 wpa2-psk 的 wifi 热点：

```
wpa_cli
 add_network -- 默认第一个返回的网络是 0，之后递增
 set_network 0 ssid "Router"
 set_network 0 psk "password"
 enable_network 0
 quit
dhclient wlan1
```
curl


iw
     无线端的 ip 工具



iwconfig
     无线设备的网络设备管理软件



iwlist
     无线信号扫描工具
### 链路检查
**ping**
ping 使用 icmp 包检查是否对端主机是否可达。
```
ping 172.0.5.75
    -- ping 172.0.5.75，检查主机是否能正行通信（通过 IP）。
ping -I eth0 172.0.5.75
    -- 使用指定接口 eth0 收发 ping 包
ping -f 172.0.5.75
    -- flood ping。玩命儿去 ping~
```

**hping3**
hping3 可以使用 rawip, icmp，tcp，udp 等多种种报文来 ping 包，默认为 tcp。

也可以用来扫描网络等~
```
hping3 172.0.5.75
    -- 使用 tcp ping 主机
hping3 -0 172.0.5.75
    -- 使用 rawip ping 主机
hping3 -1 172.0.5.75
    -- 使用 icmp ping 主机
hping3 -2 172.0.5.75
    -- 使用 udp ping 主机
hping3 --flood 172.0.5.75
    -- flood ping。玩命儿去 ping~
```
**arping**
arping 使用 arp 报文来 ping 主机，可以直接使用 mac 地址作为目标，也可以使用 ip 地址作为目标，但无论是使用 mac 地址还是 ip 地址作为目标，arping 实际收发使用的都是 arp 包。

由于使用的是 arp 报文，所以如果是跨网段去 ping 的话，会返回网关的 IP 和 mac 地址。
```
arping -i br0 172.0.5.75
arping -i br0 bc:16:65:ef:2d:4b
```
**tracepath & traceroute**
tracepath命令和traceroute命令功能类似，但不需要root权限。并且Ubuntu预装了这个命令，traceroute命令没有预装的。tracepath追踪出到指定的目的地址的网络路径，并给出在路径上的每一跳(hop)。如果你的网络有问题或是慢了，tracepath可以查出网络在哪里断了或是慢了。
**mtr**
mtr命令把ping命令和tracepath命令合成了一个。mtr会持续发包，并显示每一跳ping所用的时间,也会显示过程中的任何问题.
```
mtr 172.0.10.155
                                                      My traceroute  [v0.85]
ltepc7 (0.0.0.0)                                                                                          Mon Aug 29 10:13:57 2016
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                                                                          Packets               Pings
 Host                                                                                   Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. 172-0-5-1.lightspeed.brhmal.sbcglobal.net                                            0.0%     5    0.6   1.6   0.6   5.8   2.2
 2. 172-0-10-155.lightspeed.wlfrct.sbcglobal.net                                         0.0%     4    0.4   0.4   0.3   0.4   0.0
```
### 网络扫描
**arp-scan**
使用 arp 进行网络扫描的工具。

由于是根据 arp 进行扫描，所以不能跨交换机扫描，比如说不能使用一个 172.0.5.x/24 网段交换机下的一个接口去扫描 172.0.10.0/24 网段，即使它们之间互通，因为对于 172.0.5.0/24 网段上的主机来说，所有 172.0.10.0 网段返回的 arp 都会被交换机改为两个交换机之间互联的那个接口的 mac 地址。

```
arp-scan --interface br0 172.0.5.0/24
...
172.0.11.241    52:54:00:ab:9b:c1   QEMU
172.0.11.243    52:54:00:3d:1f:0f   QEMU
172.0.11.245    2c:4d:54:44:9d:3f   (Unknown)
172.0.11.246    52:54:00:92:c3:2d   QEMU
172.0.11.249    52:54:00:e2:06:af   QEMU
172.0.11.250    34:97:f6:01:38:10   (Unknown)
172.0.11.250    9c:5c:8e:4e:dc:99   (Unknown) (DUP: 2) <<<<<< 说明 IP 冲突
172.0.11.251    52:54:00:92:40:98   QEMU
172.0.11.253    52:54:00:70:3b:51   QEMU
...
```
** nmap**
主要用来扫描端口

```
namp -n -sn 172.0.5.0/24
    -- 扫描 172.0.5.0/24 网段起来的主机，-n 表示不解析主机名，-sn 表示禁用端口扫描
nmap -sL 172.0.5.0/24
    -- 列出该网段中的所有 IP
nmap -sP 172.0.5.0/24
    -- 列出该网段中所有对 ping 包做出响应的主机 IP
nmap -A 172.0.5.6
    -- 列出指定IP开放的端口及其服务
nmap -sV 172.0.5.6 -p 0-100
    -- 列出指定端口范围的服务
```
### 抓包

**wireshark**
带图形界面的抓包工具。

常用过滤参数
ip.addr==172.0.5.75
tcp.port==9000
tcp
gtpu
tcp or gtpu
tcp and gtpu
not gtpu


**tcpdump**
网络抓包工具 
tcpdump主要包括三种类型的关键字，第一种是关于类型的关键字，主要包括host，net，port，第二种是确定传输方向的关键字，主要包括src，dst，src or dst，src and dst，这些关键字指明了传输的方向。第三种是协议关键字，包括 fddi，ip，arp， 
rarp，tcp，udp，imcp 等。除了这三种类型的关键字外，还有其他重要的关键字，如：gateway，broadcast，less，greater，还有三种逻辑运算，取非运算是’not’、’!’，与运算符是’and’、’&&’、或运算符是’or’、’||’

常用操作
```
tcpdump 'icmp' -n -i eth3 -e
指定监听包格式：
    'icmp || arp' 同时监听 icmp 和 arp 格式的包
eg:
tcpdump 'icmp' -n -i eth3 'not arp'
tcpdump 'icmp' -n -i eth3 'host 172.1.10.155'
tcpdump -ni eth0 'tcp port 9000'
    -- 指定监听 eth0 上 9000 端口的 tcp 报文
```
tcpdump 参数说明
- -i：指定tcpdump监听的网络接口
- -s：指定要监听数据包的长度
- -c：指定要监听的数据包数量，达到指定数量后自动停止抓包
- -w：指定将监听到的数据包写入文件中保存
- -A：指定将每个监听到的数据包以ACSII可见字符打印
- -n：指定将每个监听到数据包中的域名转换成IP地址后显示
- -nn：指定将每个监听到的数据包中的域名转换成IP、端口从应用名称转换成端口号后显示
- -e：指定将监听到的数据包链路层的信息打印出来，包括源mac和目的mac，以及网络层的协议
- -p：将网卡设置为非混杂模式，不能与host或broadcast一起使用
- -r：指定从某个文件中读取数据包
- -S：指定打印每个监听到的数据包的TCP绝对序列号而非相对序列号
- -l：Make stdout line buffered. Useful if you want to see the data while capturing it. 
e.g:
```
tcpdump -l | tee dat
or
tcpdump -l > dat & tail -f dat
```

### 流量监控
**nethogs**
nethogs 可以检测到具体每个应用的流量状况
```
apt-get install nethogs
nethogs br0
```
**slurm**
slurm 是一个简单实用的网络流量监控工具，一般用来监控当前网速
`slurm -i eth0`
**nload**
nlaod 是一个简单的查看网口实时流量的工具。
```
nload eth0 eth1
    -- 可以同时指定多个接口，然后用左右方向键切换查看不同的接口
```
**speedometer**
speedometer 是一个控制端下显示网口试试流量的工具，显示方式和 gnome-system-monitor 类似。
```
speedometer -r eth0 -t eth0
    -- speedometer 需要指定下行和上行的接口，-r 表示下行，-t 表示上行，这两个选项可以指定不同的接口
```
**vnstat**
vnstat 是控制台下查看网络流量的工具，直接执行会输出当前所有接口的流量数据并推出，因而可以方便地作为后台工具收集网络信息。同时也可以用 -l 选项实时监控接口流量。
```
vnstat
    -- 输出当前所有接口的流量信息并退出
vnstat -l -i eth0
    -- 监控 eth0 的实时流量，退出时会输出统计信息。
```
**iptraf-ng**
iptraf-ng 是一个 dialog jiemian 界面下的功能强大的网络监控工具，可以用来查看网速，网络使用情情况等。
```
iptraf-ng
    第一个选项查看系统各 IP 的收发包数据
    第二个选项查看个接口的统计信息
    第三个选项 "Detailed interface statistics" 是监控网速
```
**iftop**
iftop 可以用来查看网络速率和数据方向
`iftop`

**ifstat**
ifstat 是一个超简单的监控当前网络收发速度的工具
ubuntu 下
```
root@ws2:/fdsk# ifstat 
       em1        
 KB/s in  KB/s out
    0.12      0.13
    0.12      0.12
    0.12      0.12
    0.12      0.12
```
rhel 下
```
watch -n 1 'ifstat'
Every 1.0s: ifstat                                                                                         Mon Aug 29 09:57:56 2016
#7382.1804289383 sampling_interval=1 time_const=60
Interface        RX Pkts/Rate    TX Pkts/Rate    RX Data/Rate    TX Data/Rate
                 RX Errs/Drop    TX Errs/Drop    RX Over/Rate    TX Coll/Rate
lo                     0 0             0 0             0 0             0 0
                       0 0             0 0             0 0             0 0
eno1                   0 0         0 0             0 0             0 0
                       0 0             0 0             0 0             0 0
```

### 网络性能测试
**iperf**
iperf 是一个支持多平台的网络测速工具，覆盖平台包括 Windows, Linux, Android, MacOS X, FreeBSD, OpenBSD, NetBSD, VxWorks, Solaris,… 
iperf 支持测试链路中的 tcp（sctp） udp 报文传输性能，支持 ipv4 和 ipv6，默认使用 ipv4.
eg
server:
```
iperf -s
    -- 运行一个 server 端
iperf -sD
    -- 后台运行
iperf -sDB 172.0.5.75 -p 9999
    -- 指定监听的 IP 和 端口
```
client
```
iperf -c 172.0.5.75 -t 10
    -- 测试 10s，server 在 172.0.5.75
```
参数说明
- -s 以server模式启动，eg：iperf -s
- -c host以client模式启动，host是server端地址，eg：iperf -c 222.35.11.23 
通用参数
- -f [k|m|K|M] 分别表示以Kbits, Mbits, KBytes, MBytes显示报告，默认以Mbits为单位,eg:iperf -c 222.35.11.23 -f K
- -i sec 以秒为单位显示报告间隔，eg:iperf -c 222.35.11.23 -i 2
--l 缓冲区大小，默认是8KB,eg:iperf -c 222.35.11.23 -l 16
- -m 显示tcp最大mtu值
- -o 将报告和错误信息输出到文件eg:iperf -c 222.35.11.23 -o c:\iperflog.txt
- -p 指定服务器端使用的端口或客户端所连接的端口eg:iperf -s -p 9999;iperf -c 222.35.11.23 -p 9999
- -u 使用udp协议
- -w 指定TCP窗口大小，默认是8KB
- -B 绑定一个主机地址或接口（当主机有多个地址或接口时使用该参数）
- -C 兼容旧版本（当server端和client端版本不一样时使用）
- -M 设定TCP数据包的最大mtu值
- -N 设定TCP不延时
- -V 传输ipv6数据包


server专用参数
- -D 以服务方式运行ipserf，eg:iperf -s -D
- -R 停止iperf服务，针对-D，eg:iperf -s -R


client端专用参数
- -d 同时进行双向传输测试
- -n 指定传输的字节数，eg:iperf -c 222.35.11.23 -n 100000
- -r 单独进行双向传输测试
- -t 测试时间，默认10秒,eg:iperf -c 222.35.11.23 -t 5
- -F 指定需要传输的文件
- -T 指定ttl值


**netperf**
netperf 是 linux 下的网络测试工具，可以 tcp，udp 协议来测试网络带宽。

下载编译安装
     同时在要测试的两端安装 netperf
```
./configure
make
make install
```
打开 server 端
```
netserver
    -- 默认监听本地所有 IP 的 12865 端口
netserver -p port_num
    -- 指定 port
```
通过客户端发起测试
1）TCP_STREAM 测试
     批量数据传输典型的例子有ftp和其它类似的网络应用（即一次传输整个文件）。根据使用传输协议的不同，批量数据传输又分为TCP批量传输和UDP批量传输。
```
netperf -H server_IP
     -- 默认是发起 -t TCP_STREAM 流量，即 TCP 批量传输，测试市场为 10 秒
netperf -H server_IP -l 1
     -- -l 选项指定测试时间，内网的话可以选择 1 秒就可以测试大致性能
root@ltepc7:~/netperf/netperf-2.7.0# netperf -H 172.0.5.7
MIGRATED TCP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 172.0.5.7 () port 0 AF_INET
Recv   Send    Send                         
Socket Socket  Message  Elapsed             
Size   Size    Size     Time     Throughput 
bytes  bytes   bytes    secs.    10^6bits/sec 
87380  16384  16384    10.01     934.39
```
- server 端接收缓冲大小为 87380
- client 端发送缓冲大小为 16384
- 发送的测试分组大小为 16384
- 测试市场为 10.01
- 吞吐量为 934.39 Mbits/秒， 即千兆网络

2）UDP_STREAM 测试
     UDP_STREAM用来测试进行UDP批量传输时的网络性能。需要特别注意的是，此时测试分组的大小不得大于socket的发送与接收缓冲大小，否则netperf会报出错
```
root@ltepc7:~/netperf/netperf-2.7.0# netperf -H 172.0.5.7 -t UDP_STREAM
MIGRATED UDP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 172.0.5.7 () port 0 AF_INET
Socket  Message  Elapsed      Messages               
Size    Size     Time         Okay Errors   Throughput
bytes   bytes    secs            #      #   10^6bits/sec
212992   65507   10.00       18212      0     954.33
212992           10.00       18212            954.33
```

- 第一行是本地系统的发送统计
- 第二行是远端系统接收的情况
- 分两个方向显示是基于 UDP 的不可靠性考虑


3）TCP_RR 测试
     TCP_RR方式的测试对象是多次TCP request和response的交易过程，但是它们发生在同一个TCP连接中，这种模式常常出现在数据库应用中。数据库的client程序与server程序建立一个TCP连接以后，就在这个连接中传送数据库的多次交易过程
```
root@ltepc7:~/netperf/netperf-2.7.0# netperf -H 172.0.5.7 -t TCP_RR
MIGRATED TCP REQUEST/RESPONSE TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 172.0.5.7 () port 0 AF_INET : first burst 0
Local /Remote
Socket Size   Request  Resp.   Elapsed  Trans.
Send   Recv   Size     Size    Time     Rate        
bytes  Bytes  bytes    bytes   secs.    per sec  
16384  87380  1        1       10.00    5595.35  
16384  87380
root@ltepc7:~/netperf/netperf-2.7.0#
```
- 第一行为本地系统情况
- 第二行为远端系统情况
- 平均交易率为 5595.35次/秒。
- 默认是每次交易中的 request 和 response 分组大小为 1 字节，不具有很大的实际意义，可以通过修改分组大小来测试具体场景。
```
root@ltepc7:~/netperf/netperf-2.7.0# netperf -H 172.0.5.7 -t TCP_RR -- -r 32,1024
MIGRATED TCP REQUEST/RESPONSE TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 172.0.5.7 () port 0 AF_INET : first burst 0
Local /Remote
Socket Size   Request  Resp.   Elapsed  Trans.
Send   Recv   Size     Size    Time     Rate        
bytes  Bytes  bytes    bytes   secs.    per sec  
16384  87380  32       1024    10.00    5369.55  
16384  87380
root@ltepc7:~/netperf/netperf-2.7.0#
```
- -r req,resp 
  – 设置 request 和 response 分组的大小
4）UDP_RR 测试
与TCP_RR不同，TCP_CRR为每次交易建立一个新的TCP连接。最典型的应用就是HTTP，每次HTTP交易是在一条单独的TCP连接中进行的。因此，由于需要不停地建立新的TCP连接，并且在交易结束后拆除TCP连接，交易率一定会受到很大的影响。
```
root@ltepc7:~/netperf/netperf-2.7.0# netperf -H 172.0.5.7 -t UDP_RR
MIGRATED UDP REQUEST/RESPONSE TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 172.0.5.7 () port 0 AF_INET : first burst 0
Local /Remote
Socket Size   Request  Resp.   Elapsed  Trans.
Send   Recv   Size     Size    Time     Rate        
bytes  Bytes  bytes    bytes   secs.    per sec  
212992 212992 1        1       10.00    6049.38  
212992 212992
root@ltepc7:~/netperf/netperf-2.7.0#
```

### 网络安全
**iptables**
基本操作
```
iptables -F
     -- 清空当前所有的 iptables 规则（默认规则，也即 policy 除外）
iptables -nL
     -- 查看当前配置的 iptables 规则
iptables -nL --line-number
     -- 显示每条规则的序号（在删除规则的时候有用到）
iptables -A INPUT -s 172.0.5.75 -p tcp --dport 46989 -j ACCEPT
     -- 允许 172.0.5.75 这个 IP 通过 tcp 协议的 46989 端口连接到本地
iptables -A OUTPUT -p tcp -j ACCEPT
     -- 开放所有的 tcp 出口端口
iptables -P INPUT DROP    
     -- 设置入口默认为 DROP
iptables -P OUTPUT DROP
     -- 设置出口默认为 DROP
iptables -P FORWARD DROP
     -- 设置转发默认为 DROP
iptables -D INPUT 1
     -- 删除 INPUT 链中的第一条规则
```
保存 iptables 规则以重启生效
```
iptables-save > /etc/iptables-rules
echo "pre-up iptables-restore < /etc/iptables-rules" >> /etc/network/interfaces
```
清空 iptabels 配置
```
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
```
**selinux**
安全增强式Linux（SELinux, Security-Enhanced Linux）是一种强制访问控制（mandatory access control）的实现。它的作法是以最小权限原则（principle of least privilege）为基础，在Linux核心中使用Linux安全模块（Linux Security Modules）。它并非一个Linux发行版，而是一组可以套用在类Unix操作系统（如Linux、BSD等）的修改。
基本管理操作
```
sestatus -b
    -- 查看所有的 selinux 服务配置
setsebool -P service_name on
    -- 开启该服务
```
禁用 selinux
```
#!/bin/sh
sed 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config > tmp_selinux
cp /etc/selinux/config /etc/selinux/config_backup
mv -vf ./tmp_selinux /etc/selinux/config
chmod 644 /etc/selinux/config
```
**firewarll**
基本操作
rhel
```
systemctl status firewalld
systemctl disable firewalld
systemctl status firewalld
```
**NAT**
network address tranlator

    NAT 只是网络地址转换，所以如果要使用的话，只需要设置系统的转发和配置 iptables，然后搭配其它网络设备使用，比如 bridge，tap/tun 等。

- SNAT: 目标地址不变，重新改写源地址，并在本机建立NAT表项，当数据返回时，根据NAT表将目的地址数据改写为数据发送出去时候的源地址，并发送给主机 
目前大多都是解决内网用户用同一个公网地址上网的情况

- DNAT: 和SNAT相反，源地址不变，重新修改目标地址，在本机建立NAT表项，当数据返回时，根据NAT表将源地址修改为数据发送过来时的目标地址，并发给远程主机

- PNAT: 在DNAT的基础上，可以根据请求数据包的端口做PNAT（端口转换，也称为端口映射），可以更句请求数据包不同的端口改写不同的目标地址，从而发送给不同的主机

DNAT 和 PNAT 在用一个公网地址做不同服务时用的比较多，而且相对来说，用NAT的方式可以隐藏后端服务器的真实地址，更加的安全



配置
打开 IP 转发功能
`echo 1 > /proc/sys/net/ipv4/ip_forward`
设置 iptables
SNAT
`iptables -t nat -A POSTROUTING -s 172.0.5.0/24  -j SNAT --to-source 10.0.0.1`
DNAT/PNAT
`iptables -t nat -A PREROUTING -p tcp -d 对外的IP --dport 80 -j DNAT --to 192.168.122.140:80`

### 常用网络配置文件
ubuntu 14.04
网络接口配置
配置文件：/etc/network/interfaces

示例
配置网口
```
auto eth0
iface eth0 inet static
        address 172.0.5.78
        netmask 255.255.255.0
        gateway 172.0.5.1
        dns-nameservers 114.114.114.114
```
配置 bridge
```
auto br0
iface br0 inet static
     address 172.0.10.155
     netmask 255.255.255.0
     gateway 172.0.10.1
     dns-nameservers 8.8.8.8
     bridge_ports em1
        bridge_stp off
        bridge_fd 0
        bridge_maxwait 0
```
配置 tap
```
auto tap1
iface tap1 inet manual
        up ifconfig $IFACE up
        down ifconfig $IFACE down
        tunctl_user root
```
配置路由
```
up route add -net xxx.xxx.xxx.xxx/xx gw xxx.xxx.xxx.xxx ethX
```
rhel 7.1
配置文件 /etc/sysconfig/network-scripts/ifcfg-em1 
示例
```
TYPE=Ethernet
BOOTPROTO=static
default via 172.0.1.1 dev em1
IPADDR=172.0.1.14
IP_GATEWAY=172.0.1.1
NETMASK=255.255.255.0
DNS1=114.114.114.114
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=em1
UUID=4f150533-b098-4ffb-b382-ea10869186ad
DEVICE=em1
ONBOOT=yes
```
openSUSE leap 15
网口配置
网口配置文件为 /etc/sysconfig/network/ifcfg-xxx 
man 5 ifcfg 查看帮助。 
示例：
```
cat /etc/sysconfig/network/ifcfg-eth0
BOOTPROTO='static'
BROADCAST=''
ETHTOOL_OPTIONS=''
IPADDR='172.0.11.95/24'
MTU=''
NAME='Ethernet Card 0'
NETWORK=''
REMOTE_IPADDR=''
STARTMODE='auto'
ONBOOT=yes
```
路由配置
配置文件为： 
/etc/sysconfig/network/ifroute- 
/etc/sysconfig/network/routes

查看帮助： 
man routes

示例：

```
cat /etc/sysconfig/network/routes 
# IPv4 routes in CIDR prefix notation:
# Destination     [Gateway]         -                  Interface
default           172.0.11.1        -                  eth
```
DNS 配置
配置文件为： 
/etc/sysconfig/network/config

配置参数：
```
NETCONFIG_DNS_STATIC_SEARCHLIST
NETCONFIG_DNS_STATIC_SERVERS
NETCONFIG_DNS_FORWARDER
```
示例:

```
NETCONFIG_DNS_POLICY="auto"
NETCONFIG_DNS_STATIC_SEARCHLIST=""
NETCONFIG_DNS_STATIC_SERVERS="1.1.1.1"
```

### 常见问题处理
**链路不通**
1. 检查网口是否 up
ifconfig 命令显示所有已 up 的网络设备 
ifconfig -a 命令查看所有可用的网络设备



2. 检查物理链路
bash 下
```
ifconfig em1 up
ethtool em1
```
3. 检查网口配置是否正确
     注意检查 ip，掩码是否一直，广播地址是否正确

4. 检查路由配置是否正确
`route -n`
5. 检查是否接在正确的交换机
     tcpdump -ni iface 查看能不能抓到 arp 包，查看抓到的 arp 包可以大致知道当前网口所接的交换机



6. Bridge 不通
     若是有桥接，收发包到桥接口停止，则检查桥接设备

ii. 检查接口是否接在 bridge 上面
`brctl show br0`
ii. 检查桥接设备上的 mac 表
`brctl showmacs br0`
7. 检查 iptables 或 acl 是否有限制
iptables
```
iptables
iptables -nL
```

8. vlan 的影响
     若配置了 vlan 则检查 vlan 配置是否正确
`cat /proc/net/vlan/xxx`
9. vrf 的影响
`ip netns xxxx`
**IP 冲突**
检查是否冲突
1. arp-scan
只适用于同一网段的检查，比如要检查 172.0.11.x 是否有 IP 冲突，需要使用另一个直接接到同一个交换机的接口。
```
arp-scan --interface eth0 172.0.5.0/24
...
172.0.11.246    52:54:00:92:c3:2d   QEMU
172.0.11.249    52:54:00:e2:06:af   QEMU
172.0.11.250    34:97:f6:01:38:10   (Unknown)
172.0.11.250    9c:5c:8e:4e:dc:99   (Unknown) (DUP: 2)
...
    -- 扫描整个网段并列出所有主机，如果有冲突，会以 (DUP: N) 标记，比如上面的 172.0.11.250
```
2. arping

只适用于同一网段的检查，比如要检查 172.0.11.x 是否有 IP 冲突，需要使用另一个直接接到同一个交换机的接口。

安装
`apt-get install iputils-arping`
检查
```
arping -I br0 -D 172.0.11.55
ARPING 172.0.11.55 from 0.0.0.0 br0
Unicast reply from 172.0.11.55 [F8:BC:12:12:1A:00]  0.669ms
Sent 1 probes (1 broadcast(s))
Received 1 response(s)
    -- 如果有冲突，则会收到至少 2 两个 response
```
3. ping IP
```
tcpdump -n -e 'icmp' -i em1
一个窗口发起ping包，另一个窗口抓包，看到对端的mac地址有超过一个的话就是IP冲突了
```
4. 检查 arp 缓存
`cat /proc/net/arp | grep IP`
查看缓存的arp信息，根据IP筛选，如果有超过1个mac地址配置的是同一个IP，说明IP冲突

5. 遍历网段内的存活的主机
`nmap -n -sn 172.0.5.0/24`
列出局域网中对ping包做出相应的所有主机，如果有超过1个主机对同一个IP做出相应，说明IP冲突

### 疑难杂症
1. 物理接口收到 arp request 后立即往回发
     物理网口收到一个 arp request 报文之后，除了将这个报文发送出去，还立即往接收的方向发送了一个相同的报文。

     这个问题是在 INTEL X710 的 10G 网卡上遇到的，将驱动从 1.2.2-k 升级到 1.5.6 即可解决。无法通过 rmmod 后重新 modprobe 的方式解决，故而认为是驱动问题。

### 参考
- [Linux 上的基础网络设备详解 - IBM ](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/)
- [Linux 上虚拟网络与真实网络的映射 - IBM ](https://www.ibm.com/developerworks/cn/linux/1312_xiawc_linuxvirtnet/)
- [ARP 协议解密（arp 欺骗） - IBM ](https://www.ibm.com/developerworks/cn/linux/l-arp/)
- [地址解析协议 - wiki 中文 ](https://zh.wikipedia.org/wiki/%E5%9C%B0%E5%9D%80%E8%A7%A3%E6%9E%90%E5%8D%8F%E8%AE%AE)
- [ARP欺骗 - wiki 中文 ](https://zh.wikipedia.org/wiki/ARP%E6%AC%BA%E9%A8%99)
- [vlan of ubuntu wiki ](https://wiki.ubuntu.com/vlan)
- [安全增强式Linux - wiki 中文 ](https://zh.wikipedia.org/wiki/%E5%AE%89%E5%85%A8%E5%A2%9E%E5%BC%BA%E5%BC%8FLinux)
- [理解linux网络设备](http://www.jianshu.com/p/d651aba19c50)
- [详解网络传输中的三张表，MAC地址表、ARP缓存表以及路由表](http://dengqi.blog.51cto.com/5685776/1223132)
- [network namespace 和 vrf 的使用 ](http://rmadapur.blogspot.com/2014/02/vrf-linux-network-name-space.html)
- [kvm 网络虚拟化基础 ](https://www.ibm.com/developerworks/community/blogs/132cfa78-44b0-4376-85d0-d3096cd30d3f/entry/KVM_%E7%BD%91%E7%BB%9C%E8%99%9A%E6%8B%9F%E5%8C%96%E5%9F%BA%E7%A1%80_%E6%AF%8F%E5%A4%A95%E5%88%86%E9%92%9F%E7%8E%A9%E8%BD%AC_OpenStack_9?lang=en)
- [Neutron - OpenStack 网络实现 ](https://yeasy.gitbooks.io/openstack_understand_neutron/content/concept/)
- [wpa_supplicant无线网络配置 ](http://blog.163.com/wxiongn@126/blog/static/11788203820102262748358/)


- [netperf 与网络性能测量 ](https://www.ibm.com/developerworks/cn/linux/l-netperf/)
- [路由详解](http://www.cnblogs.com/xingyun/p/4858619.html)
- [Linux 常用网络管理命令 ](http://tech.it168.com/a2014/0307/1600/000001600096_all.shtml)
- [iperf 官网说明 ](https://iperf.fr/)
- [iperf 使用说明 ](https://sites.google.com/site/ibmsdu/network-security-development/Linux-Firewall/iperf%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
- [IPtables之四：NAT原理和配置 ](http://lustlost.blog.51cto.com/2600869/943110)
- [通过iptables实现端口转发和内网共享上网](http://wwdhks.blog.51cto.com/839773/1154032)


     





