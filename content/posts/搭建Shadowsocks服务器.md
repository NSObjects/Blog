---
title: "搭建Shadowsocks服务器"
date: 2017-11-01T11:56:56+08:00
draft: false
---


最近VPN总是掉线连不上，所以决定自己搭建一个Shadowsocks服务器,找了各种教程后发现搭建Shadowsocks服务器非常简单
搭建Shadowsocks需要一个在墙外的服务器。
###VPS
免费：
亚马逊有免费1年的vps服务器，只需要绑定信用卡就可以了。 
付费：
https://www.vultr.com
https://bandwagonhost.com/cart.php
https://bandwagonhost.com
https://www.linode.com

如果对网速有要求只能选香港的vps,看youtube 4K无压力，平时只是用用google推荐使用vultr。创建服务器前最好先测试下网速。

linode
https://www.linode.com/speedtest
vultr 打开网页后搜索How can I test Vultr download speeds?
https://www.vultr.com/faq/

###安装GO
首先在官网找到最新二进制版本的连接
https://golang.org/dl/

####下载二进制包
`wget https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz`

####解压到Local目录下

`sudo tar -xzf go1.8.1.linux-amd64.tar.gz -C /usr/local`

####设置环境变量

`vim /etc/profile`

```
export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/work
export PATH=$GOPATH/bin:$PATH
```

`source /etc/profile`

###安装supervisor
`apt-get -y install supervisor`

`vim /etc/supervisor/conf.d/shadowsocks.conf`

```
[program:shadowsocks]
command=/root/work/bin/shadowsocks-server -c config.json
directory=/root
autostart=true
autorestart=true
```

###安装Go版本shadowsocks
`go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-server`

####创建配置文件
`vim config.json`
####参数

```
server          vps公网ip
server_port     服务器端口， 需要在安全组里打开这个端口，不然客户端无法访问
local_port      本地socks5代理端口
method          加密方式,下面是所有支持的加密方式:
                 aes-128-cfb, aes-192-cfb, aes-256-cfb, bf-cfb, cast5-cfb, des-cfb, rc4-md5, chacha20, salsa20, rc4,
password        用于加密传输的密码r
timeout         超时设置，以秒为单位
```

```
{
  "server":"xxx.xxx.xxx.xx",
  "server_port":2333,
  "local_port":1080,
  "password":"123456",
  "timeout":60,
  "method":"aes-256-cfb"
}
```

Mac os使用
首先下载客户端,NG是最新版的功能比较多。

```
https://sourceforge.net/projects/shadowsocksgui/?source=typ_redirect
```

```
https://github.com/shadowsocks/ShadowsocksX-NG/releases/download/v1.6.1/ShadowsocksX-NG.1.6.1.zip
```
iOS推荐使用Wingy，非常方便的小工具
![](http://7u2kla.com1.z0.glb.clouddn.com/15101075736153.jpg)

linux上使用
先下载客户端程序。然后编辑config文件

```
go get github.com/shadowsocks/shadowsocks-go/cmd/shadowsocks-local
```
```
{
    "server":"xx.xx.xx.xx", 服务器地址
    "server_port":8080, 服务器端口
    "local_port":1080, 本地监听的端口
    "password":"123456",
    "method": "aes-128-cfb-auth",
    "timeout":600
}
```
在后台启动这个服务
`nohup ./shadowsocks-local config.json &`

在linux上使用需要浏览器设置好代理方式。以firefox为例;
先安装AutoProxy-ng插件
![](http://7u2kla.com1.z0.glb.clouddn.com/15101081468737.jpg)




