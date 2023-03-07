---
layout: post
title: 第四课
date: 2023-02-26
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

# 使用 `display mac-address` 查看 mac 地址表

```
[Huawei]display mac-address
MAC address table of slot 0:
-------------------------------------------------------------------------------
MAC Address    VLAN/       PEVLAN CEVLAN Port            Type      LSP/LSR-ID  
               VSI/SI                                              MAC-Tunnel  
-------------------------------------------------------------------------------
5489-9817-6f6f 1           -      -      Eth0/0/1        dynamic   0/-         
5489-9873-1c5a 1           -      -      Eth0/0/2        dynamic   0/-         
5489-9868-08c5 1           -      -      Eth0/0/3        dynamic   0/-         
-------------------------------------------------------------------------------
Total matching items on slot 0 displayed = 3 

[Huawei]
```

# 使用 `port-security enable` 开启端口安全性

```
[Huawei-Ethernet0/0/1]port-security enable
[Huawei-Ethernet0/0/1]
```

# 使用 `port-security mac-address sticky` 将 mac 与端口设置绑定

```
[Huawei-Ethernet0/0/1]port-security mac-address sticky
[Huawei-Ethernet0/0/1]
```

# 使用 `port-security mac-address sticky [mac] vlan [vid]` 将 mac 与 vlan 绑定

```
[Huawei-Ethernet0/0/1]port-security mac-address sticky 5489-9817-6f6f vlan 1
[Huawei-Ethernet0/0/1]
```

# 再查看 mac 地址表

```
[Huawei-Ethernet0/0/1]display mac-address
MAC address table of slot 0:
-------------------------------------------------------------------------------
MAC Address    VLAN/       PEVLAN CEVLAN Port            Type      LSP/LSR-ID  
               VSI/SI                                              MAC-Tunnel  
-------------------------------------------------------------------------------
5489-9817-6f6f 1           -      -      Eth0/0/1        sticky    -           
-------------------------------------------------------------------------------
Total matching items on slot 0 displayed = 1 

[Huawei-Ethernet0/0/1]
```

# ARP

## `arp -a` 或 `arp -g`

用于查看高速缓存中的所有项目，`-a` 和 `-g` 参数的结果是一样的。
