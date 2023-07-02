---
title: "在esxi上运行iperf3"
linkTitle: "iperf3"
weight: 272
date: 2021-11-23
description: >
  使用iperf3验证网卡的速度
---

## 准备软件

### 在esxi6.7上准备iperf

ssh 登录 esxi 6.7, 然后执行以下命令：

```bash
mkdir /temp
cd /temp
wget http://vibsdepot.v-front.de/depot/bundles/iperf-2.0.5-1-offline_bundle.zip 
esxcli software vib install -d /temp/iperf-2.0.5-1-offline_bundle.zip  --no-sig-check

```

结果输出为：

```bash
Installation Result
   Message: Operation finished successfully.
   Reboot Required: false
   VIBs Installed: DAST_bootbank_iperf_2.0.5-1
   VIBs Removed: 
   VIBs Skipped: 
```

清理并进入目录：

```bash
rm -rf /temp
cd /opt/iperf/bin
cp iperf iperf.copy
```

### 准备网络

查看目前 esxi 的网络情况：

```bash
esxcli network ip interface ipv4 get
```

如果只有一个网卡（兼管理网络），输出类似：

```bash
Name  IPv4 Address  IPv4 Netmask   IPv4 Broadcast  Address Type  Gateway      DHCP DNS
----  ------------  -------------  --------------  ------------  -----------  --------
vmk0  192.168.0.58  255.255.255.0  192.168.0.255   STATIC        192.168.0.1     false
```

另外测试之前关闭esxi的防火墙：

```bash
esxcli network firewall set --enabled false
```

为了测试 hp544+ 网卡（连接为56G）在 esxi 下的性能，需要在 esxi 中取消直通，然后设置相关的网络

- virtual switches 中添加 virtual switch， 如 "vSwitch-56G", uplink 选择 hp544+ 网卡
- port group 中添加新的 port group,如 "56G Network"，virtual switch 选择 "vSwitch-56G"
- vmkernel NICs 中添加新的 VMkernel NIC，port group, 选择 "56G Network"

再次查看网络：

```bash
esxcli network ip interface ipv4 get
```

可以看到新加的 56g 网络的 IP 地址：

```bash
Name  IPv4 Address    IPv4 Netmask   IPv4 Broadcast   Address Type  Gateway      DHCP DNS
----  --------------  -------------  ---------------  ------------  -----------  --------
vmk0  192.168.0.28    255.255.255.0  192.168.0.255    STATIC        192.168.0.1     false
vmk1  192.168.100.28  255.255.255.0  192.168.100.255  STATIC        0.0.0.0         false
```



## 测试

作为服务器端启动：

```bash
/opt/iperf/bin/iperf.copy -s -B 192.168.100.28
```

### 作为客户端

作为客户端执行：

```bash
/opt/iperf/bin/iperf.copy -c 192.168.100.1 -t 10 -P 10
```

`-t` 表示测试的总时间长度，`-P` 启动多线程进行并发测试，一般 8-10 比较合适。

或者增加参数， `-i` 表示每隔多长时间打印一次测试结果，避免长时间等待（默认需要等测试时间 -t 结束才显示结果）， `-f g ` 表示用 Gbits 来显示带宽：

```bash
/opt/iperf/bin/iperf.copy -c 192.168.100.1 -t 10 -i 2 -f g -P 10
```

### 测试结果

#### 场景1

- 服务器端是直通 + ubuntu server：之前测试 56g eth 模式下测速打开 48G

- 客户端为 esxi 6.7, 自带驱动，不直通，不开启 

- 测试带宽为： 32G左右

- cpu使用率有点高，大概20%-30%

结果：和正常的48G相比，只有 2/3 的性能，估计是 rdma 没有开启的原因。

```bash
esxcli software vib list | grep mlx
net-mlx4-core                  1.9.7.0-1vmw.670.0.0.8169922          VMW      VMwareCertified     2023-03-19
net-mlx4-en                    1.9.7.0-1vmw.670.0.0.8169922          VMW      VMwareCertified     2023-03-19
nmlx4-core                     3.17.13.1-1vmw.670.2.48.13006603      VMW      VMwareCertified     2023-03-19
nmlx4-en                       3.17.13.1-1vmw.670.2.48.13006603      VMW      VMwareCertified     2023-03-19
nmlx4-rdma                     3.17.13.1-1vmw.670.2.48.13006603      VMW      VMwareCertified     2023-03-19
nmlx5-core                     4.17.13.1-1vmw.670.3.73.14320388      VMW      VMwareCertified     2023-03-19
nmlx5-rdma                     4.17.13.1-1vmw.670.2.48.13006603      VMW      VMwareCertified     2023-03-19
```



## 参考

- https://www.controlup.com/resources/blog/entry/iperf-on-esxi/

