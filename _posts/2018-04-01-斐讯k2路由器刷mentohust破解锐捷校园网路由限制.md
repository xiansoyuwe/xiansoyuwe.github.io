---
layout:     post
title:      斐讯k2路由器刷mentohust破解锐捷校园网路由限制
date:       2019-04-01
author:     诸葛冰箱
header-img: img/post-bg-o.jpg
catalog: true
tags:
    - 大学
---
此文章综合大神的教程，部分内容纯属搬运，将附上详细链接。

所在学校：华科，其他学校大同小异，只要是用的锐捷，方法都差不多。

适合版本k2、k1s，v22系列固件版本如V22.3.15.128、V22.3.17.148、V22.4.5.39

## 一、先刷breed： ##

1、开启路由器，电脑连接路由器wifi，浏览器输入地址：192.168.2.1或p.to，输入初始账号密码，进入路由器设置页面

2、下载文件tianbaoha_breed_ssh.dat，地址 http://download.csdn.net/download/f1121814098/10130014

3、右上角“高级设置”——“系统设置”——“备份恢复”，在恢复配置文件一栏，点击浏览，选中配置文件“tianbaoha_breed_ssh.dat”，选择好后，点击“恢复备份”，然后耐心等待系统重启，经过这一步操作后，路由内已经刷入breed，并且原厂固件已被破解 4、重新连接wifi，刷新192.168.2.1，登入密码变为 tianbaoha


## 二、刷入华硕固件 ##

1、下载华硕固件，k2下载地址：http://download.csdn.net/download/f1121814098/10130035

2、在路由器设置页面（192.168.2.1）找到  “高级设置”——“系统设置”——“手动升级”，升级文件选择下载好的华硕固件，点击升级，等待升级完成并重启


3、重新连接wifi，账号密码如下
wifi名称：PDCN        管理页面192.168.123.1      账号密码均为admin          wifi密码：1234567890

4、能进入管理页面192.168.123.1，则说明固件刷成功了，否则重复第一部分


## 三、安装mentohust ##

1、下载页面：https://github.com/tkkcc/mentohust，下载mentohust文件

2、下载putty，winscp（windows下），具体文件百度吧

3、电脑连接路由器wifi，然后winscp，putty输入ip192.168.123.1，连接进入我们刚刷入的华硕固件linux系统，账号密码同上

4、使用winscp上传mentohust至/etc/storage/目录，把mentohust的文件属性改为777加入执行权限：chmod 777 /etc/storage/mentohust

5、查看网卡输入ifconfig，看网线对应的网卡是哪块，我这是eth2.2，试着启动mentohust：/etc/storage/mentohust -u用户名 -p密码 -neth2.2 -d2 

6、认证成功则ctrl+c终止掉，在上面的命令加入-w和-b1：mentohust：/etc/storage/mentohust -u用户名 -p密码 -neth2.2 -d2  -b1 -w，这样程序在后台运行且参数写进了文件，下次就不用配置了

7、自己输入mentohust -h查看帮助信息

8、登录华硕固件管理界面，在自定义设置----自定义脚本里面，添加mentohust自启动命令。 我放在了“路由启动后”这一栏里面，

## 四、开启锐捷拨号 ##

 /etc/storage/mentohust

 logger -t "进行锐捷拨号"

9、大功告成，还可以设置ss翻墙等内容，破解校园网限制简直美滋滋

## 参考链接： ##

- https://www.cnblogs.com/Arago/p/6480005.html
- http://www.right.com.cn/forum/thread-185514-1-1.html
- https://github.com/tkkcc/mentohust
- http://blog.sina.com.cn/s/blog_dc642faa0102x1on.html

[原文发自我的csdn](https://blog.csdn.net/f1121814098/article/details/78614470)