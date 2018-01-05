---
layout: post
title: MySql 重置 root 账户密码
date: 2016-10-28 23:28:12
description: mysql 重置密码
img: mysql_reset_password.png
tags: [mysql]
---

#### 1.停止 mysqld 进程
```shell
systemctl stop mysqld.service
```

#### 2.安全模式下进入 mysql
```shell
mysqld_safe --user=mysql --skip-grant-tables --skip-networking &
mysql -u root mysql
```


#### 3.更新 root 账户密码
```shell
mysql> update mysql.user set password=PASSWORD("new password") where user='root';
mysql> FLUSH PRIVILEGES;
```

#### 4.重启 mysql
```shell
systemctl restart mysqld.service
```

