---
layout: post
title: 第五课
date: 2023-03-01
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

GARP（Generic Attribute Registration Protocol）协议主要用于建立一种属性传递扩散的机制，以保证协议实体能够注册和注销该属性。GARP作为一个属性注册协议的载体，可以用来传播属性。将GARP协议报文的内容映射成不同的属性即可支持不同上层协议应用。

GVRP（GARP VLAN Registration Protocol）是GARP的一种应用，用于注册和注销VLAN属性。

GARP协议通过目的MAC地址区分不同的应用。在IEEE Std 802.1Q中将01-80-C2-00-00-21分配给VLAN应用，即GVRP。

# GVRP 动态 vlan

## 开启 GVRP

在 system-view 中输入 `gvrp`

```shell
[Huawei]gvrp
[Huawei]
```

## 为端口配置 GVRP

进入配置了 trunk 的端口，输入 `gvrp`
```shell
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/1]port link-type trunk
[Huawei-GigabitEthernet0/0/1]port trunk allow-pass vlan all
[Huawei-GigabitEthernet0/0/1]gvrp
[Huawei-GigabitEthernet0/0/1]
```

# 查看 vlan

## 简单查看 vlan

```shell
<Huawei>display vlan summary
static vlan:
Total 2 static vlan.
  1 20 

dynamic vlan:
Total 1 dynamic vlan.
  10 

reserved vlan:
Total 0 reserved vlan.
<Huawei>
```

## 以表格形式查看 vlan

```shell
<Huawei>display vlan
The total number of vlans is : 3
--------------------------------------------------------------------------------
U: Up;         D: Down;         TG: Tagged;         UT: Untagged;
MP: Vlan-mapping;               ST: Vlan-stacking;
#: ProtocolTransparent-vlan;    *: Management-vlan;
--------------------------------------------------------------------------------

VID  Type    Ports                                                          
--------------------------------------------------------------------------------
1    common  UT:GE0/0/1(U)      GE0/0/3(D)      GE0/0/4(D)      GE0/0/5(D)      
                GE0/0/6(D)      GE0/0/7(D)      GE0/0/8(D)      GE0/0/9(D)      
                GE0/0/10(D)     GE0/0/11(D)     GE0/0/12(D)     GE0/0/13(D)     
                GE0/0/14(D)     GE0/0/15(D)     GE0/0/16(D)     GE0/0/17(D)     
                GE0/0/18(D)     GE0/0/19(D)     GE0/0/20(D)     GE0/0/21(D)     
                GE0/0/22(D)     GE0/0/23(D)     GE0/0/24(D)                     

10   dynamic TG:GE0/0/1(U)                                                      

20   common  UT:GE0/0/2(U)                                                      

             TG:GE0/0/1(U)                                                      


VID  Status  Property      MAC-LRN Statistics Description      
--------------------------------------------------------------------------------

1    enable  default       enable  disable    VLAN 0001                         
10   enable  default       enable  disable    VLAN 0010                         
20   enable  default       enable  disable    VLAN 0020                         
<Huawei>
```
