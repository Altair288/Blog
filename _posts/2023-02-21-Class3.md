---
layout: post
title: 第三课 - 交换机VLAN
date: 2023-02-21
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 创建 VLAN

想要单独创建一个 vlan ，可以直接输 `vlan [vid]` 。

```shell
[Huawei]vlan 1
[Huawei-vlan1]quit
[Huawei]
```

想要一次创建多个 vlan ，可以输 `vlan batch [vids]` 。
如果需要连续创建，可以输 `vlan batch [vid_begin] to [vid_end]` 。

```shell
[Huawei]vlan batch 1 2
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei]vlan batch 1 to 10
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei]
```

# 查看 VLAN

可以通过 `display vlan summary` 来查看已存在 vlan 的信息。

```shell
[Huawei]display vlan summary
static vlan:
Total 4 static vlan.
  1 10 20 30 

dynamic vlan:
Total 0 dynamic vlan.

reserved vlan:
Total 0 reserved vlan.
[Huawei]
```

# 配置 VLAN

## 基于端口划分 VLAN

首先使用 `interface interface-type interface-number` 进入需要配置的端口的接口配置视图。

```shell
[Huawei]interface g0/0/1
[Huawei-GigabitEthernet0/0/1]
```

使用 `port link-type access` 将端口设置为 Access 模式，并用 `port default vlan vid` 将端口添加到某个 vlan 中。

```shell
[Huawei-GigabitEthernet0/0/1]port link-type access
[Huawei-GigabitEthernet0/0/1]port default vlan 10
[Huawei-GigabitEthernet0/0/1]
```

可以通过 `display vlan` 验证配置结果。

```shell
[Huawei]display vlan
The total number of vlans is : 4
--------------------------------------------------------------------------------
U: Up;         D: Down;         TG: Tagged;         UT: Untagged;
MP: Vlan-mapping;               ST: Vlan-stacking;
#: ProtocolTransparent-vlan;    *: Management-vlan;
--------------------------------------------------------------------------------

VID  Type    Ports                                                          
--------------------------------------------------------------------------------
1    common  UT:GE0/0/3(D)      GE0/0/4(D)      GE0/0/5(D)      GE0/0/6(D)      
                GE0/0/7(D)      GE0/0/8(D)      GE0/0/9(D)      GE0/0/10(U)     
                GE0/0/11(D)     GE0/0/12(D)     GE0/0/13(D)     GE0/0/14(D)     
                GE0/0/15(D)     GE0/0/16(D)     GE0/0/17(D)     GE0/0/18(D)     
                GE0/0/19(D)     GE0/0/20(D)     GE0/0/21(D)     GE0/0/22(D)     
                GE0/0/23(D)     GE0/0/24(D)                                     

10   common  TG:GE0/0/10(U)                                                     
20   common  UT:GE0/0/1(U)                                                      
             TG:GE0/0/10(U)                                                     

30   common  UT:GE0/0/2(U)                                                      

             TG:GE0/0/10(U)                                                     


VID  Status  Property      MAC-LRN Statistics Description      
--------------------------------------------------------------------------------

1    enable  default       enable  disable    VLAN 0001                         
10   enable  default       enable  disable    VLAN 0010                         
20   enable  default       enable  disable    VLAN 0020                         
30   enable  default       enable  disable    VLAN 0030                         
[Huawei]
```

## 基于 mac 地址划分 VLAN

使用 `vlan [vid]` 进入 vlan 视图，使用命令 `mac-vlan mac-address [mac-address]` 将 MAC 地址与该 VLAN 进行关联。

```shell
[Huawei]vlan 10
[Huawei-vlan10]mac-vlan 54-89-98-0D-7E-06
[Huawei-vlan10]
```

查看 MAC 地址与 VLAN 的关联。

```shell
[Huawei]display mac-vlan mac-address all
---------------------------------------------------
MAC Address     MASK            VLAN    Priority   
---------------------------------------------------
5489-980D-7E06  ffff-ffff-ffff  10      0

Total MAC VLAN address count: 1 

[Huawei]
```

配置端口到 `Hybrid` 模式，使用 `port hybrid untagged vlan [vid]` 允许传输 VLAN 的流量。

```shell
[Huawei]interface g0/0/1
[Huawei-GigabitEthernet0/0/1]port link-type hybrid
[Huawei-GigabitEthernet0/0/1]port hybrid untagged vlan 10
[Huawei-GigabitEthernet0/0/1]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-GigabitEthernet0/0/1]
```

### 可以使用 Port group 同时配置多个端口。

使用命令 `port-group port-group-name` 创建并进入端口组。
使用命令 `group-member interface-type interface-number to interface-type interface-number` 添加端口范围。

```shell
[Huawei]port-group port5-9
[Huawei-port-group-port5-9]group-member g0/0/5 to g0/0/9
[Huawei-port-group-port5-9]port hybrid untagged vlan 20 30
[Huawei-GigabitEthernet0/0/5]port hybrid untagged vlan 20 30
[Huawei-GigabitEthernet0/0/6]port hybrid untagged vlan 20 30
[Huawei-GigabitEthernet0/0/7]port hybrid untagged vlan 20 30
[Huawei-GigabitEthernet0/0/8]port hybrid untagged vlan 20 30
[Huawei-GigabitEthernet0/0/9]port hybrid untagged vlan 20 30
[Huawei-port-group-port5-9]mac-vlan enable
[Huawei-GigabitEthernet0/0/5]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-GigabitEthernet0/0/6]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-GigabitEthernet0/0/7]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-GigabitEthernet0/0/8]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-GigabitEthernet0/0/9]mac-vlan enable
Info: This operation may take a few seconds. Please wait for a moment...done.
[Huawei-port-group-port5-9]
```

# 设置 Trunk 端口并允许 VLAN

## 创建 Trunk 端口

使用命令 `interface interface-type interface-number` 进入相应端口的接口配置视图。

```shell
[Huawei]interface g0/0/1
[Huawei-GigabitEthernet0/0/1]
```

使用命令 `port link-type trunk` 将端口设置为 Trunk 模式，并使用命令 `port trunk allow-pass vlan { {vlan-id} [to vlan-id2] | all}` 设置允许 Trunk 传输的 VLAN 。

```shell
[Huawei-GigabitEthernet0/0/1]port link-type trunk
[Huawei-GigabitEthernet0/0/1]port trunk allow-pass vlan all
[Huawei-GigabitEthernet0/0/1]
```

## 移除 Trunk 允许的 VLAN

VLAN 1 是缺省 VLAN ，在实际工作中，出于安全性考虑，会根据需要在 Trunk 链路上移除 VLAN 1 。

使用 `undo port trunk allow-pass vlan { {vlan-id} [to vlan-id2] | all}` 移除允许的 VLAN 。

```shell
[Huawei-GigabitEthernet0/0/1]undo port trunk allow-pass vlan 1
[Huawei-GigabitEthernet0/0/1]
```

# 查看当前端口配置

使用命令 `display current-configuration` 查看所有端口的配置信息。

```shell
[Huawei]display current-configuration 
#
sysname Huawei
#
vlan batch 10 20 30
#
cluster enable
ntdp enable
ndp enable
#
drop illegal-mac alarm
#
diffserv domain default
#
drop-profile default
#
aaa
 authentication-scheme default
 authorization-scheme default
 accounting-scheme default
 domain default
 domain default_admin
 local-user admin password simple admin
 local-user admin service-type http
#
interface Vlanif1
#
interface MEth0/0/1
#
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 20
#
interface GigabitEthernet0/0/2
 port link-type access
 port default vlan 30
#
interface GigabitEthernet0/0/3
#
interface GigabitEthernet0/0/4
#
interface GigabitEthernet0/0/5
#
interface GigabitEthernet0/0/6
#
interface GigabitEthernet0/0/7
#
interface GigabitEthernet0/0/8
#
interface GigabitEthernet0/0/9
#
interface GigabitEthernet0/0/10
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
interface GigabitEthernet0/0/11
#
interface GigabitEthernet0/0/12
#
interface GigabitEthernet0/0/13
#
interface GigabitEthernet0/0/14
#
interface GigabitEthernet0/0/15
#
interface GigabitEthernet0/0/16
#
interface GigabitEthernet0/0/17
#
interface GigabitEthernet0/0/18
#
interface GigabitEthernet0/0/19
#
interface GigabitEthernet0/0/20
#
interface GigabitEthernet0/0/21
#
interface GigabitEthernet0/0/22
#
interface GigabitEthernet0/0/23
#
interface GigabitEthernet0/0/24
#
interface NULL0
#
user-interface con 0
user-interface vty 0 4
#
return
[Huawei]
````
