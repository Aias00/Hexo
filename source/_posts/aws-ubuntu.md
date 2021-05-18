---
title: 在亚马逊免费试用ubuntu
tags: [liunx]
abbrlink: db8ef88b
date: 2020-12-12 10:08:58
---

本篇不写参与试用方法,只记录启动实例之后做的一系列操作,只要是 ubuntu-18.04 版本的系统都可以参考此操作流程

### 更新 apt

```shell
apt-get update
```

### 安装 shadowsocks-libev

因为服务器在东京,正好手头没有合适的梯子,不利用起来就是浪费

1.准备编译环境

```shell
apt install --no-install-recommends build-essential autoconf libtool \
    libssl-dev gawk debhelper dh-systemd init-system-helpers pkg-config asciidoc \
    xmlto apg libpcre3-dev zlib1g-dev libev-dev libudns-dev libsodium-dev \
    libmbedtls-dev libc-ares-dev automake
```

2.安装 git

```shell
apt install git -y
```

3.获取源码并编译

首先要进入到你想要保存源码的目录,比如/usr/local/xxx(自己建的文件夹)

```shell
git clone https://github.com/shadowsocks/shadowsocks-libev.git
cd shadowsocks-libev
git submodule update --init
./autogen.sh && ./configure --disable-documentation && make
sudo make install
```

4.创建配置文件

```shell
sudo mkdir /etc/shadowsocks-libev
sudo vim /etc/shadowsocks-libev/config.json
```

配置文件内容,记得修改密码

```json
{
  "server": "0.0.0.0",
  "server_port": 8388,
  "local_port": 1080,
  "password": "password",
  "timeout": 600,
  "method": "aes-256-cfb",
  "fast_open": false
}
```

5.创建 Shadowsocks-libev.service 配置文件,用于配置服务启动

```shell
sudo vim /etc/systemd/system/shadowsocks-libev.service
```

内容如下:

```shell
[Unit]
Description=Shadowsocks-libev Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/ss-server -c /etc/shadowsocks-libev/config.json -u
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

6.启动服务

```shell
sudo systemctl start shadowsocks-libev
```

7.配置开机启动

```shell
sudo systemctl enable shadowsocks-libev
```

至此,配置完成,自己找找客户端进行连接,上网冲浪吧(记得开放安全组端口噢)

### 未完待续
