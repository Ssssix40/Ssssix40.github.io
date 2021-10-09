---
layout: article
title: 自建 wireguard 加速游戏并实现NAT1(fullcone)
key: 202011131
tags:
  - Tutorials
  - wireguard
  - games
lang: zh-Hans
---

## 1. 前言

参考本文可以实现以下几点需求：

1. 通过wireguard建立隧道以及给linux内核添加FULLCONE模块，实现一条NAT1的**虚假**IPLC。
2. 因为wireguard本身会加速所有的流量，所以通过`netch`配合**虚假**`IPLC`局域网中转机的`socks5`代理服务达到在pc只加速游戏的效果。
3. @#￥%……&*（

本文实现的方式主要是通过打隧道的方式实现一个**虚假**的`iplc`，也就是说，如果您完全按照我的教程走的话，您需要具备以下的前提：

1. 一台延迟尚可且具有公网ip的落地vps，毕竟是用于打游戏，延迟相对比较重要，同时FULLCONE需要公网IP支持。
2. 一台中转的服务器，建议是局域网内的一台虚拟机或者一个树莓派。我这边是自己组了一台`pve`服务器，然后开了个lxc。需要注意的是，当服务开启之后，该中转机的所有流量都会走`wireguard`。
3. 当然还有一定的linux知识，因为本教程在一些细节方面可能讲的并不是那么全面。

本文会根据不同的需求拆开成[本体](#2-本体)以及[DLC](#3-dlc)两部分，完成**本体**就可以实现本文所述功能，但是**DLC**则提供了一些特殊的功能。比如`udp2raw`可以帮助`wireguard`躲避运营商给你时不时的一个断流打击。

## 2. 本体

### 2.1 安装wireguard

本文主要介绍`ubuntu`的安装方式，其他linux发行版敬请请参考[官方教程](https://www.wireguard.com/install/)或者google。

您需要在落地和中转机器上都安装wireguard，落地机器作为**server**而中转机器作为**client**。

```shell
su - root
apt update
# ubuntu 18.04 及以上版本
apt install wireguard resolvconf -y
# ubuntu 16.04 及以下版本
add-apt-repository ppa:wireguard/wireguard
apt update
apt install wireguard resolvconf -y
# 在安装完成之后建议重启一下
# reboot
```



### 2.2 配置wireguard

在**落地机器**依次输入下列命令:

```shell
# 开启ipv4流量转发
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# 创建并进入WireGuard文件夹
mkdir -p /etc/wireguard && chmod 0777 /etc/wireguard
cd /etc/wireguard
umask 077

# 生成服务器和客户端密钥对
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey
```

通过`ifconfig`检查主网卡名称，一般会为`eth0`或者`ens3`。

生成server配置文件`/etc/wireguard/wg0.conf`：

```shell
# 重要！如果名字不是eth0, 以下PostUp和PostDown处里面的eth0替换成自己服务器显示的名字
# ListenPort为端口号，可以自己设置想使用的数字
# 以下内容一次性粘贴执行，不要分行执行
echo "
[Interface]
  PrivateKey = $(cat server_privatekey)
  Address = 10.0.0.1/24
  PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
  PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
  ListenPort = 52540
  DNS = 8.8.8.8
  MTU = 1200

[Peer]
  PublicKey = $(cat client_publickey)
  AllowedIPs = 10.0.0.2/32 " > wg0.conf

```



生成client配置文件`/etc/wireguard/client.conf`:

```shell
# Endpoint是自己服务器ip和服务端配置文件中设置的端口号，自己在本地编辑好再粘贴到SSH里
# 以下内容一次性粘贴执行，不要分行执行
echo "
[Interface]
  PrivateKey = $(cat client_privatekey)
  Address = 10.0.0.2/24
  DNS = 8.8.8.8
  MTU = 1200

[Peer]
  PublicKey = $(cat server_publickey)
  Endpoint = 1.2.3.4:52540
  AllowedIPs = 0.0.0.0/0, ::0/0
  PersistentKeepalive = 25 " > client.conf
```

此时需要将`clinet.conf`的内容复制到**中转机器**的`/etc/wireguard/wg0.conf`里面。

然后在两台机器上分别启动wireguard，wireguard的部分命令如下：

**请特别注意，中转机器开启wg之后，无法通过ssh访问，需要从落地机器访问回来**

```shell
# 启动WireGuard
wg-quick up wg0

# 停止WireGuard
wg-quick down wg0

# 查看WireGuard运行状态
wg
```

到此时，一个**虚假**的`IPLC`了。

可以在中转机器执行如下命令进行测试：

```shell
curl ifconfig.me
> 1.2.3.4 # 注意此处显示的应该是你的落地机器ip
ping 10.0.0.1
# 正常应该是可以ping通的
```

### 2.3 开启FULLCONE NAT

虽然此时`IPLC`已经打通，但是此时的NAT类型并不是NAT1，而是NAT4：Symmetric NAT。

（可以通过[pystun](#31-pystun)验证，非必须步骤)

这个原因是因为

> TLDR: Linux内核树上未实现真正意义上的Full Cone NAT，Linux的 SNAT/MASQUERADE（以iptables的配置为例）均是Symmetric NAT。

所以，此时我们要从linux的内核下手，实现FULLCONE。

安装完整的内核：

```shell
apt install linux-image-$(uname -r)
```




安装一些依赖：（一般需要的都装好了，不通的发行版可能需求不一样 ，缺什么装什么就好了）

```shell
apt install gcc autoconf autogen libtool pkg-config libgmp3-dev -y
```

下载一些软件的源码:

```shell
cd ~
mkdir fullcone
cd fullcone
git clone git://git.netfilter.org/libmnl
git clone git://git.netfilter.org/libnftnl.git
git clone  git://git.netfilter.org/iptables.git
git clone https://github.com/Chion82/netfilter-full-cone-nat.git
```



#### 2.3.1 编译libmnl

```shell
cd ~/fullcone/libmbl
sh autogen.sh
./configure
make
make install
```

然后:

```shell
whereis libmnl

> libmnl: /usr/local/lib/libmnl.so /usr/local/lib/libmnl.la /usr/include/libmnl

ldd /usr/local/lib/libmnl.so
```

#### 2.3.2 编译libnftnl

```shell
cd ~/fullcone/libnftnl
sh autogen.sh
./configure
make
make install
```

#### 2.3.3 编译和临时启用netfilter-full-cone-nat

```shell
cd ~/fullcone/netfilter-full-cone-nat
make
insmod xt_FULLCONENAT.ko
#如果编译模块报错，请安装kernel-devel,载入模块报错Unknown symbol in module ，先modprobe nf_nat 再载入模块。
```

#### 2.3.4 编译和替换iptables

```shell
cp ~/fullcone/netfilter-full-cone-nat/libipt_FULLCONENAT.c ~/fullcone/iptables/extensions/
ln -sfv /usr/sbin/xtables-multi /usr/bin/iptables-xml
./autogen.sh
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH
./configure
make
make install

#先关闭iptables
systemctl  stop iptables
#进入相应目录，并覆盖相关文件
cd /usr/local/sbin
cp /usr/local/sbin/iptables /sbin/   
cp /usr/local/sbin/iptables-restore /sbin/
cp /usr/local/sbin/iptables-save /sbin/
```

#### 2.3.5 检验fullcone

```shell
iptables -t nat -A POSTROUTING -o eth0 -j FULLCONENAT # 没有错误
lsmod | grep xt_FULLCONENAT # 有结果
```

[![正常结果](https://s3.ax1x.com/2020/11/13/Dp5hIP.png)](https://imgchr.com/i/Dp5hIP)

#### 2.3.6 开启自动加载

```shell
mv ~/fullcone/netfilter-full-cone-nat/xt_FULLCONENAT.ko  /lib/modules/$(uname -r)/
depmod
```

新建编辑`/etc/modules-load.d/fullconenat.conf`:

```shell
xt_FULLCONENAT
```

验证:

```shell
reboot
lsmod | grep xt_FULLCONENAT
```

*ps*:

此处如果没有结果，可能是内核更新了，请重新执行

```shel
cd ~/fullcone/netfilter-full-cone-nat
make
mv ~/fullcone/netfilter-full-cone-nat/xt_FULLCONENAT.ko  /lib/modules/$(uname -r)/
depmod
```

然后重启即可

### 2.4 修改wireguard配置

在落地机器修改`/etc/wireguard/wg0.conf`:

将`postup` 尾部的`iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`修改为`iptables -t nat -A POSTROUTING -o eth0 -j FULLCONENAT`

将`postdown`尾部的`iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE`修改为`iptables -t nat -D POSTROUTING -o eth0 -j FULLCONENAT`

重启落地机器的wireguard网卡:

```shell
wg-quick down wg0
wg-quick up wg0
```

此时我们已经实现了一个具有NAT1的**虚假**的`IPLC`。

### 2.5 加速游戏

本文用到的游戏加速软件为[netch](https://github.com/NetchX/Netch)，netch支持常见的的远程代理协议，具体在其github有提供。

至于那些@#￥%……&……的■■■的■■■■，本文暂且不表，各位自由发挥。

我们只需要将这些服务搭载在局域网的中转机器上， 然后通过PC的netch软件连接，就可以实现nat1的游戏加速了。

唯一需要注意的一点是：搭建的这个服务需要支持完整的udp，否则之前的一切努力都白费了。

当然搭建单纯的socks5服务(比如:dante-server)也是可以的，但是由于本人遇到一些奇怪的问题，所以此处也就不贴教程了。



## 3. DLC

### 3.1 pystun

pystun是一个可以检测linux服务器nat类型的工具。

要安装pystun需要借助于python的包管理工具`pip`。

执行下面命令安装`pip`:

```shell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py
```

通过`pip`安装pystun:

```shell
pip install pystun3
```

执行`pystun3`应该会得到如下结果:

```shell
NAT Type: Full Cone
External IP: your ip
External Port: Random Port
```

其中NAT Type对应了你的机器NAT类型。具体NAT类型的分别敬请google。在本文中想要的到最终结果为NAT1，即FULLCONE。

### 3.2 udp2raw

因为wireguard使用udp传输，电信经常在我玩的热火朝天的时候给我断流，所以我需要在wireguard外面套上一层udp2raw。

具体实现步骤如下：

首先前往[下载地址](https://github.com/wangyu-/udp2raw-tunnel/releases)下载udp2raw，然后在落地机器开启server：

```shell
#记得更改udp2raw的路径以及password
nohup path/to/udp2raw -s -l0.0.0.0:9898 -r 127.0.0.1:52540 --raw-mode faketcp -a -k password >/root/app/logs/udp2raw.log 2>&1 &
```



然后在中转机器开启client:

```shell
# 因为后续要更改wireguard的endpoint到中转机器的udp2raw,所以要先添加转发规则
ip route add 1.2.3.4 via $(ip route | awk '$1=="default" {print $3}')
# 记得修改udp2raw的路径、ip和password
nohup path/to/udp2raw -c -r 1.2.3.4:9898 -l 127.0.0.1:52540 --raw-mode faketcp -k password >/root/udp2raw.log 2>&1 &
```

修改中转机器的wireguard设置`/etc/wireguard/wg0.conf`

将`Endpoint`修改为`127.0.0.1:52540`

重启wireguard

```shell
wg-quick down wg0
wg-quick up wg0
```

至此完成了将wg伪装为tcp的过程，对于丢包严重的朋友，还可以使用udp2raw作者开发的[udpspeeder](https://github.com/wangyu-/UDPspeeder)，具体使用方法请参考其github，关于udp2raw的一些具体用法也有介绍。

## 鸣谢

[从DNAT到netfilter内核子系统，浅谈Linux的Full Cone NAT实现](https://blog.chionlab.moe/2018/02/09/full-cone-nat-with-linux/)

[Centos 7当网关启用Fullcone nat](https://www.jianshu.com/p/88e7cd6a0c95)

[Installing nftables from sources on Debian](https://isabelcmdcosta.medium.com/installing-nftables-from-sources-on-debian-eb8c24bbc5ca)

[netch](https://github.com/NetchX/Netch)

[udp2raw-tunnel](https://github.com/wangyu-/udp2raw-tunnel)

[WireGuard 搭建和使用折腾小记](https://10101.io/2018/11/10/wireguard)