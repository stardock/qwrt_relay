# qwrt_relay
Relay option for openwrt


一、无线桥接（WDS）

先放一张拓扑图：

主要是参考官方的文章：[OpenWrt Wiki] 使用Atheros和MAC80211 WDS实现无线网桥（无线中继）

简单总结一下：

1、两个路由器都需要使用WDS，路由器R1的wifi需要设置为“AP接入点（WDS）”，路由器R2的wifi设置为“客户端Client（WDS）”。

2、路由器R2的dhcp关闭，地址设置为与路由器R1同网段的一个地址（其实设置为其他网段的地址也可以，这里记得，不管设置成什么地址，电脑需要设置成同网段的静态IP才能重新连接上路由器R2）。

同一网段的目的是为了路由器R2自身也加入到整个局域网，这里还需要配置其他几个地方（不设置也可以，通过路由器R2连接的手机/电脑可以正常上网，只是路由器R2不通网）：

    至网络, DHCP/DNS， 设置DNS forwardings（DNS 转发）为无线接入点路由的ip地址。
    至网络、接口、Lan、修改，将“IPv4 网关”设置为无线接入点路由的ip地址。至Physical Settings（物理设置）并启用STP。

3、路由器R2的wifi设置，连接R1的wifi时，需要注意信道、SSID、密码的一致，还有网络要选lan，防火墙默认是lan。

（关于网络选择lan，做下补充：1、建议连接路由器R2时，不要用扫描加入，可以用“添加”，然后把WIFI信息自己填写，因为扫描加入会增加一个新的接口。2、选择LAN接口可能无法保存，这种情况需要去接口->lan->修改->物理设置，在网桥接口里把无线网络选上，之后在无线中，可以看到网络默认选择了lan。）

二、无线桥接（relayd）

先放拓扑图：

参考官方的文章：[OpenWrt Wiki] Wi-Fi extender / repeater / bridge configuration

主路由没有要求，192.168.1.1网段，默认的dhcp开启，WIFI为普通AP接入点模式。

主要是从路由的配置流程：

准备：需要确保已安装的两个包：luci-proto-relay和 relayd。

1、LAN配置

关闭dhcp、IP设置固定地址192.168.2.1（与主路由不同网段）。

之后电脑需要设置192.168.2.1的同网段静态IP，重新登录页面继续配置。

2、无线配置

点击无线接口的扫描按钮，加入上级路由的无线网络。将防火墙区域设置为lan。

无线的网络接口取名为wwan。

3、无线网络接口配置

网络->接口->wwan，点击后修改：

    协议：         静态地址

    IP地址：     192.168.1.30

    子网掩码：  255.255.255.0

    网关：         192.168.1.1

    使用自定义的 DNS 服务器： 192.168.1.1

4、添加中继桥

网络->接口->添加新接口，点击后修改：

    新接口的名称：repeater_bridge

    新接口的协议：中继桥

    本地IP地址：192.168.1.30

    网络间中继：lan、wwan

以上配置完成后，电脑设置为自动dhcp，连接从路由，可以从主路由获取192.168.1.1网段的ip。

三、无线中继

无线中继相对wds、relayd比较简单，因为不用实现在同一网段。

主路由默认的dhcp开启，WIFI为普通ap接入点模式。

从路由设置：

1、从路由dhcp默认开启，网段需要设置跟主路由不同的网段。

2、点击无线接口的扫描按钮，加入上级路由的无线网络。将防火墙区域设置为wan，无线的网络接口取名为wwan。

电脑连接后，IP地址是从路由分配的，主路由网络无法访问从路由网络。

添加IPv6支持

Activate IPv6 support on your Internet box, this will get you a public IPv6 prefix. We will now activate IPv6 on our Wi-Fi extender to allow for Stateless Address Autoconfiguration (SLAAC) of your public IPv6 addresses and IPv6 traffic.

1. Go to Network / Interfaces and create a new interface. Name it WWAN6, using protocol DHCPv6, cover the WWAN interface. In the Common Configuration of the new interface, configure: Request IPv6 address: disabled. In the Firewall settings: check that the “lan / repeater bridge…” line is selected. Leave the other settings by default, especially, leave the “Custom delegated IPv6-prefix” field empty. On the Interfaces / overwiew page check that the WWAN interface gets a public IPv6 address.

2. Edit the LAN interface settings, DHCP server / IPv6 settings: check/modify the following settings: Router Advertisement Service: relay mode, DHCPv6 service: disabled, NDP-Proxy: relay mode.

3. Open a SSH session on your OpenWrt device. Issue the following commands:
```
uci set dhcp.wan.interface=wwan
uci set dhcp.wan.ra=relay
uci set dhcp.wan.ndp=relay
uci set dhcp.wan.master=1
uci commit
```
We suppose that you created a wwan interface when you joined to the other Wi-Fi network as suggested earlier in this guide; otherwise, change the dhcp.wan.interface=… line accordingly.

That's it. Restart ophcpd (LuCI System/Starup page, or /etc/init.d/odhcpd restart) and your IPv6-network should begin to configure itself. Connected IPv6-enabled devices should get their public IPv6 addresses, derived from your public IPv6 prefix, and IPv6 traffic should go through your Wi-Fi extender.


Ref: 
https://blog.csdn.net/qq_26202991/article/details/133127238  
https://openwrt.org/zh/docs/guide-user/network/wifi/relay_configuration
