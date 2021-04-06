---
layout: post
title: 一天一个Linux命令--chage
tags:
  - Linux
categories:
  - IT技术
date: 2020-07-31 10:32:00
---

## 介绍

chage 命令是用来修改账号和密码的有效期限

<!--more-->

### 使用语法

```
chage [选项] 用户名
```

### 选项

```
  //设置最后改密码的日期
  -d, --lastday LAST_DAY        set date of last password change to LAST_DAY

  //设置账号有效期的最终日期
  -E, --expiredate EXPIRE_DATE  set account expiration date to EXPIRE_DATE

  //使账号失效
  -I, --inactive INACTIVE       set password inactive after expiration
                                to INACTIVE

  //查看账号有效期信息
  -l, --list                    show account aging information

  //设置修改密码的最小间隔天数，0表示可以随时修改密码
  -m, --mindays MIN_DAYS        set minimum number of days before password
                                change to MIN_DAYS

  //设置密码有效的最长天数， 99999表示永久有效
  -M, --maxdays MAX_DAYS        set maximim number of days before password
                                change to MAX_DAYS

  //设置密码过期前开始提示警告的天数
  -W, --warndays WARN_DAYS      set expiration warning days to WARN_DAYS

```

### 示例

```bash
[huanghe@huanghe_centos ~]$ chage -l huanghe
```

表示查询用户 huanghe 的有效期信息

```bash
Last password change					: Mar 14, 2019
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

```bash
[huanghe@huanghe_centos ~]$ chage -M 99999 huanghe //将用户huanghe有效期改为永久
```
