---
title: Archlinux 下 gpg import key 无法与服务器通信
date: 2020-08-17 22:01:59
tags:
	- Linux
categories:
	- Linux
description:
 当在国内网络环境中，需要 gpg --recv-keys key 时，由于网络通信的原因，导致服务器通信失败，进而造成获取公钥失败的一种迂回解决方案。
---

### 起因

日常更新代码时，python2-mako 作者的公钥更新了，更新软件时需要 import 作者对应的公钥，但是由于国内的网络环境，无法正常通信，导致无法获取公钥，软件包便无法更新

```bash
gpg: keyserver receive failed: Server indicated a failure
```

修改 --keyserver 选项也无法正常工作

```bash
gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 83AF7ACE251C13E6BB7DEFBD330239C1C4DAFEE1
gpg: keyserver receive failed: Server indicated a failure
```

### 解决方法

寻找一个国内可以正常访问的公钥服务器，例如 ubuntu 的

```bash
https://keyserver.ubuntu.com/
```

浏览器中打开对应网站，搜索对应公钥（前面需要添加 0x）

点击搜索结果中第一个 pub rsa2048/83af7ace251c13e6bb7defbd330239c1c4dafee1 这条记录

会获取到一串公钥，此时有两种方式

1. 拷贝完整内容为文本，然后使用 gpg import filename 导入公钥

2. 复制链接，使用 curl 下载该链接，使用管道 / 存储为文件导入公钥

   ```bash
   curl https://keyserver.ubuntu.com/pks/lookup\?op\=get\&search\=0x83af7ace251c13e6bb7defbd330239c1c4dafee1 | gpg --import
   ```





