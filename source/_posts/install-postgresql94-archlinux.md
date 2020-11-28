---

title: archlinux 安装并初始化配置 postgresql 9.4.26
date: 2020-11-28 20:01:40
tags: 
	- Arch Linux
	- postgresql
categories:
	- 数据库
description:
	在 archlinux 上安装并初始化配置 postgresql 9.4.26
---

## 安装并初始化数据库

### 安装 postgresql 9.2.46

```shell
yay -S postgresql-9.4
sudo pacman -S spostgresql-9.4
```

其中 postgresql 9.4 为当前仍然可以直接下载到的 psotgresql 最低版本

从 postgresql ftp 原始地址下载慢的话可以从 tsinghua 源下载压缩包后进行编译安装

```http
https://mirrors.tuna.tsinghua.edu.cn/postgresql/source/v9.4.26/
```

下载压缩包后放入 yay 对应的目录即可

```shell
/home/orange/.cache/yay/postgresql-9.4
```

安装完成后会自动创建 postgres 用户

```shell
cat /etc/passwd | grep postgres
postgres:x:88:88:PostgreSQL user:/var/lib/postgres:/bin/bash
```

### 初始化数据库

#### 初始化数据库前的准备：用户及路径权限

默认创建的 psotgres 有密码，但是我没有查到对应密码是多少，先使用 sudo 将原始密码删除

```shell
sudo passwd -d postgres                
passwd: password expiry information changed.
```

重新给 postgres 用户设置密码

```shell
sudo passwd postgers
```

然后重复两次密码就可以给 postgres 用户添加上密码了

进入 postgres 用户

```shell
su - postgres       
Password: 
mkdir: cannot create directory '/var/lib/postgres/.local': Permission denied
-bash: appendpath: command not found
-bash: appendpath: command not found
-bash: appendpath: command not found
```

mkdir 错误是因为 /var/lib/postgres/ 路径 postgres 用户没有权限读写，将该路径所有者修改为 postgres 即可

appendpath: command not found 错误则是 archlinux 更新带出的问题，使用 zsh 即可解决

修改路径所有者

在主用户下执行

```shell
sudo chown postgres:postgres /var/lib/postgres
```

将 psotgres 用户的 bash 更换为 zsh 并拷贝主用户下的部分配置文件

在 postgers 用户下执行

```shell
[postgres@BiGOranGe ~]$ chsh -s /usr/bin/zsh postgres
Changing shell for postgres.
Password: 
Shell changed.
```

拷贝主用户的配置文件并修改权限

主用户下执行

```shell
sudo cp -rf .zshrc .oh-my-zsh /var/lib/postgres/
sudo chown postgres:postgres /var/lib/postgres
```

#### 初始化数据库

进入 postgres 用户

```shell
postgres@BiGOranGe:~% initdb --locale en_US.UTF-8 -E UTF8 -D '/var/lib/postgres/data'
```

```shell
Success. You can now start the database server using:

    postgres -D /var/lib/postgres/data
or
    pg_ctl -D /var/lib/postgres/data -l logfile start
```

返回结果中包含上述内容则为初始化成功

##### 启动数据库并随开机启动

主用户下执行

```shell
sudo systemctl start postgresql.service
sudo systemctl enable postgresql.service 
Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.
```

##### 在数据库中创建用户

在 postgers 用户下执行

```shell
postgres@BiGOranGe:~% createuser -s -d orange
```

-s 具有超级用户权限

-d 具有创建数据库的权限

orange 则为主用户，这样可以在主用户下直接调用 psql 命令进入数据库

##### 创建数据库

主用户下执行

```shell
createdb orange
```

尝试连接数据库

主用户下执行

```shell
psql -d orange
```

-d orange 指定要登陆的数据库名，直接 psql 则会尝试登陆与当前用户名相同的数据库

##### 配置 PGDATA 等环境变量

postgres 用户下执行

```shell
vim /var/lib/postgres/.zshrc
```

```shell
export PGHOME="/var/lib/postgres"   
export PGDATA="$PGHOME/data"
```

添加这两个环境变量并使之生效

```shell
source /var/lib/postgres/.zshrc
```

##### 配置数据库可被外部连接

postgres 用户下执行

```shell
cd $PGDATA
```

```shell
vim postgresql.conf
```

添加这一行内容，使 postgresql 可以监听任意 ip 地址的连接

```shell
listen_addresses = '*'
```

```shell
vim pg_hba.conf
```

添加这一行内容，使任意 ip 都可以连接 postgersql 数据库

```shell
host    all             all             0.0.0.0/0               trust
```

修改配置后需要重启数据库才可以生效

主用户下执行

```shell
sudo systemctl restart postgresql.service
```



## pgAdmin3 登陆数据库

### 安装 pgAdmin3

```shell
yay -S pgadmin3
sudo pacman -S pgadmin3
```

同样下载缓慢的话使用 tsinghua 的源进行下载源码包

```http
https://mirrors.tuna.tsinghua.edu.cn/postgresql/pgadmin/pgadmin3/v1.22.2/src/
```

下载后的源码包放入 pgadmin3 编译目录

```shell
/home/orange/.cache/yay/pgadmin3
```

### 建立数据库连接

Host: 127.0.0.1

Port: 5432

sMaintenance DB: orange

Username: orange

建立连接后进行数据库操作







