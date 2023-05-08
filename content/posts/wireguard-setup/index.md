---
title: 使用 WireGuard 无缝接入内网
date: 2020-07-27T10:54:48+08:00
categories:
- 折腾
tags:
- wireguard
- vpn
- linux
---



去年心血来潮想搞个 NAS 放在家里，奈何囊肿羞涩，最终只能捡垃圾捡了个蜗牛星际，又因为种种原因吃了近一年的灰。最近又比较心血来潮，想起来了在角落中已经蒙尘的蜗牛星际。

在某宝上买的机子已经由卖家预装了黑群晖，用起来也是完全的傻瓜式操作，免去了自己折腾各种软件浪费时间。目前仅仅作为一个 `samba` 文件服务器，在局域网中共享一些视频或者其他资源。



<!-- more -->



OK，开始正题。现在需要支持在外面远程控制 NAS 下载一些东西。目前有这么几种方案：

- 直接端口映射到内网的 NAS

  这种方案需要你有一个公网 IP，在路由器中将需要外网访问的端口直接映射到内网的 NAS 对应的端口上即可，操作也比较简单，速度则取决于你的上行带宽。

- 使用 [frp](https://github.com/fatedier/frp) 之类的内网穿透工具

  这种方案需要一个拥有公网 IP 的服务器，可以将内网的端口映射到这台服务器指定的端口，在外面直接访问服务器的端口即可。具体的配置方法也比较简单，在此不再赘述，这个方案的速度却决于你的这台服务器的带宽。

- 直接上 VPN

  这种方案也需要一个拥有公网 IP 的服务器，相较上面两种方案，这个可能不是那么的方便。但有个好处就是可以透明地访问整个内网的任意主机，速度也是取决于这台服务器的带宽。



考虑到后面可能会折腾好多东西(虽然目前就这一个)，干脆将搞个 VPN 直接支持整个内网的访问吧，这样也不用一个端口一个端口地去映射了。但缺点也显而易见，访问内网就得连接 VPN，还是有那么一点点麻烦。



那么用哪个 VPN 方案呢？



- OpenVPN
- PPTP
- L2TP&IPSec
- WireGuard



先说 OpenVPN，这个安全性是毋庸置疑的，可以说是非常安全。但缺点是配置繁琐，而且性能可能不大好。

PPTP 和 L2TP 在大部分平台中都有内置，配置起来非常方便，性能也挺好的，但不是很安全。

WireGuard 是一个极其迅速而且简单的 VPN，它的目标是更快，更简单，更小，并且比 IPSec 更好用(来自 WireGuard 官网)。目前 WireGuard 已经合并进入了 Linux 内核。



可以看一下性能对比(来自 [WireGuard 官网](https://www.wireguard.com/performance/#results))

![image-20200727151702974](image-20200727151702974.png)



话不多说，开始安装吧。网络拓扑大致如下。

![wireguard-network](wireguard-network.svg)

其中：

- `1.1.1.1` 为服务器的公网 IP
- `192.168.1.0/24` 为内网的 IP 段
- `192.168.2.0/24` 为 WireGuard 的 VPN 内网网段



最终实现的效果是，电脑和手机通过连接 `1.1.1.1` 上的 VPN 服务，接入其他两个网段。

## 安装 WireGuard

WireGuard 不区分服务端与客户端，安装也比较简单，不出意外的话可以一行命令直接安装。

具体可参考 [Installation - WireGuard](https://www.wireguard.com/install/)，基本上涵盖了大部分 Linux 发行版，Windows 以及 MacOS，也包含了 Android, iOS 的安装包。

由于群晖不便于配置服务自启动（应该是我不会，没有 `systemd` 用的顺手），干脆在黑群晖中装个 Debian10 的虚拟机当作接入内网的网关（也就是上图中的 NAS）。

WireGuard 使用 `wg` 命令来进行控制，并且提供了一个 `wg-quick` 命令，可用来快速管理 VPN 连接。



## 生成密钥

```bash
# 生成私钥
$ wg genkey > priv_key
$ cat priv_key
qNzPeN726rM8L+JiSYeqm5zi6VwFlaa4OZhaCNPa4Uk=

# 生成公钥
$ wg pubkey < priv_key > pub_key # 使用私钥来生成公钥
$ cat pub_key
NtMtwyEB6tsQxOA5Z6xhuQkhwTFXeGPmbnirCA4lqgw=
```



WireGuard 的服务端与客户端是对等的。生成密钥的步骤适用于服务端，也适用于客户端。

## 服务器端配置

WireGuard 的配置文件一般情况下放在 `/etc/wireguard` 中，且为了安全考虑，需要将这个目录的所有者设置为 `root`，权限设置为 `700`。

一般情况下，配置文件可依次命名为 `wgX.conf`，其中 `X` 为任意的编号，`wgX` 最终也是创建出来的网络接口的名称。

此处，在服务端与客户端中，都将命名为 `wg0.conf`



先来看一下最基础的配置文件 `/etc/wireguard/wg0.conf`

```ini
[Interface]
Address = 192.168.2.1/32
ListenPort = 28385
PrivateKey = Server private key

[Peer]
PublicKey = Client public key
AllowedIPs = IP Range
```



就是这么简单。分为两种配置组：

- `Interface`

  这个配置项为本地接口的配置，其中：

  - `Address` 为 VPN 连接的本地 IP 地址
  - `ListenPort` 作为服务端需要声明一个监听的端口，WireGuard 使用 UDP 协议，这个端口可以任意填写。需要保证防火墙已开放 UDP 的这个端口
  - `PrivateKey` 为上一步生成的私钥

  

- `Peer`

  这个为对端的配置，如果有多个，则需添加多个 `Peer` 配置，服务端的 `Peer` 配置项即定义了各个可连接的客户端。其中：

  - `PublicKey` 为对端的公钥
  - `AllowedIPs` 在[配置路由](#配置路由)会讲到

## 客户端配置

客户端的配置与服务端的配置基本一致。每个端都需要生成各自的密钥对。

下面的为 NAS 的配置文件 `/etc/wireguard/wg0.conf`

```ini
[Interface]
PrivateKey = Client private key
Address = 192.168.2.2/32

[Peer]
PublicKey = Server private key
AllowedIPs = IP Range
Endpoint = 1.1.1.1:28385
```



客户端与服务端不同的地方在于：

- `Interface` 配置中没有了 `ListenPort`
- `Peer` 即为服务端，与服务端不同的地方在于多了一个 `Endpoint`



一般情况下，只有主动发起连接的端需要在 `Peer` 中指定 `Endpoint`，此处为服务器的公网 IP 和监听的端口。



基础的配置已经完成，现在可以直接使用 `wg-quick up wg0` 启动 VPN，其中 `wg0` 为配置文件的名称。



## 配置路由

通过上面的配置成功启动 VPN 后，即使连接成功，也不能访问 `192.168.1.0/24` 和 `192.168.2.1/24` 网段，这是因为截至目前，我们仅仅配置了 VPN 隧道，但路由还没有配置，发出去的 IP 包不知道应该被发往哪去。



按照预先设计的网络架构，在外面通往内网的流量大致流向为：

```
192.168.2.1(WireGuard 服务端) <------+ 192.168.2.X(客户端)
       +
       |
       v
  192.168.2.2(NAS)
       +
       |
       v
192.168.1.0/24(内网网段)

```

按照上图所示，我们需要：

- 外网客户端将发往 `192.168.1.0/24` 和 `192.168.2.0/24` 网段的包转发到服务端(即 `192.168.2.1`)
- 服务端将  `192.168.1.0/24`(内网) 网段的包发给 NAS (即 `192.168.2.2`)
- NAS 将来自于 `192.168.2.0/24` 网段发往 `192.168.1.0/24` 网段的包进行 SNAT



上面的配置中的 `AllowedIPs` 可以被认为是路由规则的定义。意思是：

- 当出向包的目的 IP 为 `AllowedIPs` 定义的网段时，将该包从 VPN 隧道发送出去
- 当从 VPN 隧道进来的包的源 IP 不匹配 `AllowedIPs` 定义的网段时，直接将包丢弃



### 服务端

所以按照上面的包转发规则，可以得出各个客户端 `Peer` 的 `AllowedIPs`

- NAS `192.168.2.2/32, 192.168.1.0/24`
- 其他客户端 `192.168.2.x/32` 为各自的 IP 地址



### 配置 `iptables`

除了配置 `AllowedIPs` 外，在 Linux 中还需要配置 `iptables`。

如果`iptables` 的 `filter` 表的 `FORWARD` 链的默认策略不是 `ACCEPT` 则还需要添加规则，使包可以被转发。

**如果默认策略是 `ACCEPT` 则可以跳过这部分。**

```bash
$ iptables -L filter -n -v
Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
...
```

虽然可以将 `iptables` 的默认策略改掉，但不推荐这样做。

关于 `iptables` 的教程可以参考 [iptables | 朱双印博客](http://www.zsythink.net/archives/tag/iptables/)



```bash
iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT # 放行 VPN 网段
iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.1.0/24 -j ACCEPT # 放行 VPN 转发到内网的流量
iptables -I FORWARD -s 192.168.1.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT # 放行内网 转发到 VPN 的流量
```



在 WireGuard 的配置文件中，可以在 `Interface` 下指定多个 `PostUp` 和 `PostDown`，分别在 VPN 启动和停止后按照定义的顺序依次执行。



服务端配置

```ini
[Interface]
Address = 192.168.2.1/32
ListenPort = 28385
PrivateKey = Server private key

PostUp = iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
PostUp = iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.1.0/24 -j ACCEPT
PostUP = iptables -I FORWARD -s 192.168.1.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT

PostDown = iptables -D FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
PostDown = iptables -D FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.1.0/24 -j ACCEPT
PostDown = iptables -D FORWARD -s 192.168.1.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT

[Peer]
PublicKey = Client public key
AllowedIPs = IP Range
```



另外还需要开启 Linux 内核的 IP 包转发

```bash
$ sysctl net.ipv4.ip_forward
0
```

如果上面的输出为 `0` 则 Linux 内核默认会丢弃所有目的 IP 不为本机的包。

编辑 `/etc/sysctl.conf`， 将 `net.ipv4.ip_forward = 0` 中的 `0` 改为 `1`。然后 `sysctl -p` 使其生效。



### NAS

NAS 中的 `Peer` 为服务端，则 `AllowedIPs` 应为 `192.168.2.0/24`。



同时，NAS 需要负责将来自 `192.168.2.0/24` 网段的发往 `192.168.1.0/24` 的包进行 SNAT。

```ini
[Interface]
PrivateKey = Client private key
Address = 192.168.2.2/32

PostUp = iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -j SNAT --to-source 192.168.1.2
PostDown = iptables -t nat -D POSTROUTING -s 192.168.2.0/24 -j SNAT --to-source 192.168.1.2

[Peer]
PublicKey = Server public key
AllowedIPs = 192.168.2.0/24
Endpoint = 1.1.1.1:28385
```

注意其中的 `--to-source 192.168.1.2` 的 IP 地址为 NAS 在内网中的 IP 地址。

同样，在 NAS 上，也需要确认已开启包转发。



## 总结



最终的配置文件如下

- 服务端

  ```ini
  [Interface]
  Address = 192.168.2.1/32
  ListenPort = 28385
  PrivateKey = <Server private key>
  
  PostUp = iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
  PostUp = iptables -I FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.1.0/24 -j ACCEPT
  PostUP = iptables -I FORWARD -s 192.168.1.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
  
  PostDown = iptables -D FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
  PostDown = iptables -D FORWARD -s 192.168.2.0/24 -i wg0 -d 192.168.1.0/24 -j ACCEPT
  PostDown = iptables -D FORWARD -s 192.168.1.0/24 -i wg0 -d 192.168.2.0/24 -j ACCEPT
  
  # NAS
  [Peer]
  PublicKey = <Client public key>
  AllowedIPs = 192.168.2.2/32, 192.168.1.0/24
  
  # 其他客户端 ...
  [Peer]
  PublicKey = <Client public key>
  AllowedIPs = 192.168.2.x/32
  ```

  

- NAS 

    ```ini
    [Interface]
    PrivateKey = <Client private key>
    Address = 192.168.2.2/32

    PostUp = iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -j SNAT --to-source 192.168.1.2
    PostDown = iptables -t nat -D POSTROUTING -s 192.168.2.0/24 -j SNAT --to-source 192.168.1.2

    [Peer]
    PublicKey = <Server public key>
    AllowedIPs = 192.168.2.0/24
    Endpoint = 1.1.1.1:28385
    ```



- 其他客户端

  ```ini
  [Interface]
  PrivateKey = <Client private key>
  Address = 192.168.2.X/32
  
  [Peer]
  PublicKey = <Server public key>
  AllowedIPs = 192.168.2.0/24, 192.168.1.0/24
  Endpoint = 1.1.1.1:28385
  ```

  

在各个机器上分别启动服务

```bash
wg-quick up wg0
```

通过 `wg` 命令可查看当前状态

```shell
$ wg # 服务端
interface: wg0
  public key: Public key
  private key: (hidden)
  listening port: 28385

peer: Client public key # NAS
  endpoint: x.x.x.x:41995
  allowed ips: 192.168.2.2/32, 192.168.1.0/24
  latest handshake: 36 seconds ago
  transfer: 132.48 KiB received, 49.25 KiB sent

peer: Client public key # 外网的电脑
  endpoint: x.x.x.x:60941
  allowed ips: 192.168.2.3/32
  latest handshake: 1 minute, 11 seconds ago
  transfer: 62.54 KiB received, 124.75 KiB sent
  
# ...
```



在外网尝试 ping 一下内网的主机

```bash
$ ping 192.168.1.1 # 路由器
PING 192.168.1.1 (192.168.1.1) 56(84) bytes of data.
64 bytes from 192.168.1.1: icmp_seq=1 ttl=63 time=113 ms
64 bytes from 192.168.1.1: icmp_seq=2 ttl=63 time=90.10 ms
# ...
```



OK，完成，现在可以访问任意内网的机器了。