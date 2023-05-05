---
layout: post
title: 第二十课
date: 2023-04-26
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---


# 什么是DHCP？

动态主机配置协议DHCP（Dynamic Host Configuration Protocol）是一种网络管理协议，用于集中对用户IP地址进行动态管理和配置。
DHCP于1993年10月成为标准协议，其前身是BOOTP协议。DHCP协议由RFC 2131定义，采用客户端/服务器通信模式，由客户端（DHCP Client）向服务器（DHCP Server）提出配置申请，DHCP Server为网络上的每个设备动态分配IP地址、子网掩码、默认网关地址，域名服务器（DNS）地址和其他相关配置参数，以便可以与其他IP网络通信。

# 为什么要使用DHCP？

在IP网络中，每个连接Internet的设备都需要分配唯一的IP地址。DHCP使网络管理员能从中心结点监控和分配IP地址。当某台计算机移到网络中的其它位置时，能自动收到新的IP地址。DHCP实现的自动化分配IP地址不仅降低了配置和部署设备的时间，同时也降低了发生配置错误的可能性。另外DHCP服务器可以管理多个网段的配置信息，当某个网段的配置发生变化时，管理员只需要更新DHCP服务器上的相关配置即可，实现了集中化管理。

总体来看，DHCP带来了如下优势：

- 准确的IP配置：IP地址配置参数必须准确，并且在处理“ 192.168.XXX.XXX”之类的输入时，很容易出错。另外印刷错误通常很难解决，使用DHCP服务器可以最大程度地降低这种风险。
- 减少IP地址冲突：每个连接的设备都必须有一个IP地址。但是，每个地址只能使用一次，重复的地址将导致无法连接一个或两个设备的冲突。当手动分配地址时，尤其是在存在大量仅定期连接的端点（例如移动设备）时，可能会发生这种情况。DHCP的使用可确保每个地址仅使用一次。
- IP地址管理的自动化：如果没有DHCP，网络管理员将需要手动分配和撤消地址。跟踪哪个设备具有什么地址可能是徒劳的，因为几乎无法理解设备何时需要访问网络以及何时需要离开网络。DHCP允许将其自动化和集中化，因此网络专业人员可以从一个位置管理所有位置。
- 高效的变更管理：DHCP的使用使更改地址，范围或端点变得非常简单。例如，组织可能希望将其IP寻址方案从一个范围更改为另一个范围。DHCP服务器配置有新信息，该信息将传播到新端点。同样，如果升级并更换了网络设备，则不需要网络配置。

# 配置 DHCP

## 开启 DHCP 功能

### 背景信息

配置DHCP Server功能之前，必须先在系统视图下开启DHCP功能。

> 说明
>
> - dhcp enable命令是DHCP相关功能的总开关，DHCP Relay、DHCP Snooping、DHCP Server等功能都要在执行dhcp enable命令使能DHCP功能后才会生效。执行undo dhcp enable命令后，设备上所有的DHCP相关的配置会被删除；再次执行dhcp enable命令使能DHCP功能后，设备上所有DHCP相关配置将被恢复为缺省配置。
>
> - 开启DHCP功能后，如果使能了STP功能，可能会造成地址分配较慢。STP功能缺省处于使能状态，如果确认不需要使能STP功能，可以执行命令undo stp enable去使能STP功能。
>
> - 如果以S2750EI、S5700-10P-LI-AC和S5700-10P-PWR-LI-AC开启DHCP功能后，为提高设备性能，建议通过命令assign forward-mode ipv4-hardware使能IPv4报文三层硬转功能。

### 操作步骤

1. 执行命令system-view，进入系统视图。
2. 执行命令dhcp enable，开启DHCP功能。
> 缺省情况下，DHCP功能处于关闭状态。
3. （可选）执行命令dhcp speed-limit auto，开启DHCP报文动态限速功能。
> 缺省情况下，DHCP报文动态限速功能处于关闭状态。

## 开启DHCP服务器功能

### 背景信息

配置设备作为DHCP服务器之前，首先需要开启DHCP功能；然后再开启基于接口地址池或者全局地址池的DHCP服务器功能。

### 操作步骤

- 基于接口方式：

执行命令 `system-view`，进入系统视图。
执行命令 `interface interface-type interface-number[.subinterface-number ]`，进入接口视图或子接口视图。

> 说明
>
> 仅S5720EI、S5720HI、S6720EI和S6720S-EI支持子接口。

（可选）对于以太网接口，执行命令 `undo portswitch`，配置接口切换到三层模式。

缺省情况下，以太网接口处于二层模式。

> 说明
>
> 仅S5720HI、S5720EI、S6720EI和S6720S-EI支持二层模式与三层模式切换。

执行命令 `ip address ip-address { mask | mask-length }`，配置接口的IP地址。
执行命令 `dhcp select interface`，开启接口采用接口地址池的DHCP服务器功能。

缺省情况下，接口下未开启采用接口地址池的DHCP服务器功能。

如果设备作为DHCP服务器为多个接口下的客户端提供DHCP服务，需要分别在多个接口上重复执行此步骤使能DHCP服务功能。

- 基于全局方式：

执行命令 `system-view`，进入系统视图。
执行命令 `interface interface-type interface-number[.subinterface-number ]`，进入接口视图或子接口视图。

> 说明
>
> 仅S5720EI、S5720HI、S6720EI和S6720S-EI支持子接口。

（可选）对于以太网接口，执行命令 `undo portswitch`，配置接口切换到三层模式。

缺省情况下，以太网接口处于二层模式。

> 说明
>
> 仅S5720HI、S5720EI、S6720EI和S6720S-EI支持二层模式与三层模式切换。

配置接口的IP地址。

1. 执行命令 `ip address ip-address { mask | mask-length }`，配置接口的主IP地址。
2. （可选）执行命令 `ip address ip-address { mask | mask-length } sub`，配置接口的从IP地址。

> 说明
>
> 设备根据接口主从IP地址选择全局地址池，仅适用于DHCP客户端和DHCP服务器处于同一个网段的场景。

接口配置IP地址后，此接口下的客户端申请IP地址时：
- 如果设备与客户端处于同一个网段（即无中继场景），设备会首先选择与此接口的主IP地址在同一个网段的地址池来分配IP地址。如果主IP地址对应地址池耗尽或未配置主IP地址对应地址池，使用从IP地址对应的地址池给客户端分配地址。如果接口未配置IP地址，或者没有和接口地址在相同网段的地址池，客户端无法成功申请IP地址。
- 如果设备与客户端处于不同网段（即有中继场景），DHCP服务器解析收到的DHCP请求报文中giaddr字段指定的IP地址，选择与此IP地址在同一个网段的地址池来进行IP地址分配。如果该IP地址匹配不到相应的地址池，客户端无法成功申请IP地址。

执行命令 `dhcp select global`，使能接口采用全局地址池的DHCP服务器功能。如果DHCP客户端和设备之间存在DHCP中继，此步骤为可选；否则，为必选。

缺省情况下，未使能接口采用全局地址池的DHCP服务器功能。

## 配置地址池

### 创建地址池

#### 背景信息

接口地址池内的IP地址与此接口的IP地址属于同一网段，且地址池中地址只能分配给此接口下的客户端。全局地址池中地址可以分配给设备所有接口下的客户端。

> 说明
>
> 创建地址池后，网段中的如下三个地址不参与自动分配：
> - 网络地址（主机号全为0）
> - 广播地址（主机号全为1）
> - 配置客户端的网关地址后，DHCP客户端的网关地址

#### 操作步骤

执行命令 `system-view`，进入系统视图。
执行命令 `ip pool ip-pool-name`，创建全局地址池，同时进入全局地址池视图。

缺省情况下，设备上没有创建任何全局地址池。
配置的ip-pool-name作为标识不同地址池的名字。例如，为楼层1的员工创建一个名字为“global\_f1”的全局地址池：
```shell
[HUAWEI]ip pool global_f1
```

执行命令 `network ip-address [ mask { mask | mask-length } ]`，配置全局地址池可动态分配的IP地址范围。
缺省情况下，未配置全局地址池可动态分配的IP地址范围。
一个地址池中只能配置一个地址段，通过设定掩码长度可控制地址范围的大小。

> 说明
>
> 配置全局地址池可动态分配的IP地址范围时，请尽量保证该地址范围与DHCP Server接口或DHCP Relay接口地址的网段一致，以免分配错误的IP地址。
> 配置地址池时，网络地址段必须是A、B、C三类IP地址中的一种，并且掩码不能配置为0、1、31和32。

（可选）执行命令 `vpn-instance vpn-instance-name`，配置地址池下的VPN实例。

缺省情况下，地址池下没有配置VPN实例。

正常情况下，为了避免IP地址冲突，一个地址池只能为一个网段的客户端分配IP地址。在BGP/MPLS IP VPN场景中，经常有不同VPN网络使用相同网段地址的情况。如果不同VPN网络中客户端通过同一个DHCP服务器获取IP地址，可以配置此步骤，使用同一个地址池为不同VPN网络中的客户端分配相同网段的IP地址。

仅S1720GW-E、S1720GWR-E、S1720X-E、S2720EI、S5720EI、S5720HI、S5720LI、S5720S-LI、S5720S-SI、S5720SI、S5730S-EI、S5730SI、S6720EI、S6720LI、S6720S-EI、S6720S-LI、S6720SI和S6720S-SI支持该步骤。

### （可选）配置不参与自动分配的IP地址

#### 背景信息

地址池中可以排除某些有特殊需求不能参与自动分配的IP地址。

> 说明
>
> 命令 `gateway-list` 或 `dhcp server gateway-list` 配置的网关地址不需要配置，设备自动将其加入到不参与自动分配的IP地址列表。
> 服务器连接客户端的接口地址不需要配置，地址分配时，设备自动将其置为冲突（Conflict）状态。

#### 操作步骤

执行命令 `system-view`，进入系统视图。
执行命令 `ip pool ip-pool-name`，进入全局地址池视图。
执行命令 `excluded-ip-address start-ip-address [ end-ip-address ]`，配置地址池中不参与自动分配的IP地址。

缺省情况下，未配置地址池中不参与自动分配的IP地址。

多次执行该命令，可以配置多个不参与自动分配的IP地址。
例如，将192.168.1.10配置为不参与自动分配的地址：

```shell
[HUAWEI-ip-pool-global_f1] excluded-ip-address 192.168.1.10
```

后续处理

若要扩大不参与自动分配的IP地址范围，可以再次配置本功能。若要缩小不参与自动分配的IP地址范围，则可以通过命令 `undo dhcp server excluded-ip-address` 或 `undo excluded-ip-address` 去除其中部分地址。

### （可选）配置客户端的网关地址

#### 背景信息

DHCP服务器上配置客户端的网关地址后，客户端会获取到服务器分配的网关地址，并自动生成到该网关地址的缺省路由。之后，客户端才能访问非本网段内的主机。如果DHCP服务器同时配置了Option121给客户端分配无分类静态路由，客户端只会据此生成路由，不会自动生成到网关的缺省路由。为了对流量进行负载分担或提高网络的可靠性，可以执行此任务配置多个网关。每个地址池最多可以配置8个网关地址。

在VRRP+DHCP的综合场景中，如果VRRP备份组设备作为DHCP服务器，则需要执行此任务配置客户端的网关地址为备份组虚拟IP地址。

如果DHCP服务器与DHCP客户端处于同一网段，并且DHCP服务器作为客户端网关时，可以不配置此任务。

#### 操作步骤

执行命令 `system-view`，进入系统视图。
执行命令 `ip pool ip-pool-name`，进入全局地址池视图。
执行命令 `gateway-list ip-address &<1-8>`，配置DHCP Client的网关地址。

缺省情况下，IP地址池视图下未配置网关地址。

### （可选）配置DHCP服务器分配的DNS信息

#### 背景信息

当管理员希望客户端的DNS配置通过DHCP方式动态获取时，可以执行此任务。

#### 操作步骤


1. 执行命令system-view，进入系统视图。
2. 执行命令ip pool ip-pool-name，进入全局地址池视图。
3. 执行命令dns-list ip-address &<1-8>，配置DHCP Client使用的DNS Server的IP地址。
> 缺省情况下，未配置DNS Server地址。
> 每个地址池最多可以配置8个DNS Server地址。
4. 执行命令domain-name domain-name，配置为DHCP Client分配的域名后缀。
> 缺省情况下，未配置DNS域名后缀。
