---
layout: post
title: Docker coder-server容器更新
date: 2024-10-10
Author: 
categories: 
tags: [code-server]
comments: false
toc: true
---



# 更新docker中的code-server容器

本人使用docker的coder/code-server镜像进行部署，根据官方文档可知，code-server的用户数据与程序本体是分开的

![PixPin_2024-10-10_10-53-12](https://share.altair288.eu.org/easyimage/i/2024/10/10/hdhwge.jpg)

https://coder.com/docs/code-server/upgrade

避免手动安装可能造成的错误，建议直接下载.deb使用命令行进行安装，例如：

dpkg -i code-server_4.11.0_amd64.deb #根据自身情况判断是否需要加sudo权限，官方镜像使用debian发行版因此用deb类型安装，其他系统会有区别

```shell
curl -fOL https://github.com/coder/code-server/releases/download/v$VERSION/code-server_${VERSION}_amd64.deb
sudo dpkg -i code-server_${VERSION}_amd64.deb
sudo systemctl enable --now code-server@$USER
```

安装完后重启容器即可，例如

```shell
docker restart youtContainerName
```

本人使用以上方案是因为在容器内进行项目的编写，一般容器的用法是容器仅提供单一功能，使用中不改变容器内结构的，因此可以定期更新镜像进行更新。有需要的小伙伴可以进入官方docker hub镜像,学习Updating Info章节内容：

Most of our images are static, versioned, and require an image update and container recreation to update the app inside. With some exceptions (ie. nextcloud, plex), we do not recommend or support updating apps inside the container. Please consult the Application Setup section above to see if it is recommended for the image.

Below are the instructions for updating containers:
Via Docker Compose

```shell
    Update all images: docker-compose pull
        or update a single image: docker-compose pull code-server
    Let compose update all containers as necessary: docker-compose up -d
        or update a single container: docker-compose up -d code-server
    You can also remove the old dangling images: docker image prune
```

Via Docker Run

```shell
    Update the image: docker pull lscr.io/linuxserver/code-server:latest
    Stop the running container: docker stop code-server
    Delete the container: docker rm code-server
    Recreate a new container with the same docker run parameters as instructed above (if mapped correctly to a host folder, your /config folder and settings will be preserved)
    You can also remove the old dangling images: docker image prune
```

Via Watchtower auto-updater (only use if you don't remember the original parameters)

```shell
    Pull the latest image at its tag and replace it with the same env variables in one run:


    docker run --rm \
    -v /var/run/docker.sock:/var/run/docker.sock \
    containrrr/watchtower \
    --run-once code-server

    You can also remove the old dangling images: docker image prune
```

Note: We do not endorse the use of Watchtower as a solution to automated updates of existing Docker containers. In fact we generally discourage automated updates. However, this is a useful tool for one-time manual updates of containers where you have forgotten the original parameters. In the long term, we highly recommend using Docker Compose.
Image Update Notifications - Diun (Docker Image Update Notifier)

```shell
    We recommend Diun for update notifications. Other tools that automatically update containers unattended are not recommended or supported.
```
