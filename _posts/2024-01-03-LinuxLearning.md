---
layout: post
title: Linux基于宝塔面板的WordPress安装及发布教程
date: 2024-01-03
Author: 
categories: 
tags: [Linux]
comments: false
toc: true
---


# Centos安装宝塔面板

## Centos安装命令

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

# 使用宝塔面板安装WordPress图文教程

> 宝塔面板安装WordPress有两种方法：

1. 自己手动安装（推荐）
2. 宝塔后台一键部署

手动安装是推荐的安装方式，因为一键部署的WordPress版本不是最新的，而且自己上传的文件比较放心。

## 第一步，上传WordPress安装包

手动上传安装包（推荐）

上传WoredPress安装包有两种方式，一种是本地上传，另外一种是直接服务器下载。

如果你服务器是国内的，那么请选择手动上传，因为远程下载可能网络不通畅会失败。

WordPress新版安装包下载地址：[WordPress 新版中文简体版本地下载](https://cn.wordpress.org/download/ "WordPress官网下载")

![1](https://picss.sunbangyan.cn/2024/01/03/1b561b5315b8c73e747fb5950c92a31a.jpeg)

服务器远程下载的话，就在宝塔面板里面点击：网站 → 网站根目录文件夹 → 远程下载，然后填写上下面的网址，点击确定。

<https://cn.wordpress.org/latest-zh_CN.zip>

上传或者下载完毕WordPress安装包后，鼠标移动到安装包那一行，点击解压按钮。如下图：

![2](https://picst.sunbangyan.cn/2024/01/03/e68df1d28b6d75008a1c3883f916a978.jpeg)

解压窗口直接点击“解压”

![3](https://picdl.sunbangyan.cn/2024/01/03/559b4a8402065890e79805340193c62e.jpeg)

很快就会解压完毕，出现一个名为WordPress的文件夹，点击进入该文件夹

![4](https://picdm.sunbangyan.cn/2024/01/03/7fb0e0a096380de5984e8f549638950e.jpeg)

进入WordPress文件夹后，点击文件名前面的勾选框全选所有文件夹和文件。

然后点击右上角的剪切按钮，步骤操作如下图：

![5](https://picst.sunbangyan.cn/2024/01/03/039c9353b5b33fc3cfd432f6f9c9cfbb.jpeg)

接着点击你网址的文件夹回到根目录，点击右上角的粘贴所有按钮，如下图：

![6](https://picdm.sunbangyan.cn/2024/01/03/8d6fd792992344505a479124e23d2a7c.jpeg)

最后你宝塔面板文件界面就是下图这个样子的。

其中WordPress文件夹和安装包文件可以删除，404.html和index.html也可以删除。不删除也没关系。

![7](https://picss.sunbangyan.cn/2024/01/03/ba6516f2e4be0fbba838f7c007c2f0ca.jpeg)

接下来我们就可以开始搭建网站了。

## 一键部署安装包

一键部署安装包是宝塔后台提供的功能，相对来说少了自己手动添加网站和上传WordPress的步骤，不过提供的WordPress版本可能不是最新的。

使用宝塔一键部署WordPress需要没有在宝塔里面添加你需要安装的网站，如果已经添加了，请先删除。

进入宝塔后台后，选择左边菜单栏的【网站】，然后点击【添加站点】。

点击后面的【一键部署】找到WordPress。

![8](https://picst.sunbangyan.cn/2024/01/03/c4fa9406ac71e20ccd1c205c3d7cc6f7.jpeg)

![9](https://picdl.sunbangyan.cn/2024/01/03/9484ef4e987398505fd7177eba5ffe3f.jpeg)

很快，就会从宝塔服务器下载好WordPress的安装文件到你网站文件夹下面，出现下图，保存好出现的资料，然后打开网站地址。

![10](https://picdm.sunbangyan.cn/2024/01/03/9dfb24b58b39f3d840a2a4303b9b3481.jpeg)

打开这个访问地址后，你就会出现下面第二步，搭建WordPress程序网站的界面，跟着操作即可。

## 第二步，搭建WordPress程序网站

现在打开你绑定的域名，会出现下图界面。点击“现在就开始”

![11](https://picst.sunbangyan.cn/2024/01/03/c34764113d93cc19f133d619d2ade4bb.jpeg)

在下图界面输入对应的数据库名，数据库用户名和数据库密码，然后点击提交。

![12](https://picdm.sunbangyan.cn/2024/01/03/c47f8b6c74efce0c83f604d0431e608a.jpeg)

数据库没错的话就会出现下图界面，点击现在安装。

![13](https://picss.sunbangyan.cn/2024/01/03/1625f4f04b684a9271dbe4350b87fce2.jpeg)

很快就会出现下图界面，要你自己设置站点标题、用户名、密码和邮箱信息。设置完毕后点击“安装WordPress”

成功，可以登录了。

![14](https://picst.sunbangyan.cn/2024/01/03/11d47a0896251bc0dfae6f941ebc3b05.jpeg)

![15](https://picss.sunbangyan.cn/2024/01/03/2a45e674a4ffd1eeaef8c9708fca3c0c.jpeg)