---
title: docker安装
comments: true
categories:
  - 软件安装
tags:
  - docker
aside: true
highlight_shrink: false
cover: /image/docker.png
abbrlink: 327177db
date: 2022-10-15 20:40:02
---


## docker安装(centos)

docker引擎安装官网地址：[Install Docker Engine on CentOS | Docker Documentation](https://docs.docker.com/engine/install/centos/)

### 手动安装

```bash
# 卸载旧版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 安装依赖工具
sudo yum install -y yum-utils
# 添加docker官方源
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
# 查询可安装版本
# yum list docker-ce --showduplicates | sort -r
# 安装特定版本
# sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io docker-compose-plugin
# 安装docker
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 启动docker服务
systemctl start docker
# 添加开机自启动
systemctl enable docker
```

### 脚本安装

```bash
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

## 阿里云镜像加速

登录阿里云，进入容器镜像服务->镜像加速器,复制代码在系统中执行即可
![](https://cdn.jsdelivr.net/gh/Li-Changwu/image/redis/20221015211053.png)

## docker卸载

1. 卸载 Docker 引擎、CLI、Containerd 和 Docker Compose 包：

   ```bash
   sudo yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

2. 主机上的映像、容器、卷或自定义配置文件不会自动删除。删除所有映像、容器和卷：

   ```b
   sudo rm -rf /var/lib/docker
   sudo rm -rf /var/lib/containerd
   ```

   > 您必须手动删除任何已编辑的配置文件。
