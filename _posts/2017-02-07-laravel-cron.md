---
layout: post
title: Laravel 学习笔记 —— 任务调度
date: 2017-02-07 22:12:09
description: laravel 任务调度配置
img: laravel_cron.jpg
tags: [laravel, linux]
---

#### 1.在 `App\Console\Kernel.php` 中编写 laravel 代码
```php
protected function schedule(Schedule $schedule)
{
   $schedule->call(function () {
      \Log::info('任务调度test');
   })->everyMinute();
 }
```
	

#### 2.在服务的 `/var/spool/cron/root` 文件中添加代码
```shell
crontab -e
```

输入以下代码：
```shell
#唯一一个需要加入到服务器的 Cron 项目

* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

	

查看 crond 列表
```shell
crontab -u root -l


[root@iZuf6]# crontab -u root -l
* * * * * php /var/www/html/XYJ/artisan schedule:run >> /dev/null 2>&1
[root@iZuf6]#
	
```


不需要重启 cron 服务，因为系统每分钟都会读一遍 `/var/spool/cron` 目录下的文件。

如果发现按照如下配置还是不能执行的话，可以用以下方法排除问题：

看一下命令有没有使用绝对路径，比如这里使用 `/usr/local/php/bin/php` 而不是 `php`，使用 `/data/wwwroot/test/artisan` 而不是 `artisan`。

如果已经使用了绝对路径还是不执行，那就直接在命令行输入 `/usr/local/php/bin/php /data/wwwroot/test/artisan schedule:run 1>> /dev/null 2>&1`，看看有没有执行，如果没有执行，那就是 laravel 代码的问题，
如果执行了说明是环境变量的问题，好好检查路径的问题。如果不知道 php 在什么地方，在命令行输入 `which php`，就会提示你 php 安装在什么位置了。