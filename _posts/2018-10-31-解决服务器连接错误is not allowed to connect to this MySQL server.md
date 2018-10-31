---
layout: post
title:  "解决服务器连接错误‘XXX’is not allowed to connect to this MySQL server"
date:    2018-10-31 11:26:50  
description: "服务器连接错误"
tag: Android
---

注:本文是在stackoverflow上查找的解决方案的汇总,只是记录解决步骤

最近在项目中,要求可以配置局域网服务器信息,当连接数据库时,出现类似以下错误:

	Host '192.168.0.1' is not allowed to connect to this MySQL server

像这种错误,主要是权限问题

问题症结是MySQL 没有开放远程登录的权限。这时要看你的服务器到底用的那种系统，linux或者是Windows，这个解决办法不同。解决的办法就是开启 MySQL 的远程登陆帐号。
有两大步：

### 1、确定服务器上的防火墙没有阻止 3306 端口。

MySQL 默认的端口是 3306 ，需要确定防火墙没有阻止 3306 端口，否则远程是无法通过 3306 端口连接到 MySQL 的。

如果您在安装 MySQL 时指定了其他端口，请在防火墙中开启您指定的 MySQL 使用的端口号。

如果不知道怎样设置您的服务器上的防火墙，请向您的服务器管理员咨询。

### 2、增加允许远程连接 MySQL 用户并授权。

##### 1）首先以 root 帐户登陆 MySQL

在 Windows 主机中点击开始菜单，运行，输入“cmd”，进入控制台，进入MySQL 的 bin 目录下，然后输入下面的命令。

	MySQL –u root –p 123456

123456 为 root 用户的密码。

在 Linux 主机中在命令提示行下输入下面的命令。（怎么登陆到linux vps的mysql呢？）

输入：

	/usr/local/mysql/bin/mysql -u root –p

然后再输入密码即可进入。

##### 2）创建远程登陆用户并授权

	mysql> grant all PRIVILEGES on shujukuming.* to 'yonghuming'@'192.168.1.1' identified by '123456' WITH GRANT OPTION;

上面的语句表示将数据库shujukuming的所有权限授权给yonghuming这个用户，允许yonghuming 用户在192.168.1.1 这个 IP 进行远程登陆，并设置yonghuming 用户的密码为 123456 。



逐一分析所有的参数：

- all PRIVILEGES 表示赋予所有的权限给指定用户，这里也可以替换为赋予某一具体的权限，例如：select,insert,update,delete,create,drop 等，具体权限间用“,”半角逗号分隔。

- shujukuming.* 表示上面的权限是针对于哪个表的，shujukuming指的是数据库，后面的 * 表示对于所有的表，由此可以推理出：对于全部数据库的全部表授权为“*.*”，对于某一数据库的全部表授权为“数据库名.*”，对于某一数据库的某一表授权为“数据库名.表名”。

- yonghuming 表示你要给哪个用户授权，这个用户可以是存在的用户，也可以是不存在的用户。

- 192.168.1.1 表示允许远程连接的 IP 地址，如果想不限制链接的 IP 则设置为“%”即可。

- 123456 为用户的密码。

> 这里的用户名和和密码指的是数据库连接的用户名和密码,不是APP登陆的用户名和密码

执行了上面的语句后，一般都会立即生效，返回值如下：

	Query OK, 0 rows affected (0.01 sec)

如果没有上面的语句那么请执行下面的命令，即可立即生效。

	Mysql> flush privileges