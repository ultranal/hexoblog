---
layout: post
title: 调整 Linux 时区设置
date: 2019-02-12 17:11:48.698489060 +08:00
categories:
  - linux
tags: 
  - timezone
---

配置 Linux 的过程中，调整时区设置通常是比较重要的一项。参考了[这个](https://unix.stackexchange.com/questions/110522/timezone-setting-in-linux)和[这个](https://unix.stackexchange.com/questions/452559/what-is-etc-timezone-used-for)回答，粗略的记录一下时区的设置。

## 时区配置的位置

系统的时区设置主要来自两个文件`/etc/timezone`和`/etc/localtime`，同时也受到环境变量`TZ`的调整。
总的来说，只要没有通过`TZ`覆盖，glibc 就会通过`/etc/localtime`来确定本地时区。换言之，除了一些特殊应用（例如老版本的Java），*nix 普遍是通过`/etc/localtime`确定时区的。

### 格式

`/etc/timezone`的格式比较简单，是纯文本记录的时区信息，例如 Asia/Shanghai。`/etc/localtime`则是`/usr/share/zoneinfo/`目录下（内有全部的时区配置文件）某一文件的符号链接。

## 调整时区配置

推荐的方法是使用`timedatectl`命令：
```
# timedatectl set-timezone "Asia/Shanghai"
```

不带参数的`timedatectl`命令则可以查看当前时区设置：
```
$ timedatectl (timedatectl status也可)
                      Local time: Tue 2019-02-12 17:31:11 CST
                  Universal time: Tue 2019-02-12 09:31:11 UTC
                        RTC time: Tue 2019-02-12 09:31:11
                       Time zone: Asia/Shanghai (CST, +0800)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```
其中 RTC 代表本机的硬件时间。

也可以通过这一命令查找时区：
```
$ timedatectl list-timezones
```
当然也可用`tzselect`：
```
$ tzselect
```

修改`/etc/timezone`的方法就比较简单粗暴：
```
# echo "Asia/Shanghai" > /etc/timezone
```
临时修改时区的话，也可用：
```
$ export TZ='Asia/Shanghai'
```