---
layout: post
title: 第十课
date: 2023-03-20
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

单臂路由（router-on-a-stick）是指在路由器的一个接口上通过配置子接口（或“逻辑接口”，并不存在真正物理接口）的方式，实现原来相互隔离的不同VLAN（虚拟局域网）之间的互联互通。

# 路由器的作用

路由器是一种网络设备，它可以将来自不同网络的数据包进行转发和交换，以实现网络之间的通信。它的主要作用包括：

1. 网络连接：路由器可以将多个局域网或广域网连接在一起，实现不同网络之间的通信。

2. 数据转发：路由器可以根据目的地址将数据包从一个网络转发到另一个网络。

3. 网络分割：路由器可以将一个大的网络分割成多个子网，以提高网络性能和安全性。

4. 网络管理：路由器可以对网络流量进行管理和控制，包括限制带宽、过滤恶意流量等。

5. 安全防护：路由器可以提供一些基本的安全防护功能，如防火墙、NAT等，以保护网络安全。

总之，路由器是网络中不可或缺的设备，它可以帮助我们构建一个高效、安全的网络环境。

# 单臂路由实现不同 VLAN 之间通信的原理

路由器重新封装 MAC 地址，转换 VLAN 标签。将路由器的 F0 接口进行逻辑划分：分为 F0.1 和 F0.2 设置网关分别为主机对应IP地址网段。

# 单臂路由的缺点

1. “单臂”为网络骨干链路，容易形成网络瓶颈。
2. 子接口依然依托于物理接口，应用不灵活。
3. VLAN 间转发需要查看路由表，严重浪费设备资源。

# 路由器的配置

## 进入子接口

```shell
[Huawei]int g0/0/0.10
[Huawei-GigabitEthernet0/0/0.10]
```

> 建议子端口 id 与 vlan 一致

## 配置 vlan 封装结构

```shell
[Huawei-GigabitEthernet0/0/0.10]dot1q termination vid 10
[Huawei-GigabitEthernet0/0/0.10]
```

## 配置 ip 地址

```shell
[Huawei-GigabitEthernet0/0/0.10]ip address 192.168.10.254 24
[Huawei-GigabitEthernet0/0/0.10]
```

## 开启向下 arp 广播请求

```shell
[Huawei-GigabitEthernet0/0/0.10]arp broadcast enable
[Huawei-GigabitEthernet0/0/0.10]
```

# 交换机的配置

## 进入与路由器连接的端口

```shell
[Huawei]int g0/0/1
[Huawei-GigabitEthernet0/0/1]
```

## 配置 Trunk

```shell
[Huawei-GigabitEthernet0/0/1]port link-type trunk
[Huawei-GigabitEthernet0/0/1]port trunk allow-pass vlan all
[Huawei-GigabitEthernet0/0/1]
```
