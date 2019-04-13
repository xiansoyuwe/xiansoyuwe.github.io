---
layout:     post
title:      DigitalOcean CentOS 7 配置 shadowsocks 服务端及客户端及优化
date:       2019-04-13
author:     诸葛冰箱
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - 科学上网
---
 
> 你为什么需要梯子？
> 
> 如果你只是想使用谷歌搜索，那我建议你别看了，太麻烦还费钱，不如使用[谷歌访问助手](https://github.com/haotian-wang/google-access-helper "谷歌访问助手")这样友好的工具或者直接更改hosts文件
> 
> 
但如果你想上YouTube这样需要好的网速场景或者是一名境外电商从业者，或者单纯地想保护个人隐私，我建议你可以买个vps搭个梯子。

>但价格可能略有点贵，大约30多人民币一个月，当然你也可以和身边的人分担。

>以下是我的教程，我自己的ss就是它搭建的。


# 服务器端 #

服务器租用商：[DigitalOcean](https://m.do.co/c/5044a7ef6da2)  

使用[我的链接注册](https://m.do.co/c/5044a7ef6da2)可以免费获得10美元，用于购买vps

安装环境：CentOS 7 X64

## 安装Shadowsocks ##
    yum install python-setuptools && easy_install pip
    pip install shadowsocks
## 配置Shadowsocks ##
    nano /etc/shadowsocks.json
然后输入
    
    {
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 1
    }


> 将上面的mypassword替换成你的密码，server_port也是可以修改的，例如3024，传说端口越小，效果越好，这个我没有去验证，但建议不要小于1024，以免引起不必要的麻烦，method是加密方式，如果想在路由器上运行的话可以改成rc4-md5这样路由器的负荷会小一些，同时加密的安全性也不错。

ctrl+o 保存

ctrl+x 退出

上方代码的解释：

- **server**	 服务端监听的地址，服务端可填写0.0.0.0
- **server_port**	 服务端的端口
- **local_address**	本地端监听的地址
- **local_port**	本地端的端口
- **password**	用于加密的密码
- **timeout**	超时时间，单位秒
- **method**	默认 "aes-256-cfb"，参见加密方法
- **fast_open**	是否使用 TCP_FASTOPEN, true / false
- **workers**	worker 数量，Unix/Linux 可用，如果不理解含义请不要改

> 加密方法参见：
[https://github.com/clowwindy/shadowsocks/wiki/Encryption](https://github.com/clowwindy/shadowsocks/wiki/Encryption "https://github.com/clowwindy/shadowsocks/wiki/Encryption")



> TCP_FASTOPEN参见：
[https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open](https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open "https://github.com/shadowsocks/shadowsocks/wiki/TCP-Fast-Open")

## 测试&启动 ##
    ssserver -c /etc/shadowsocks.json

## 使用supervisor自动后台运行Shadowsocks ##
    easy_install supervisor
然后创建配置文件,supervisord程序在运行后会自动查找并加载此目录配置文件。
    echo_supervisord_conf > /etc/supervisord.conf

编辑配置文件supervisord.conf，

    nano /etc/supervisord.conf

在后面添加

    [program:shadowsocks]
    command=ssserver -c /etc/shadowsocks.json
    autostart=true
    autorestart=true
    user=nobody

完成后启动supervisord

    supervisord -c /etc/supervisord.conf

## 设置supervisord开机启动 ##

编辑文件：

    nano /etc/rc.local

在末尾另起一行添加

    supervisord

保存退出（和上文类似），另centos7还需要为rc.local添加执行权限

    chmod +x /etc/rc.local

至此运用supervisord控制shadowsocks开机自启和后台运行设置完成

## shadowsocks服务器TCP优化 ##

    nano /etc/sysctl.conf



>     fs.file-max = 51200
    #提高整个系统的文件限制
    net.ipv4.tcp_syncookies = 1
    #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
    net.ipv4.tcp_tw_reuse = 1
    #表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
    net.ipv4.tcp_tw_recycle = 0
    #表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭；
    #为了对NAT设备更友好，建议设置为0。
    net.ipv4.tcp_fin_timeout = 30
    #修改系統默认的 TIMEOUT 时间。
    net.ipv4.tcp_keepalive_time = 1200
    #表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
    net.ipv4.ip_local_port_range = 10000 65000 #表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为10000到65000。（注意：这里不要将最低值设的太低，否则可能会占用掉正常的端口！）
    net.ipv4.tcp_max_syn_backlog = 8192
    #表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
    net.ipv4.tcp_max_tw_buckets = 5000
    #表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。
    #额外的，对于内核版本新于**3.7.1**的，我们可以开启tcp_fastopen：
    net.ipv4.tcp_fastopen = 3
    # increase TCP max buffer size settable using setsockopt()
    net.core.rmem_max = 67108864 
    net.core.wmem_max = 67108864 
    # increase Linux autotuning TCP buffer limit
    net.ipv4.tcp_rmem = 4096 87380 67108864
    net.ipv4.tcp_wmem = 4096 65536 67108864
    # increase the length of the processor input queue
    net.core.netdev_max_backlog = 250000
    # recommended for hosts with jumbo frames enabled
    net.ipv4.tcp_mtu_probing=1

保存并退出该文件,然后使用以下指令使配置生效：

    sysctl -p

# 客户端 #

服务端启动完成后需要配置本地端，
在此网页根据自己的系统下载最新的Shadowsocks GUI

所有版本

> [https://github.com/search?q=org%3Ashadowsocks+shadowsocks](https://github.com/search?q=org%3Ashadowsocks+shadowsocks "https://github.com/search?q=org%3Ashadowsocks+shadowsocks")

windows

> [https://github.com/shadowsocks/shadowsocks-windows](https://github.com/shadowsocks/shadowsocks-windows "https://github.com/shadowsocks/shadowsocks-windows")

客户端配置对应服务端配置

Save后可创建快捷方式放到开始菜单启动项里即可开机自动运行

## 配置多用户 ##

如果想多用户使用的话就需要更改配置。

首先通过ssh连上vps

在终端输入

    vi /etc/shadowsocks.json

创建配置文件

按 i 插入

插入以下内容（用户数任意，注意最后一个用户密码后面没有逗号）

    {
    	"server":"my_server_ip",  #填入你的IP地址
    	"local_address": "127.0.0.1",
    	"local_port":1080,
    	"port_password": {
    	"8381": "foobar1",#端口号，密码
    	"8382": "foobar2",
    	"8383": "foobar3",
    	"8384": "foobar4"
    	},
    	"timeout":300,
    	"method":"aes-256-cfb",
    	"fast_open": false
    }

上面开了四个端口（用户）为8381-8384，密码分别为foobar1-foobar4，你还需要填入你的IP地址。

**下面是详细配置说明**：

- server	服务器地址，填ip或域名
- local_address	本地地址
- local_port	本地端口，一般1080，可任意
- server_port	服务器对外开的端口
- password	密码，可以每个服务器端口设置不同密码
- port_password	server_port + password ，服务器端口加密码的组合
- timeout	超时重连
- method	默认: “aes-256-cfb”，见 Encryption
- fast_open	开启或关闭 TCP_FASTOPEN, 填true / false，需要服务端支持

**下面是详细操作说明**：

1、然后按Esc退出编辑，按shift+:，输入wq，回车，就保存退出了。
2、建议使用后端启动

- 前端启动：`ssserver -c /etc/shadowsocks.json`；
- 后端启动：`ssserver -c /etc/shadowsocks.json -d start`；
- 停止：`ssserver -c /etc/shadowsocks.json -d stop`；
- 重启(修改配置要重启才生效)：`ssserver -c /etc/shadowsocks.json -d restart`

3、设置开机启动

- 在终端输入`vi /etc/rc.local`，
- 把里面最后的带有ssserver的一大段默认的代码删除掉，
- 再把`ssserver -c /etc/shadowsocks.json -d start`加进去，
- 按wq保存退出。


4、到此就配置好了，试试多用户运行吧！