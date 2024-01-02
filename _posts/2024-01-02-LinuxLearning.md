---
layout: post
title: Linux宝塔面板安装教程
date: 2024-01-02
Author: 
categories: 
tags: [Linux]
comments: false
toc: true
---


# Centos安装宝塔面板

## 安装要求：

- 内存：512M以上，推荐768M以上（纯面板约占系统60M内存）
- 硬盘：300M以上可用硬盘空间（纯面板约占20M磁盘空间）
- 系统：CentOS 7.1+ (Ubuntu16.04+.、Debian9.0+)，确保是干净的操作系统，没有安装过其它环境带的Apache/Nginx/php/MySQL/pgsql/gitlab/java（已有环境不可安装）
- 架构：x86_64（主流服务器均是此架构），ARM不完整兼容（面板环境安装慢，部分软件可能安装不上）

## Centos安装命令：

```shell
yum install -y wget && wget -O install.sh https://download.bt.cn/install/install_6.0.sh && sh install.sh 12f2c1d72
```

[![piji5kt.jpg](https://s11.ax1x.com/2024/01/02/piji5kt.jpg)](https://imgse.com/i/piji5kt)

此处询问你“你现在想安装宝塔控制面板吗” 直接输入y按下回车继续安装

[![pijFknJ.jpg](https://s11.ax1x.com/2024/01/02/pijFknJ.jpg)](https://imgse.com/i/pijFknJ)

出现下面界面则表示已经安装成功了，并且要记住宝塔内外网面板地址和账号密码，后面登录宝塔面板会用到

[![pijFE7R.jpg](https://s11.ax1x.com/2024/01/02/pijFE7R.jpg)](https://imgse.com/i/pijFE7R)

## 登录宝塔面板

使用之前给出的内部面板网址访问宝塔面板，输入第一次配置自动生成的账号密码

[![pijFmh6.jpg](https://s11.ax1x.com/2024/01/02/pijFmh6.jpg)](https://imgse.com/i/pijFmh6)

[![pijFu9K.jpg](https://s11.ax1x.com/2024/01/02/pijFu9K.jpg)](https://imgse.com/i/pijFu9K)

出现上图就为配置成功

---

# 配置WEB服务器

对于服务器的web部署来说很多人都熟悉lamp和lnmp两种方式，首先说一下字面上看其实就是一个字母差别，其实也是web环境中核心的环境差别。

LAMP：Linux + Apache + MySQL+php的组合方式

LNMP：Linux + Nginx + MySQL+php的组合方式

LAMP和LNMP最主要的区别在于：一个使用的是Apache，一个使用的是Nginx。

Linux 开源免费软件，作为网站的操作系统 
Apache/Nginx Web服务器软件 
MySQL 多线程多用户的数据库管理系统，用来存放数据 
PHP 服务器端的应用程序软件，快速执行动态网页

LAMP：使用的是Apache，Apache是世界是用排名第一的Web服务器软件，其几乎可以在所有广泛使用的计算机平台上运营，由于其跨平台和安全性被广泛使用，是最流行的Web服务端软件之一。相比于nginx，apache有些臃肿，内存和CPU开销较大，性能上有损耗，nginx对于静态文件的响应能力远高apache。 Apache是负载PHP的最佳选择，如果流量很大的话，可以使用nginx来负载非PHP的Web请求。。

LNMP：使用的是Nginx，Nginx是一款高性能额Http和反向代理服务器，也是一个AMAP/POP3/SMTP服务器。nginx使用资源更少，支持更多并发连接，效率更高，作为负载均衡服务器。nginx即可对内进行支持，也可对外进行服务，安装简单。

总之：

1、LNMP方式的优点：占用VPS资源较少，Nginx配置起来也比较简单，利用fast-cgi的方式动态解析PHP脚本。缺点：php-fpm组件的负载能力有限，在访问量巨大的时候，php-fpm进程容易僵死，容易发生502 bad gateway错误。

2、基于 LAMP 架构设计具有成本低廉、部署灵活、快速开发、安全稳定等特点，是 Web 网络应用和环境的优秀组合。若是服务器配置比较低的个人网站，当然首选 LNMP 架构。当然，在大流量的时候。把Apache和Nginx结合起来使用，也不失为一个不错选择。

## 一键安装及部署LNMP

首次使用宝塔面板推荐安装一组套件，对于刚接触Linux环境部署的博友们，推荐使用LNMP，采用极速安装、一键安装的方式

[![pijF3Bd.jpg](https://s11.ax1x.com/2024/01/02/pijF3Bd.jpg)](https://imgse.com/i/pijF3Bd)

- 出现下面界面说明正在安装，根据服务器的配置不同，安装所需要的时间也不一样

[![pijFJAI.png](https://s11.ax1x.com/2024/01/02/pijFJAI.png)](https://imgse.com/i/pijFJAI)

- 点击消息列表，出现下面界面，说明套件已经安装成功了

[![pijFBuQ.png](https://s11.ax1x.com/2024/01/02/pijFBuQ.png)](https://imgse.com/i/pijFBuQ)

- 在软件商店中选择已安装可以查看安装的详细信息

![6](https://picst.sunbangyan.cn/2024/01/02/39b8ccc3f53ab3d2dbe4bfe80c26edf5.jpeg)

- 更改面板端口

将端口更改为不常用的端口。

- 绑定域名

你可以绑定一个域名绑定完域名后只能通过你绑定的域名来访问面板。

- 绑定ip

如果你有固定的ip，你可绑定ip访问，绑定了ip访问你只能通过绑定得这个ip进行访问。如果你是家用电脑就不要绑定ip了，因为家用电脑的ip是动态的。这就会造成ip发生改变面板访问不了。

- 更改默认的面板用户和密码

更改宝塔安装完成时的默认用户名和密码，设置一个自己能记住的用户名和密码，密码不要太简单了。

![7](https://picst.sunbangyan.cn/2024/01/02/d78ca17d25fb774f55d8dc3a48d99a08.jpeg)

- 设置面板端口
内网访问面板时使用到的端口

- 自定义安全入口名称
使用内网IP访问时的安全入口
    -- 例如:192.168.200.10:13265/homepage

![8](https://picss.sunbangyan.cn/2024/01/02/2cf0d39293980adcedea80627b907f7d.jpeg)

---

# 宝塔面板 FTP 安装与使用教程

## 创建 FTP 及 用户

![10](https://picdl.sunbangyan.cn/2024/01/02/73f15ec850d5907bf7d65e6269f103b3.jpeg)

![11](https://picss.sunbangyan.cn/2024/01/02/5acf1200bd0117eb48bf34f643fbd7e4.jpeg)


## 确保端口放行

![12](https://picst.sunbangyan.cn/2024/01/02/9ef81c4257bfe10c0ddc52f24f1c1637.jpeg)

## 测试客户端连接FTP服务器

- Windows下使用文件管理器，在路径栏上写入服务器FTP地址，输入创建时设置的账号密码即可登录服务器

![13](https://picst.sunbangyan.cn/2024/01/02/183de95775486107b4afa7911b8ca62a.jpeg)


# Linux系统利用宝塔面板Tomcat环境教程

## 安装Tomcat

- 在软件商店中搜索Tomcat并下载（以下为安装完成效果）

![14](https://picss.sunbangyan.cn/2024/01/02/8d779bbadfd90633152a2fe03b38c56c.jpeg)

- 确保Tomcat服务在运行中

![15](https://picdl.sunbangyan.cn/2024/01/02/cb76038d035f06af12b0772e7630e510.jpeg)

## 创建Tomcat项目

- 在项目管理中点击添加项目

![16](https://picdl.sunbangyan.cn/2024/01/02/47875b1cc8141aef4c4f8e83f9a16566.jpeg)

- 项目类型选择独立环境

- 项目版本选择刚刚安装的Tomcat版本

- 项目域名写为你想要的域名（例如：test.com） 

- 项目端口选择你需要的端口（注意不要与其他已用端口重复，并且在规则中选择端口放行）

![17d](https://picst.sunbangyan.cn/2024/01/02/d06ed4fcdb81b5c1ecb425f4077ab3a0.jpeg)

- 出现以下情况即为配置成功

![18](https://picdm.sunbangyan.cn/2024/01/02/c4dc8cfdaf20e88596c951d86550d7a9.jpeg)

---

# 宝塔Linux面板添加网站站点

## 添加站点

- 点击宝塔页面左侧的网站。再点击PHP项目下的添加站点既可以添加。

![3ca7349612ba478ea88e341ca54d2ee5](https://picss.sunbangyan.cn/2024/01/02/bd86f93ce823165727f718e62dda6ec1.jpeg)

## 点击添加站点后，会弹出详情窗口，输入详细配置。

- 有域名填域名，没有就填写服务器ip地址。

- FTP,数据库就根据自己的需求进行选择。

![30e33d8f59ee49d2af681d8d2d5b51e9](https://picdm.sunbangyan.cn/2024/01/02/1a20f822e73ce946d8cc3bb5084c0ffe.jpeg)

- 创建完成

## 点击创建的网络的根目录

- 再点击左侧的文件，根据创建站点内的文件根目录找到自己的文件夹，再将打包好的dist文件夹的内容全部放进去即可。
- 上传完成后如图。最后就可以自己去打开自己的网站了。

![19](https://picst.sunbangyan.cn/2024/01/02/ff4b0dcef86bcd1754cb34d86f904af4.jpeg)

![20](https://picss.sunbangyan.cn/2024/01/02/2fc2f651e4c653bc8757dbd9b1e1bba6.jpeg)