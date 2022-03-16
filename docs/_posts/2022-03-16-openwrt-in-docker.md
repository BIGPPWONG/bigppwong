layout: post
title: "使用Docker运行OpenWrt旁路由"
date: 2022-03-16 15:07:00 -0000
categories: 网络

`https://www.sjlx.win/auth/register?code=tIim`   
Method ONE(recommended):  
step1: `ip link set [net interface] promisc on`  
step2: `docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=[net interface] macnet`  
step3: `docker run --restart always -d --network macnet --privileged -v /lib/modules:/lib/modules raymondwong/openwrt_r9:autobuild-21.12.6-arm64 /sbin/init`  
step4: go to `http://192.168.1.254` Enjoy!  

Method TWO(docker compse):  
step1:`ip link set [interface] promisc on`  
step2: `mkdir openwrt&&cd openwrt`   
step3: copy below code to a file named 'docker-compose.yaml' in current directory, and modify `option parent`to your network interface  
step4: `docker-compose up -d`  
step5: go to `http://192.168.1.254` Enjoy!  
`user: root password:password`   
```
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

