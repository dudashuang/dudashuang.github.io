---
layout: post
title: Laravel学习笔记——注意事项
date: 2016-10-11 23:12:02
description: 在使用laravel过程中常见的一些问题
img: laracon2014.png
tags: [laravel]
---

#### 1. utf8mb4字符集
Laravel 5.4默认使用utf8mb4字符编码，而不是之前的utf8编码。刚升级到5.4的小伙伴经常会遇到如下的错误

	[Illuminate\Database\QueryException]
	SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 1000 bytes (
	SQL: alter table `password_resets` add index `password_resets_email_index`(`email`))
	[PDOException]
	SQLSTATE[42000]: Syntax error or access violation: 1071 Specified key was too long; max key length is 1000 bytes

这主要是mysql支持的utf8最大长度字符为3字节，我们在执行migration的时候字符串默认长度为255，在utf8编码下为765个字节，utf8mb4最大长度字符为4字节，所以765个字节只能转换为191的长度。解决方法：ServiceProvider.php


	use Illuminate\Support\Facades\Schema; 
	/** 
	* Bootstrap any application services. 
	* 
	* @return void 
	*/ 
	public function boot() 
	{ 
	     Schema::defaultStringLength(191); 
	}


#### 2. mysql严格模式
laravel5.4默认打开MySQL的严格模式，严格模式的意思是指mysql会对数据进行严格的校验，例如一个字段定义长度为10时，当写入长度大于10的数据时，严格模式下会报错，非严格模式下不存储溢出部分，而不会报错。还有类型检查，字符型的字段写入数字等操作在严格模式下均会报错。但这其实不是坑，而且非常有助于我们养成严格规定数据类型的习惯。在这里提出来，是因为我们有时候接手二次开发的项目，进行重构的时候经常会出现数据格式错误，临时解决办法是：

	/**
	config/database.php
	*/
	'mysql' => [
	...
	    'strict' => false,
	]

PHP作为一门弱类型、动态类型检查的语言，在给我们带来方便的同时也带来了很多隐患，养成良好的编程习惯，才是解决问题的根本方法。

#### 3. 队列Job超时问题

有时候我们会有一些异步的请求，需要用到队列去处理，Laravel为我们提供了很方便的任务队列。但是，laravel队列中每个任务的默认执行时间为1分钟，超过一分钟的任务会重新进入队列的尾部。这会带来一些问题，当我们处理一些很耗时的任务时，这个任务会多次重新进入队列，导致这个任务执行多次。举个例子，当我们在一个任务中遍历用户列表，给每个用户发送一个模板消息，当1分钟内处理不完，会再次进入队列，导致每个用户收到多条消息。解决办法：

	/**
	 * Execute the job.
	 * @param AliyunOssService $aliyunOssService
	 * @return void
	 */
	public function handle(AliyunOssService $aliyunOssService)
	{
	    $this->job->delete();
	            ...
	}

在任务执行开始，删除这个任务，这样不管这个任务执行成功与否都不会再进入队列。

> 未完待续，本篇文章将长期更新...