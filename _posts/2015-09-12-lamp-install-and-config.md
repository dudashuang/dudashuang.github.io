---
layout: post
title: LAMP环境安装指引
date: 2015-09-12 21:12:20 +0300
description: LAMP环境安装
img: sword.gif # Add image post (optional)
tags: [LAMP, Linux]
---

#### 1. linux镜像
{% highlight bash %}
http://mirrors.aliyun.com/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1611.iso
{% endhighlight %}	
这里linux的镜像选择的是阿里云的源。


#### 2. linux基础设置
{% highlight bash %}
#permanent为设置永久生效，否则重启则失效，开启防火墙的80端口
firewall-cmd --zone=public --add-port=80/tcp --permanent        

#重启防火墙
firewall-cmd --reload

#关闭selinux
vi /etc/sysconfig/selinux
SELINUX=disabled
{% endhighlight %}	
	
#### 3. 安装epel、remi源
以/opt为下载目录
{% highlight bash %}
wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
rpm -Uvh epel-release-7-9.noarch.rpm

wget http://rpms.famillecollet.com/enterprise/7/remi/x86_64/remi-release-7.3-1.el7.remi.noarch.rpm
rpm -Uvh remi-release-7.3-1.el7.remi.noarch.rpm
{% endhighlight %}

#### 4. 安装apache、PHP
{% highlight bash %}
yum --enablerepo=remi install php70-php
{% endhighlight %}

**生成文件：**
{% highlight bash %}
/etc/httpd/conf.d/php70-php.conf
/etc/httpd/conf.modules.d/15-php70-php.conf
/opt/remi/php70/root/usr/lib64/httpd/
/opt/remi/php70/root/usr/lib64/httpd/modules/libphp7.so
/usr/lib64/httpd/modules/libphp70.so
/usr/share/httpd/icons/php70-php.gif
/var/opt/remi/php70/lib/php/opcache/
/var/opt/remi/php70/lib/php/session/
/var/opt/remi/php70/lib/php/wsdlcache/
{% endhighlight %}

**配置文件位置：**
{% highlight bash %}
/etc/opt/remi/php70/php.ini
/etc/httpd/conf/httpd.conf
{% endhighlight %}

#### 5. 安装PHP常用扩展
{% highlight bash %}
yum  install php70-php-xml php70-php-mbstring php70-php-mcrypt php70-php-gd php70-php-mysqlnd php70-php-opcache
{% endhighlight %}	
会在/etc/opt/remi/php70/php.d文件夹下生成对应的.ini文件

#### 6. 修改apache的配置文件
{% highlight bash %}
vim /etc/httpd/conf/httpd.conf
AllowOverride All      #将AllowOverride 字段修改为all，否则apache默认拒绝所有访问
systemctl restart httpd.service   #重启apache
{% endhighlight %}

#### 7. 安装mysql
{% highlight bash %}
#下载mysql-server
wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm

yum install mysql
{% endhighlight %}

#### 8. 设置快捷方式
{% highlight bash %}
ln -s /opt/remi/php70/root/usr/bin/php /usr/bin/php
ln -s /opt/remi/php70/root/usr/bin/php-cgi /usr/bin/php-cgi
ln -s /opt/remi/php70/root/usr/bin/php-phar /usr/bin/php-phar
{% endhighlight %}

#### 9. 安装composer
{% highlight bash %}
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
ln -s /opt/composer.phar /usr/bin/composer
{% endhighlight %}	

#### 10. 设置开机自启动
{% highlight bash %}
systemctl enable httpd.service
systemctl enable mysqld.service
{% endhighlight %}

最后，诸如redis，supervisor，git，vim等常用工具均可用yum安装。对于刚入门的小伙伴来说，搭建环境是一个很繁琐的过程。希望本文能对初学者有一点帮助：）