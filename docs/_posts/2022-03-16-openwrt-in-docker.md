---
layout: single
title:  "使用Docker搭建OpenWrt旁路由"
author: 剧烈燃烧的CO2
header:
  #image: https://simonhux.com/p/docker-%E5%AE%89%E8%A3%85-openwrt/bg.png
  overlay_color: "#333"
categories: 
 - 网络
 - OpenWrt
---

**本方案原理：**使用docker的macvlan网络为容器虚拟出一个二层网卡，作为容器物理网卡，和虚拟机桥接网络类似

**固件默认账号：**`root` **密码：**`password` 
{: .notice}

**传送门:** [镜像仓库链接](https://hub.docker.com/r/raymondwong/openwrt_r9)，[我在用的网站](https://www.sjlx.win/auth/register?code=tIim)
{: .notice--info}

方案一（命令行）:  
```shell
ip link set [修改为本地网卡名称，如eth0] promisc on 
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=[修改为本地网卡名称，与上一步保持一致] macnet
docker run --restart always -d --network macnet --privileged -v /lib/modules:/lib/modules raymondwong/openwrt_r9:autobuild-21.12.6-arm64
```
>等待容器进入`running`状态后，打开[http://192.168.1.254](http://192.168.1.254)，测试部署是否成功
{: .text-left}

方案二(docker compse):  
```shell
ip link set [修改为本地网卡名称，如eth0] promisc on 
mkdir openwrt&&cd openwrt 
将下面代码复制到文件 'docker-compose.yaml', 并将 `driver_opts: parent`的值改为需要桥接的网口
在创建'docker-compose.yaml'文件的同一目录下运行命令`docker-compose up -d` 
```
>等待容器进入`running`状态后，打开[http://192.168.1.254](http://192.168.1.254)，测试部署是否成功
  
**docker-compose.yaml：**
```yaml
version: '2'
services:
  openwrt:
    image: raymondwong/openwrt_r9:21.2.1-arm64
    container_name: openwrt_r9
    privileged: true
    restart: always
    networks:
      openwrt_macnet:
        ipv4_address: 192.168.1.254

networks:
  openwrt_macnet:
    driver: macvlan
    driver_opts:
      parent: en0
    ipam:
      config:
        - subnet: 192.168.1.0/24
          ip_range: 192.168.1.128/25
          gateway: 192.168.1.1
```

备注：
>此教程方案为单网卡方案，如果遇到某些插件工作不正常，可以尝试添加2个虚拟网卡，模拟真正的路由   