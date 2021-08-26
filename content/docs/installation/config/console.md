---
title: "配置esxi控制台"
linkTitle: "控制台"
weight: 222
date: 2021-08-23
description: >
  配置esxi的页面控制台
---

## 在主机界面上进行配置

### 设置控制台的IP地址

两块intel千兆网卡的mac地址分别是：

- 04:d4:c4:5a:e2:77
- 04:d4:c4:5a:e2:78

在路由器的dhcp中手工指定这两个网卡的IP地址，分别为 192.168.0.40 和 192.168.0.41 

在 esxi 主机的界面上（不是web页面）上，按 F2 Customize System/View Logs，输入root密码。

选择 "Configuration Management Network" -> "IPv4 configuration"，设置 "Set static IPv4 address and network configuration":

- IPv4 Address: 192.168.0.40
- Subnet mask: 255.255.255.0
- Default Gateway: 192.168.0.1

"Restart management network" 之后IP地址就改好了。

### 设置esxi主机的hostname

同样在 "Configuration Management Network" 选择 "DNS Configuration"，设置：

- Primary DNS Server: 192.168.0.1
- Hostname: skyesxi

再次 "Restart management network" 即可生效。

### 设置网络适配器

同样在 "Configuration Management Network" 选择 "Network Adapters"，选择将两块intel网卡（识别为 vmnic0 和 vmnic1）一起作为主机的默认管理网络连接。

再次 "Restart management network" 即可生效。

参考：[ESXI6.5只识别一块网卡解决方法](https://blog.51cto.com/bella41981/2071978)

## 使用控制台进行配置

用上面的IP地址 192.168.0.40，登录控制台页面。

### 开启ssh

登录 exsi web管理界面，进入 "管理" --> "服务"，在列表中找到 TSM-SSH，右键点启动。

![](./images/enable-ssh.png)

也可以在策略中选择"随主机启动和停止"。

### 设置swap

登录 exsi web管理界面，进入 "管理" --> "系统" --> "交换"，点击"编辑设置"。

数据存储下拉框中选择需要使用的 datastore：

![](images/config-swap.png)

设置好之后需要重启 esxi 主机。

