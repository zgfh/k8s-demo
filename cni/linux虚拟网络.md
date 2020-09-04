## linux虚拟网络
在Linux中，独立的网络虚拟化实现就是net namespace技术，依托别的技术实现的网络虚拟化就是虚拟机技术，



## 网卡物理层
* 以太网ETHx：普通双绞线或者光纤；  
* TUN/TAP：用户可以用文件句柄操作的字符设备；
* IFB：一次到原始网卡的重定向操作；
* VETH：触发虚拟网卡对儿peer的RX；
* VTI：加密引擎；

## OS层


## bridge
https://wiki.linuxfoundation.org/networking/bridge


## bound & 子接口
[官方文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-configuring_a_vlan_over_a_bond)
[教程](https://www.unixmen.com/linux-basics-create-network-bonding-on-centos-76-5/)

创建bound0网卡

```
modprobe --first-time bonding

cat >/etc/sysconfig/network-scripts/ifcfg-bound0 <<-EOF
DEVICE=bond0
NAME=bond0
TYPE=Bond
BONDING_MASTER=yes
IPADDR=10.0.2.200  # change this
NETMASK=255.255.255.0  # change this
GATEWAY=192.168.1.1  # change this
ONBOOT=yes
BOOTPROTO=none
BONDING_OPTS="mode=1 miimon=100" 
EOF
```

修改作为slave0的网卡，添加

```
BOOTPROTO=none
MASTER=bond0
ONBOOT="yes"
SLAVE=yes
```
如: 
```
cat >/etc/sysconfig/network-scripts/ifcfg-bound0-slave0 <<-EOF
DEVICE=enp0s17 # 更改为真实的网卡
NAME=bond0-slave
TYPE=Ethernet
BOOTPROTO=none
ONBOOT=yes
MASTER=bond0
SLAVE=yes
EOF
```

```
# 重启网络
systemctl restart network.service
# 查看生效
cat /proc/net/bonding/bond0
# 测试，停掉一个网卡，网络不会中断
ifdown /etc/sysconfig/network-scripts/ifcfg-bound0-slave0
```

## 子接口
[官方文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/sec-configuring_a_vlan_over_a_bond)

命令行模式
```
# 开启混杂模式
ip link set bond0  promisc on
ip link add link bond0 name bond0.201 type vlan id 201
ip -d link show bond0.201

ip addr add 10.201.0.17/24 brd 10.201.0.255 dev bond0.201 
ip link set dev bond0.201  up
ip route replace 10.201.0.0/24 via 10.201.0.1 dev bond0.201
```

配置文件
```
cat >/etc/sysconfig/network-scripts/ifcfg-bond0.192 <<-EOF
DEVICE=bond0.192
NAME=bond0.192
BOOTPROTO=none
ONPARENT=yes
IPADDR=192.168.10.1
NETMASK=255.255.255.0
VLAN=yes
NM_CONTROLLED=no
EOF

```


## VEPA


## IPVLAN


## VIP
https://www.cnblogs.com/crazylqy/p/7741958.html

## MACVTAP
https://cizixs.com/2017/02/14/network-virtualization-macvlan/
https://www.hi-linux.com/posts/40904.html

## MACVLAN
一种将一块以太网卡虚拟成多块以太网卡的极简单的方案
四种模式
private mode：过滤掉所有来自其他 macvlan 接口的报文，因此不同 macvlan 接口之间无法互相通信
vepa(Virtual Ethernet Port Aggregator) mode： 需要主接口连接的交换机支持 VEPA/802.1Qbg 特性。所有发送出去的报文都会经过交换机，交换机作为再发送到对应的目标地址（即使目标地址就是主机上的其他 macvlan 接口），也就是 hairpin mode 模式，这个模式用在交互机上需要做过滤、统计等功能的场景。
bridge mode：通过虚拟的交换机讲主接口的所有 macvlan 接口连接在一起，这样的话，不同 macvlan 接口之间能够直接通信，
  不需要将报文发送到主机之外。这个模式下，主机外是看不到主机上 macvlan interface 之间通信的报文的。
passthru mode：暂时没有搞清楚这个模式要解决的问题



## 路由命令
http://cn.linux.vbird.org/linux_server/0140networkcommand.php
所谓路由表，指的是路由器或者其他互联网网络设备上存储的表，该表中存有到达特定网络终端的路径，在某些情况下，还有一些与这些路径相关的度量。
路由器的主要工作就是为经过路由器的每个数据包寻找一条最佳的传输路径，并将该数据有效地传送到目的站点。
由此可见，选择最佳路径的策略即路由算法是路由器的关键所在。为了完成这项工作，在路由器中保存着各种传输路径的相关数据——路由表（Routing Table）
，供路由选择时使用，表中包含的信息决定了数据转发的策略。打个比方，路由表就像我们平时使用的地图一样，标识着各种路线，路由表中保存着子网的标志信息、网上路由器的个数和下一个路由器的名字等内容。
路由表根据其建立的方法，可以分为动态路由表和静态路由表。

```
Usage: ip rule [ list | add | del ] SELECTOR ACTION （add 添加；del 删除； llist 列表）
SELECTOR := [ from PREFIX 数据包源地址] [ to PREFIX 数据包目的地址] [ tos TOS 服务类型][ dev STRING 物理接口] [ pref NUMBER ] [fwmark MARK iptables 标签]
ACTION := [ table TABLE_ID 指定所使用的路由表] [ nat ADDRESS 网络地址转换][ prohibit 丢弃该表| reject 拒绝该包| unreachable 丢弃该包]
[ flowid CLASSID ]
TABLE_ID := [ local | main | default | new | NUMBER ]
```

举例:
```
# 删除 122.252.228.38/32 的路由
ip route delete 122.252.228.38/32
# 通过路由表 inr.ruhep 路由来自源地址为192.203.80/24的数据包 
ip route add default via 195.96.98.253 dev ppp2 table John

# 范例二：路由的增加与删除
[root@www ~]# route del -net 169.254.0.0 netmask 255.255.0.0 dev eth0
# 上面这个动作可以删除掉 169.254.0.0/16 这个网域！
# 请注意，在删除的时候，需要将路由表上面出现的信息都写入
# 包括 netmask , dev 等等参数喔！注意注意

[root@www ~]# route add -net 192.168.100.0 \
> netmask 255.255.255.0 dev eth0
# 透过 route add 来增加一个路由！请注意，这个路由的设定必须要能够与你的网络互通。
# 举例来说，如果我下达底下的指令就会显示错误：
# route add -net 192.168.200.0 netmask 255.255.255.0 gw 192.168.200.254
# 因为我的主机内仅有 192.168.1.11 这个 IP ，所以不能直接与 192.168.200.254
# 这个网段直接使用 MAC 互通！这样说，可以理解吗？

[root@www ~]# route add default gw 192.168.1.250
# 增加预设路由的方法！请注意，只要有一个预设路由就够了喔！
# 同样的，那个 192.168.1.250 的 IP 也需要能与你的 LAN 沟通才行！
# 在这个地方如果你随便设定后，记得使用底下的指令重新设定你的网络
# /etc/init.d/network restart
```

linux 系统中，可以自定义从 1－252个路由表，其中，linux系统维护了4个路由表：

0#表： 系统保留表
253#表： defulte table 没特别指定的默认路由都放在改表
254#表： main table 没指明路由表的所有路由放在该表
255#表： locale table 保存本地接口地址，广播地址、NAT地址 由系统维护，用户不得更改




参考:
[图解几个与Linux网络虚拟化相关的虚拟网卡](https://blog.csdn.net/dog250/article/details/45788279)
