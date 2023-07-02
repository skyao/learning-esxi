---
title: "Realtek 8111网卡"
linkTitle: "Realtek 8111网卡"
weight: 300
date: 2021-08-23
description: >
 Realtek 8111网卡在esxi中的使用
---

### 安装驱动

下载下来 net55-r8168-8.045a-napi-offline_bundle.zip 文件，解压缩，将 vib 文件通过 esxi 的数据存储浏览器上传到 datastore1 中。

然后在 manage -》 package 页面中点 Install update ，给出路径如：

`/datastore1/upload/net55-r8168-8.045a-napi.x86_64.vib`

奈何安装失败！
