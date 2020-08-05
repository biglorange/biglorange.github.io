---
title: Fail2ban
date: 2018-11-10 22:18:34
tags:
 - fail2ban
 - linux
categories:
 - fail2ban
description:
 Fail2ban 安装及配置
---

### 安装

安装 fail2ban mailx sendmail

### 配置

配置 fail2ban jail 部分

```bash
sudo vim /etc/fail2ban/jail/local
DEFAULT]
# 以空格分隔的列表，可以是 IP 地址、CIDR 前缀或者 DNS 主机名
# 用于指定哪些地址可以忽略 fail2ban 防御
ignoreip = 127.0.0.1 172.31.0.0/24 10.10.0.0/24 192.168.0.0/24
 
# 客户端主机被禁止的时长（秒）
bantime = 864000
 
# 客户端主机被禁止前允许失败的次数 
maxretry = 3
 
# 查找失败次数的时长（秒）
findtime = 600
 
mta = mailx

destemail=orange804072060@gmail.com
#sendmail-whois[name=SSH, dest=orange804072060@gmail.com, sender=orange804072060@163.com]
 
[ssh-iptables]
enabled = true
filter = sshd
action = iptables[name=SSH, port=10881, protocol=tcp]
	 mail-whois[name=SSH, dest=orange804072060@gmail.com]
# sendmail-whois[name=SSH, dest=your@email.com, sender=fail2ban@email.com]
# Debian 系的发行版 
# logpath = /var/log/auth.log
# Red Hat 系的发行版
logpath = /var/log/secure
# ssh 服务的最大尝试次数 
maxretry = 3
```

配置邮件服务

```bash
sudo vim /etc/mail.rc
set from=username@domain.com
set smtp=smtp.163.com
set smtp-auth-user=username@domain.com
set smtp-auth-password=passwd
set smtp-use-starttls
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/home/orange/

```

开放 tcp 25端口 / 465  端口

```bash 
git clone https://github.com/fail2ban/fail2ban.git
## 下载源码，复制 mail-whois.conf 文件
```

```bash
复制 mail-whois.conf 到  /etc/fail2ban/action.d/mail-whois.conf 
```

修改 mail-whois.conf

```bash
sudo vim /etc/fail2ban/action.d/mail-whois.conf
# mail 全部替换为 mailx 所在的绝对路径
# Fail2Ban configuration file
#
# Author: Cyril Jaquier
#
#

[INCLUDES]

before = mail-whois-common.conf

[Definition]

# bypass ban/unban for restored tickets
norestored = 1

# Option:  actionstart
# Notes.:  command executed on demand at the first ban (or at the start of Fail2Ban if actionstart_on_demand is set to false).
# Values:  CMD
#
actionstart = printf %%b "Hi,\n
              The jail <name> has been started successfully.\n
              Regards,\n
              Fail2Ban"|/usr/bin/mailx -s "[Fail2Ban] <name>: started on <fq-hostname>" <dest>

# Option:  actionstop
# Notes.:  command executed at the stop of jail (or at the end of Fail2Ban)
# Values:  CMD
#
actionstop = printf %%b "Hi,\n
             The jail <name> has been stopped.\n
             Regards,\n
             Fail2Ban"|/usr/bin/mailx -s "[Fail2Ban] <name>: stopped on <fq-hostname>" <dest>

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck = 

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = printf %%b "Hi,\n
            The IP <ip> has just been banned by Fail2Ban after
            <failures> attempts against <name>.\n\n
            Here is more information about <ip> :\n
            `%(_whois_command)s`\n
            Regards,\n
            Fail2Ban"|/usr/bin/mailx -s "[Fail2Ban] <name>: banned <ip> from <fq-hostname>" <dest>

# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionunban = 

[Init]

# Default name of the chain
#
name = default

# Destination/Addressee of the mail
#
dest = root
```

应用 fail2ban  服务

```bahs
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

