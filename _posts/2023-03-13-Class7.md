---
layout: post
title: 第八课
date: 2023-03-13
Author: 
categories: 
tags: [eNSP]
comments: false
toc: true
---

> VLANIF（Virtual Local Area Network Interface），是华为交换机的一个配置项，应用于三层交换机。这是一种逻辑接口，物理上不存在。
> 在配置好二层vlan后，三层交换机上使用vlanif命令建立对应vlan的路由，配置完成后，可以实现VLAN之间的通信。
> From: [https://blog.csdn.net/qq\_51577576/article/details/122140073](https://blog.csdn.net/qq_51577576/article/details/122140073)

# 创建 vlan 并配置端口

[前往第三课](../third-class)

# 进入 vlanif

`interface vlanif <vid>` 进入对应的 vlanif。

```shell
[Huawei]interface vlanif 1
[Huawei-Vlanif1]
```

# 配置 ip

给 vlanif 配置 ip，以开启路由功能。

```shell
[Huawei-Vlanif1]ip address 192.168.1.1 24
[Huawei-Vlanif1]
```
