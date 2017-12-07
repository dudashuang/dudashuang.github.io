---
layout: post
title: mysql重置root账户密码
date: 2016-10-28 23:28:12
description: mysql重置密码
img: mysql_reset_password.png
tags: [mysql]
---

#### 1.停止mysqld进程

	systemctl stop mysqld.service

#### 2.安全模式下进入mysql

	mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
	mysql -u root mysql

#### 3.更新root账户密码

	mysql> update mysql.user set password=PASSWORD("new password") where user='root';
	mysql> FLUSH PRIVILEGES;

#### 4.重启mysql


