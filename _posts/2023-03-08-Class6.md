---
layout: post
title: 第七课
date: 2023-03-08
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

Hybrid类型端口可以允许多个VLAN通过，可以接收和发送多个VLAN 报文，可以用于交换机的间连接也可以用于连接用户计算机。

Hybrid 接口处理 VLAN 帧的过程如下：
1. 收到一个二层帧，判断是否有 VLAN 标签。没有标签，则标记上 Hybrid 接口的 PVID，进行下一步处理；有标签，判断该 Hybrid 接口是否允许该 VLAN 的帧进入，允许则进行下一步处理，否则丢弃。
2. 当数据帧从 Hybrid 接口发出时，交换机判断 VLAN 在本接口的属性是 Untagged 还是 Tagged。如果是 Untagged，先剥离帧的 VLAN 标签，再发送；如果 Tagged，则直接发送帧。

# 设置端口类型为 hybrid

```shell
[Huawei-GigabitEthernet0/0/1]port link-type hybrid
[Huawei-GigabitEthernet0/0/1]
```
> 所有端口类型默认就是 hybrid，不用再配置。

# 设置 hybrid 的 pvid

从此端口接受到的 Untagged 帧会加入至 pvid 配置的 vlan。

```shell
[Huawei-GigabitEthernet0/0/1]port hybrid pvid vlan 1
[Huawei-GigabitEthernet0/0/1]
```
> 所有类型为 hybrid 的端口 pvid 默认是 1。

# 设置 untagged 的 vlan

untagged 配置的 vlan 的帧会从此端口以 Untagged 的方式发送。

```shell
[Huawei-GigabitEthernet0/0/1]port hybrid untagged vlan 1
[Huawei-GigabitEthernet0/0/1]
```
> 所有类型为 hybrid 的端口默认会为 untagged 配置 vlan 1。

# 设置 tagged 的 vlan

tagged 配置的 vlan 的帧会从此端口包含 vlan tag 发送。

```shell
[Huawei-GigabitEthernet0/0/1]port hybrid tagged vlan 1
[Huawei-GigabitEthernet0/0/1]
```
