---
layout: post
title: Laravel 学习笔记 —— 提升性能篇
date: 2016-12-05 22:31:21
description: laravel 性能提升
img: laravel_performance.png
tags: [laravel, php]
---

> 请升级到PHP7.0以上版本，较之上个版本，性能有着质的提升


#### 1. 使用 OPcache 扩展
OPcache 可以把编译后的 bytecode 存储在内存里面, 避免重复编译 PHP 所造成的资源浪费。
执行：
```shell
yum install php70-php-opcache
```


安装好 php-opcache 扩展后，将下列代码放到 php.ini 的末尾，重启 apache。
```shell
[Zend Opcache]
;zend_extension = /usr/local/php/lib/php/extensions/no-debug-non-zts-20090626/opcache.so
opcache.enable=1 ;启用操作码缓存
opcache.enable_cli=1 ;仅针对CLI环境启用操作码缓存
opcache.memory_consumption=128 ;共享内存大小，单位MB
opcache.interned_strings_buffer=8 ;存储临时字符串的内存大小，单位MB
opcache.max_accelerated_files=4000 ;哈希表中可存储的脚本文件数量上限
;opcache.max_wasted_percentage=5 ;浪费内存的上限，以百分比计
;opcache.use_cwd=1;附加改脚本的工作目录,避免同名脚本冲突
opcache.validate_timestamps=1 ;每隔revalidate_freq 设定的秒数 检查脚本是否更新
opcache.revalidate_freq=60 ;
;opcache.revalidate_path=0 ;如果禁用此选项，在同一个 include_path 已存在的缓存文件会被重用
;opcache.save_comments=1 ;禁用后将也不会加载注释内容
opcache.fast_shutdown=1 ;一次释放全部请求变量的内存
opcache.enable_file_override=0 ; 如果启用，则在调用函数file_exists()， is_file() 以及 is_readable() 的时候， 都会检查操作码缓存
;opcache.optimization_level=0xffffffff ;控制优化级别的二进制位掩码。
;opcache.inherited_hack=1 ;PHP 5.3之前做的优化
;opcache.dups_fix=0 ;仅作为针对 “不可重定义类”错误的一种解决方案。
;opcache.blacklist_filename="" ;黑名单文件为文本文件，包含了不进行预编译优化的文件名
;opcache.max_file_size=0 ;以字节为单位的缓存的文件大小上限
;opcache.consistency_checks=0 ;如果是非 0 值，OPcache 将会每隔 N 次请求检查缓存校验和
opcache.force_restart_timeout=180 ; 如果缓存处于非激活状态，等待多少秒之后计划重启。
;opcache.error_log="" ;OPcache模块的错误日志文件
;opcache.log_verbosity_level=1 ;OPcache模块的日志级别。致命（0）错误（1) 警告（2）信息（3）调试（4）
;opcache.preferred_memory_model="" ;OPcache 首选的内存模块。可选值包括： mmap，shm, posix 以及 win32。
;opcache.protect_memory=0 ;保护共享内存，以避免执行脚本时发生非预期的写入。 仅用于内部调试。
;opcache.mmap_base=null ;在Windows 平台上共享内存段的基地址

```
	
#### 2. 配置信息缓存
```shell
php artisan config:cache
```


上面命令会生成文件 bootstrap/cache/config.php，可以使用以下命令来取消配置信息缓存：
```shell
php artisan config:clear
```

此命令做的事情就是把 bootstrap/cache/config.php 文件删除。

#### 3. 使用路由缓存
```shell
php artisan route:cache
```

以上命令会生成 bootstrap/cache/routes.php 文件，需要注意的是，路由缓存不支持闭包函数路由，例如：
```php
Route::get('user/{id}', function ($id) {
	// Only called if {id} is numeric.
});
```


可以使用下面命令清除路由缓存：
```shell
php artisan route:clear
```


此命令做的事情就是把 `bootstrap/cache/routes.php` 文件删除。

#### 4. 类映射加载优化
```shell
php artisan optimize --force 
```
	
会生成 `bootstrap/cache/compiled.php` 和 `bootstrap/cache/services.json` 两个文件。

你可以可以通过修改 `config/compile.php` 文件来添加要合并的类。

在 production 环境中，参数 --force 不需要指定，文件就会自动生成。

要清除类映射加载优化，请运行以下命令：
```shell
php artisan clear-compiled
```

此命令会删除上面 optimize 生成的两个文件。

> 注意：此命令要运行在 `php artisan config:cache` 后，因为 optimize 命令是根据配置信息（如：`config/app.php` 文件的 providers 数组）来生成文件的。

#### 5. 自动加载优化
```shell
composer dump-autoload
```

此命令不止针对于 Laravel 程序，适用于所有使用 composer 来构建的程序。此命令会把 PSR-0 和 PSR-4 转换为一个类映射表，来提高类的加载速度

#### 6. 使用 Memcached 来存储会话

每一个 Laravel 的请求，都会产生会话，修改会话的存储方式能有效提高程序效率，会话的配置信息是 `config/session.php`，建议修改为 Memcached 或者 Redis 等专业的缓存软件：
```php
'driver' => 'redis',
```


#### 7. 使用专业缓存驱动器

「缓存」是提高应用程序运行效率的法宝之一，默认缓存驱动是 file 文件缓存，建议切换到专业的缓存系统，如 Redis 或者 Memcached，不建议使用数据库缓存。
```php
'default' => 'redis',
```


#### 8. 为数据集编写缓存逻辑
```php
/**
 * 用于select的省份列表
 * @return array
 */
public static function getProvinceList()
{
	$result = \Cache::tags(self::CACHE_TAG)->rememberForever('province-list:', function () {
		$infos = self::where(['upid' => 0, 'type' => 1])->select(['id', 'name'])->orderBy('displayorder', 'asc')->get()->toArray();
		return $infos;
	});
	return $result;
}
```
	


#### 9. 数据请求优化

规范路由和请求 API，合理运用预加载和延迟预加载