---
title: fcitx
date: 2018-10-31 22:31:40
tags:
 - Arch Linux
 - fcitx
 - sogou
 - systemd
categories:
 - Arch Linux
 - sogou
description: 
 暴力使用脚本检测 fcitx-sogou cpu 使用率，高于阈值 kill 掉。
---

### 起因

Linux 下中文输入法搜狗算是比较好用的，但是存在不定时 CPU 占用飙升的 BUG ，Linux 下又没有比搜狗好用的中文拼音输入法（个人观点），关闭搜狗云联想可以解决这个问题，但是使用体验下降，通过脚本检测的方式监控搜狗输入法 CPU 占用率，高于阈值 Kill 。

### 实现

创建脚本文件

```bash
sudo vim /usr/local/bin/fcitx_high_cpu_kill.sh
#!/bin/bash

PROCESS="fcitx"

function get_pid_x { #process

PID=""

while [ ! $PID ]; do

PID=$(pidof $1)

sleep 1

done

echo $PID

}

function get_consume_cpu_process { #PID

echo $(top -b -p $1 -n 1 | tail -n 1 | awk '{print $9}')

}

pid=$(get_pid_x $PROCESS)

while [[ 1 ]]; do

#echo "the fcitx's pid is $pid"

#echo "the fcitx consumes $(get_consume_cpu_process $pid)%"

if [[ $(get_consume_cpu_process $pid) > 10.0 ]]; then

times=4

while [[ times > 0 ]]; do

sleep 1

times=$(($times-1))

if [[ $(get_consume_cpu_process $pid) < 10.0 ]]; then

break

elif [[ $(get_consume_cpu_process $pid) > 10.0 && times = 0 ]]; then

#echo "kill the process $pid."

kill $pid

pid=$(get_pid_x $PROCESS)

#else

#echo "not need to kill the process $pid"

fi

done

fi

sleep 4

done

```

赋予脚本可执行权限

```bash
sudo chmod +x /usr/local/bin/fcitx_high_cpu_kill.sh
```

创建 .service 文件通过 systemd 管理脚本自启动

```bash
sudo vim /etc/systemd/system/fcitx_cpu_limit.service
[Unit]
Description=Autostart Scripts

[Service]
ExecStart=/bin/sh /usr/local/bin/fcitx_high_cpu_kill.sh

[Install]
WantedBy=multi-user.target
After=display-manager.service
```

启动 .service 服务

```bash
sudo systemctl enable fcitx_cpu_limit.service
```

