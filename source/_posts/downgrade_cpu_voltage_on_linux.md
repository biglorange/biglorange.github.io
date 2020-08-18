---
title: Linux 下降低 CPU 电压
date: 2019-12-15 15:09:29
tags:
	- Arch Linux
	- CPU
categories:
	- Arch Linux
description: Arch Linux 下使用 undervolt 降低 CPU 电压以减小发热
---

#### 软件

python-undervolt

#### 使用方式

sudo undervolt --core -100 --cache -100

使用方式注意点

核心和 cache 的电压要降低相同的数值，同时 CPU 会自动设置降压为最接近设定值的数值

比如我的降压设置为 -85mV，系统自动匹配值为-84.96mV

```shell
temperature target: -3 (97C)
core: -84.96 mV
gpu: 0.0 mV
cache: -84.96 mV
uncore: 0.0 mV
analogio: 0.0 mV
```

降低电压会导致系统不稳定，过低会造成死机，重启后会自动恢复为默认值，当降低的电压可以稳定运行时，可以将其配置为启动时自动生效

要设置降压一直生效，需要配置对应的自启动任务

#### 设置降压在系统自启动时生效

创建 systemd 服务

`/etc/systemd/system/undervolt.service`

```shell
[Unit]
Description=undervolt

[Service]
Type=oneshot
ExecStart=/usr/bin/undervolt -v --core -85 --cache -85
```

然后创建一个定时器 `/etc/systemd/system/undervolt.timer` 来周期性触发任务

```shell
[Unit]
Description=Apply undervolt settings

[Timer]
Unit=undervolt.service
# Wait 2 minutes after boot before first applying
OnBootSec=2min
# Run every 30 seconds
OnUnitActiveSec=30

[Install]
WantedBy=multi-user.target
```

启用定时器和服务

```shell
sudo systemctl enable undervolt.timer
```

