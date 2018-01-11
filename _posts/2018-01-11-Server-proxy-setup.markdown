---
layout: post
title:  "为服务器设置科学上网"
date:   2018-01-11 10:00:00
category: Tools
---
<br />
最近Nexus OSS代理的某些Repo速度简直降到了不可用的地步。然后就为服务器增加了科学上网，走的是早就部署的SS。
<br />

#### 安装
首先为服务器安装SS，为了简单我就与科学上网的服务器一样选了shadowsocks-libev。
采用从代码编译安装，步骤如下

```
# Installation of basic build dependencies
## Debian / Ubuntu
sudo apt-get install --no-install-recommends gettext build-essential autoconf libtool libpcre3-dev asciidoc xmlto libev-dev libc-ares-dev automake libmbedtls-dev libsodium-dev
## CentOS / Fedora / RHEL
sudo yum install gettext gcc autoconf libtool automake make asciidoc xmlto c-ares-devel libev-devel
## Arch
sudo pacman -S gettext gcc autoconf libtool automake make asciidoc xmlto c-ares libev

# Installation of Libsodium
export LIBSODIUM_VER=1.0.13
wget https://download.libsodium.org/libsodium/releases/libsodium-$LIBSODIUM_VER.tar.gz
tar xvf libsodium-$LIBSODIUM_VER.tar.gz
pushd libsodium-$LIBSODIUM_VER
./configure --prefix=/usr && make
sudo make install
popd
sudo ldconfig

# Start building
./autogen.sh && ./configure && make
sudo make install
```

这里要注意Libsodium的版本，要卸掉从apt下载的版本，不然make会报错

#### 配置
新建一个配置文件`$ vi ~/shadowsocks-libev/local.json `

```
{
    "server":"server_ip",
    "server_port":server_port,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"foobar",
    "timeout":600,
    "method":"chacha20-ietf-poly1305"
}
```
然后就可以启动试试了`$ ss-local -c ~/shadowsocks-libev/local.json`，不出意外SS就已经成功连上了

将`ss-local`的启动命令加进`rc.local`做成自启动

```
# vi /etc/rc.local

在文件末尾加入
/usr/local/bin/ss-local -c /path/to/shadowsocks-libev/local.json
```
齐活

#### 设置代理
支持socks5的应用可以直接设置127.0.0.1:1080为代理了，但偏偏Nexus OSS只支持HTTP Proxy(我不会告诉你我就这么直接用的，还找了半天错查了很久log)。所以还需要加个代理链，把socks5代理转为http代理。

安装polipo`# apt-get install polipo`

修改配置

```
# vi /etc/polipo/config

socksParentProxy = "127.0.0.1:1080"
socksProxyType = socks5
proxyPort = 8888
```
启动`# service polipo start`

<br />
让我来用wget测试一下

```
$ export http_proxy="http://127.0.0.1:8888"
$ wget www.google.com

--2018-01-11 10:06:56--  http://www.google.com/
正在连接 127.0.0.1:8888... 已连接。
已发出 Proxy 请求，正在等待回应... 302 Found
位置：http://www.google.com.ph/?gfe_rd=cr&dcr=0&ei=wMZWWorEHKHD8AfpwYDgBw [跟随至新的 URL]
--2018-01-11 10:06:56--  http://www.google.com.ph/?gfe_rd=cr&dcr=0&ei=wMZWWorEHKHD8AfpwYDgBw
再次使用存在的到 127.0.0.1:8888 的连接。
已发出 Proxy 请求，正在等待回应... 200 OK
长度： 未指定 [text/html]
正在保存至: “index.html”

index.html              [ <=>                  ]  11.40K  --.-KB/s   用时 0.09s

2018-01-11 10:06:56 (132 KB/s) - “index.html” 已保存 [11671]
```
成了

<br />
<br />
接下来给各个应用设置相应的代理就好了   
(我设置好后经测试速度并没有显著提升😳)