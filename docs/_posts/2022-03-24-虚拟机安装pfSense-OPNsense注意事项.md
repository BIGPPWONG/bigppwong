---
author: 剧烈燃烧的CO2
title:  "虚拟机安装pfSense/OPNsense注意事项"
comments: true
header:
  overlay_color: "#333"
categories: 
 - 网络
---

ESXi、PVE、HyperV环境下安装pfSense、OPNsense容易被忽略的配置造成奇怪的网络问题

最低硬件配置
: 虚拟机安装比裸机安装的配置要高一些，官方建议
: - 内存最低1GB
- 磁盘空间最低8GB

关闭网卡硬件加速
: 因为freeBSD驱动的原因，打开硬件加速有时候会导致防火墙错误block掉一些正常的流量
: pfSense配置方法
- 打开 **System > Advanced, Networking**这个配置页面
- 找到**Networking Interfaces**这一栏
- 勾选**Disable hardware checksum offload**
- 点击最下面**Save**保存
- 重启pfSense
![](https://docs.netgate.com/pfsense/en/latest/_images/screen_shot_2017-06-30_at_18.51.25.png)

: OPNsense配置方法
- 打开**Interfaces > Settings**
- 按下图修改配置
![OPNsense disable off-loading](https://docs.opnsense.org/_images/disableoffloading.png)

ESXi特定配置
: - 在ESXi下建议根据[ESXi安装FreeBSD官方教程](http://partnerweb.vmware.com/GOSIG/FreeBSD_11x.html)安装系统
- 虚拟机网卡选择`e1000`能获得最好兼容性，尤其是在使用QoS功能的适合
- 虚拟机网卡选择`vmx`类型能获得最好性能
- 如果磁盘出现问题，尝试将磁盘模式改为`IDE`
- pfSense可以选择在**System > Packages**安装**Open-VM-Tools**
- OPNsense可以选择在**System > Firmware > Plugins**安装**os-vmare**  

安装`Open-VM-Tools`、`os-vmare`可以获得更好的网络和磁盘性能，如果想通过web重启、关闭防火墙虚机，这两个包是必须的
{: .notice--info}

PVE（KVM）特定配置
: - `i440FX chipset`主板类型能完美工作
- `Q35 chipset`不支持，驱动硬件会有问题，属于FreeBSD和KVM的问题