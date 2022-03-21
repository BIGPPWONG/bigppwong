---
author: 剧烈燃烧的CO2
title:  "pfSense配置OpenNAT的方法"
comments: true
header:
  overlay_color: "#333"
categories: 
 - 网络
---

pfSense在移动宽带DoubleNAT内网IP环境下配置Xbox为OpenNAT的方法，也适用于PS等其它需要开放类型NAT的设备

有兴趣可以了解下[什么是Easy NAT（Endpoint-Independent NAT mapping）？](https://tailscale.com/blog/how-nat-traversal-works/)

要让Xbox获取开放类型的NAT，要解决两个问题
: 1. NAT mapping
2. 开启防火墙端口

图片看不清的话右键图片新标签页打开
{: .notice--info}

解决NAT mapping
: 默认情况下，pfSense会对源地址端口做一个随机操作，导致每一次连接源端口都不同
- 打开**Firewall > NAT**,**Outbound**
- 找到**Outbound NAT Mode**，选择**Hybrid Outbound NAT**
- 点击**Save**保存
![](/assets/post-images/pfsense-opennat-1.png)
- 找到下面**Mappings**，添加一条规则
- 按下图配置，**Source**中的IP修改成Xbox的内网IP，掩码选32，**Translation**那里一定要勾上**Static Port**
![](/assets/post-images/pfsense-opennat-2.png)
- 点击**Save**保存，如果出现绿色的**Apply Changes**提示，也要点击Apply

这里使用`1:1 NAT`也可以达到同样效果，但是增加了多余的流量和降低了安全性
{: .notice--info}

解决自动开启防火墙端口
: 这个应该大家都很熟悉了，就是配置UPNP，但pfSense自带的miniupnp有个坑，当WAN口地址为内网地址的时候会拒绝工作，不太符合咱们国内环境，下面说说绕过的方法
- 打开**Services > UPnP & NAT-PMP**
- 按下图配置**UPnP & NAT-PMP Settings**，**Override WAN address**随便填入一个**公网IP**
![](/assets/post-images/pfsense-upnp-1.png)
- **不要**开启STUN
- **UPnP Access Control Lists**中，**ACL Entries**填入
```
allow 1024-65535 192.168.1.151/32 1024-65535
```
把`192.168.1.151`修改成XBox的IP
![](/assets/post-images/pfsense-upnp-2.png)
- 点击**Save**保存，如果出现绿色的**Apply Changes**提示，也要点击Apply
- 在Xbox上运行NAT测试，然后pfSense打开**Status > UPnP & NAT-PMP**看看是否成功添加映射

这里相当于欺骗了miniupnp我们有公网IP，实测XBOX能成功变为开放NAT类型，但同时miniupnp会将这个随便设置的IP回复给客户端设备，这个是否有影响尚不明确
{: .notice--info}

