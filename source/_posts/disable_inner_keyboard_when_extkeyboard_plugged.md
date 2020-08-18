---
title: 不知道第几个脚本
date: 2018-11-13 12:31:33
tags:
 - linux
 - script
 - shell
categories:
 - script
 - shell
description:
 在外接键盘时禁用笔记本内置键盘，防止误触
---

起因

在 B 站看到一期视频[禁用笔记本内置键盘](https://www.bilibili.com/video/av22403776?t=174)，感觉手动操作还是有一点麻烦，所以写了一个小的 shell 脚本，在 zsh 中设置 alias 别名使用，但还是觉得有一些麻烦，改进了第二版，对是否插着外接键盘进行判断，再进行禁用，第三版目标，实现自动化

第一版

```bash
#/usr/bin/bash
id=$(xinput | grep AT | cut -c53-54)
xinput set-prop $id "Device Enabled" 0
```

第二版

```bash
#/usr/bin/bash
#echo $1
if [ $1 == "disable" ]
then
	usbkeyboard=''
	usbkeyboard=$(xinput | grep "USB Keyboard" | sed -n '2p')
	if [ -n "$usbkeyboard" ]
	then
		echo "USB Keyboard Plugged"
		id=$(xinput | grep AT | cut -c53-54)
		xinput set-prop $id "Device Enabled" 0
		echo "Inner Keyboard disabled"
	else
		echo " USB Keyboard NOT Plugged,please check!"
	fi
elif [ $1 == "enable" ]
then
	id=$(xinput | grep AT | cut -c53-54)
	xinput set-prop $id "Device Enabled" 1
	echo "Inner Keyboard enabled"
fi

```

第三版

使用 udev 规则进行触发脚本









#### 参考文章

[Linux Udev](https://segmentfault.com/a/1190000011010908)

[udev 的使用](https://winddoing.github.io/post/3820.html)

[通过udev自动检测外置键盘并开启关闭内置键盘](http://www.xzcblog.com/post-266.html)

[外接鼠标禁用触摸板](https://harttle.land/2013/10/27/synaptics-settings-linux.html)