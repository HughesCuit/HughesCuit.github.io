---
layout: post
title: 使用Laravel + Infyom Laravel Generator + MySQL开发RESTful规范的移动应用API
tags:
  - php
  - 服务器
  - laravel
categories:
  - 开发日常
date: 2018-02-11 15:46:00
---

## 命名

### 驼峰规则

由于涉及到多种格式规范的命名，我们约定在`PHP`代码中:`类`名及`静态方法`的命名使用**_大驼峰规则_**，即`AaaBbbCcc`格式，而`变量`和`实例方法`的命名使用使用**_小驼峰规则_**，即`aaaBbbCcc`格式。

### 下划线命名

除 PHP 程序代码使用`驼峰规则`，我们在下述几种情况下使用`下划线命名`，即单词均为**_纯小写_**，单词之间使用**_下划线_**分隔(`aaa_bbb_ccc`)。

使用下划线命名的情况包括：

1. 数据库表名
2. 数据库字段
3. RESTful API 返回值`key`名
4. RESTful API 请求参数名

## 命令

### 安装 composer

[官网](https://getcomposer.org/download/)
composer 用来做包管理器

### 安装 Laravel

下载安装程序

```
composer global require "laravel/installer"
```

确保 $HOME/.composer/vendor/bin 目录（或你的操作系统的等效目录）已经放在你的环境变量 $PATH 中，以便系统可以找到 laravel 的可执行文件。

### 创建项目

#### 通过 Laravel

```
laravel new 你的项目
```

将自动帮你创建目录，以及安装`Laravel`的依赖。

#### 通过 composer

```
composer create-project --prefer-dist laravel/laravel 你的项目
```

### Infyom Laravel Generator

默认到此步骤，已完成数据库连接的配置。

此处可以通过`Infyom Laravel Generator`来生成`Model`、`Controller`、`API`等模板代码

#### 生成 Model

```
php artisan infyom:model 模型名称 --fromTable --tableName=数据库表名
```

#### 生成 API

```
php artisan infyom:api 模型名称 --fromTable --tableName=数据库表名
```

#### 生成脚手架

```
php artisan infyom:scaffold 模型名称 --fromTable --tableName=数据库表名
```
