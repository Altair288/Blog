---
layout: post
title: 第一课
date: 2023-02-13
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 什么是eNSP

eNSP 是一款由华为自主开发的、免费的、可扩展的、图形化操作的网络仿真工具平台，主要对企业网络路由器、交换机及相关物理设备进行软件仿真，完美呈现真是设备实景，支持大型网络模拟，可以让广大用户能够在没有真实设备的情况下模拟演练，学习网络技术。
> Copper: 双绞线

# IPv4地址分为哪几个大类？

为了便于对IP地址进行管理， 根据IPv4地址的第一个字节，IPv4地址可以分为以下五大类。
* A类：0~127。
* B类：128~191。
* C类：192~223。
* D类：224~239，组播地址。
* E类：240~254，保留为研究测试使用。
> 网络IP由网络号和主机号组成。

# 子网掩码

子网掩码不能单独存在，它必须结合IP地址一起使用。子网掩码只有一个作用，就是将某个IP地址划分成网络地址和主机地址两部分。

子网掩码的设定必须遵循一定的规则。与IP地址相同，子网掩码的长度也是32位，左边是网络位，用二进制数字“1”表示；右边是主机位，用二进制数字“0”表示。其中，“1”有24个，代表与此相对应的IP地址左边24位是网络号；“0”有8个，代表与此相对应的IP地址右边8位是主机号。这样，子网掩码就确定了一个IP地址的32位二进制数字中哪些是网络号、哪些是主机号。这对于采用TCP/IP协议的网络来说非常重要，只有通过子网掩码，才能表明一台主机所在的子网与其他子网的关系，使网络正常工作。

常用的子网掩码 子网掩码有数百种，这里只介绍最常用的两种子网掩码，它们分别是“255.255.255.0”和“255.255.0.0”。

子网掩码是“255.255.255.0”的网络：最后面一个数字可以在0~255范围内任意变化，因此可以提供256个IP地址。但是实际可用的IP地址数量是256-2，即254个，因为主机号不能全是“0”或全是“1”。

子网掩码是“255.255.0.0”的网络：后面两个数字可以在0~255范围内任意变化（注意与第一种情况的不同），可以提供255^2个IP地址。但是实际可用的IP地址数量是255^2-2，即65023个。 IP地址的子网掩码设置不是任意的。如果将子网掩码设置过大，也就是说子网范围扩大，那么，根据子网寻径规则，很可能发往和本地机不在同一子网内的目的机的数据，会因为错误的判断而认为目的机是在同一子网内，那么，数据包将在本子网内循环，直到超时并抛弃，使数据不能正确到达目的机，导致网络传输错误；如果将子网掩码设置得过小，那么就会将本来属于同一子网内的机器之间的通信当做是跨子网传输，数据包都交给缺省网关处理，这样势必增加缺省网关的负担，造成网络效率下降。因此，子网掩码应该根据网络的规模进行设置。 如果一个网络的规模不超过254台电脑，采用“255.255.255.0”作为子网掩码就可以了，现在大多数局域网都不会超过这个数字，因此“255.255.255.0”是最常用的IP地址子网掩码。 默认子网掩码 在Windows操作系统中，如果给一个网卡指定IP地址，系统会自动填入一个默认的子网掩码。这是Windows为了节省用户输入时间自动产生的子网掩码。比如，局域网最常使用的IP地址“192.168.x.x”默认的子网掩码是“255.255.255.0”。一般情况下，IP地址使用默认子网掩码就可以了。

# 网关

网关实质上是一个网络通向其他网络的IP地址。比如有网络A和网络B，网络A的[IP]{"https://baike.baidu.com/item/IP?fromModule=lemma_inlink" "百度百科 IP"}地址范围为“192.168.1.1~192. 168.1.254”，子网掩码为255.255.255.0；网络B的IP地址范围为“192.168.2.1~192.168.2.254”，子网掩码为255.255.255.0。在没有路由器的情况下，两个网络之间是不能进行TCP/IP通信的，即使是两个网络连接在同一台交换机（或集线器）上，TCP/IP协议也会根据子网掩码（255.255.255.0）与主机的IP 地址作 “与” 运算的结果不同判定两个网络中的主机处在不同的网络里。而要实现这两个网络之间的通信，则必须通过网关。如果网络A中的主机发现数据包的目的主机不在本地网络中，就把数据包转发给它自己的网关，再由网关转发给网络B的网关，网络B的网关再转发给网络B的某个主机。这就是网络A向网络B转发数据包的过程。

# 路由器

## 使用 system-view 指令使路由器控制台进入系统视图

	<Huawei>system-view
	Enter system view, return user view with Ctrl+Z.
	[Huawei]

## 设置主机名 sysname

	[Huawei]sysname localhost
	[localhost]

## 开关信息中心 undo info-center enable

	[Huawei]undo info-center enable
	Info: Information center is disabled. 
	[Huawei]

## 给端口设置IP

```shell
[Huawei]interface g0/0/0    #进入端口g0/0/0
[Huawei-GigabitEthernet0/0/0]ip address <IP Address> 24    #设置IP
Feb 13 2023 15:53:05-08:00 Huawei %%01IFNET/4/LINK_STATE(l)[0]:The line protocol
 IP on the interface GigabitEthernet0/0/0 has entered the UP state. 

[Huawei-GigabitEthernet0/0/0]display current-configuration    #查看当前信息
[V200R003C00]
#
 snmp-agent local-engineid 800007DB03000000000000
 snmp-agent 
#
 clock timezone China-Standard-Time minus 08:00:00
#
portal local-server load portalpage.zip
#
 drop illegal-mac alarm
#
 set cpu-usage threshold 80 restore 75
#
aaa 
 authentication-scheme default
 authorization-scheme default
 accounting-scheme default
 domain default 
 domain default_admin 
 local-user admin password cipher %$%$K8m.Nt84DZ}e#<0`8bmE3Uw}%$%$
 local-user admin service-type http
#
firewall zone Local
 priority 15
#
interface Ethernet0/0/0
#
interface Ethernet0/0/1
#
interface Ethernet0/0/2
#
interface Ethernet0/0/3
#
interface Ethernet0/0/4
#
interface Ethernet0/0/5
#
interface Ethernet0/0/6
#
interface Ethernet0/0/7
#
interface GigabitEthernet0/0/0
 ip address 192.168.1.1 255.255.255.0 
#
interface GigabitEthernet0/0/1
#
interface NULL0
#
user-interface con 0
 authentication-mode password
user-interface vty 0 4
user-interface vty 16 20
#
wlan ac
#
return
[Huawei-GigabitEthernet0/0/0]
```
