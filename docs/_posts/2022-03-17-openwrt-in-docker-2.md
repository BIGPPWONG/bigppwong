---
# layout: single
author: 剧烈燃烧的CO2
title:  "将编译好的OpenWrt镜像转换为Docker镜像"
comments: true
header:
  overlay_color: "#333"
categories: 
 - Docker
 - OpenWrt
---

将openwrt-x86-64-generic-rootfs.tar.gz系统文件打包成Docker镜像，以便进行旁路由部署

1. **获取OpenWrt系统文件**
- 自己编译系统文件
- 从[OpenWrt官方下载](https://downloads.openwrt.org/releases/21.02.2/targets/x86/64/openwrt-21.02.2-x86-64-rootfs.tar.gz)预编译文件  
2. **将下载下来的文件重命名为**`OpenWrt.tar.gz`  
3. **将下面代码保存为Dockerfile文件**
```
FROM scratch
ADD OpenWrt.tar.gz /
EXPOSE 80 443 22
CMD ["/sbin/init"]
```
4 . **构建镜像**  
```
docker build -t myopenwrt .
```  
理论上来说到这一步Docker镜像就已经制作完成了，但由于OP默认的防火墙和IP设置，直接运行镜像会导致网络崩溃，所以接下来还要修改一些配置  
4. **将下面几个文件保存到Dockerfile同一目录**