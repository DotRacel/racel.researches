---
title: "对于所在学校的校园网络了解和研究"
description: "在对寝室网络不断的折腾中所了解到的一些事情。"
date: "2024-3-3"
tags: ["校园网", "openwrt", "路由器", "计算机网络"]
categories: ["中文"]
---

# 情况概述
我校使用的校园网是由`Dr.Com 哆点网络`提供的鉴权方案，其中通过自建云服务，对接了哆点的`Portal`登录协议。
## 网络类型
- 校内WIFI: 校内基本覆盖了Wifi，连接后自动跳转到 _哆点网络验证页面_。 
- 寝室有线网络：寝室内有光纤口，学校内仅有的联通营业厅提供光猫抵押服务，通过其抵押的校园网专用光猫（带Wifi功能），连接光纤口后会自动注册下发配置，将管理后台彻底关闭，关闭Wifi功能，仅支持LAN口的DHCP。
## 验证方式
普通未绑定运营商账号的学生账户，与学校的CAS系统账号密码同步；而绑定了运营商账号的账户，则是普通的学生账号但是添加了`2`作为后缀区分，密码则是完全相同。  
未绑定运营商账号登录时，除了密码之外的参数都是明文，而绑定了运营商账号的登录请求参数 _全都是明文_。
成功登录后，校园网会以`客户端IP`为 __唯一标识__ 用以作为判别终端数量的一个基础标准。 

# 校园网针对终端限制进行的检测措施
## User-Agent 检测
于 _2024年第一学期_ 勘测发现，校园网新增了 __UA检测__ 用以进一步对多终端共享网络这一行为进行限制。检测到多终端后，便会进行 _5分钟的连接阻断_。  
`User-Agent`是HTTP请求的一个请求头，其中包含了浏览器信息和设备信息等，可以用于区分不同终端。应该注意到的是，User-Agent在HTTP请求中是 __明文__ 的，在HTTPS的第一个请求中也是明文的。   
``Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36`` 便是一个正常的`User-Agent`内容。  
你也可以通过[专门的UA检测网站](http://ua.233996.xyz/)来查看你当前的`User-Agent`。
### 尚未发现的检测措施
目前我校仅仅部署了UA检测，尚未发现其他检测措施，但也了解到了一些其他的检测措施。

> 基于 IPv4 数据包包头内的 TTL 字段的检测  
> DPI (Deep Packet Inspection) 深度包检测技术  
> 基于 IPv4 数据包包头内的 Identification 字段的检测  
> 基于网络协议栈时钟偏移的检测技术  
> Flash Cookie 检测技术  
>
> 来自 sunbk201[^1]

[^1] https://blog.sunbk201.site/posts/crack-campus-network/

# 校园网优化方案
## 对抗终端限制
首先我购买了一个路由器用以伪装成一个终端，连接寝室光猫使用，从而达成网络共享的目标。我购买的是`红米 AC2100`[^2]，价格低廉，可玩性高，使用了集成了`UA2F`的`OpenWRT`固件[^3]。  
学校目前仅仅部署了`User-Agent检测`，这也是最常见的一个检测，其中有多种方案来对抗。
- `UA2F`[^4] 为多种设备提供了`User-Agent`修改方案，其使用`iptables`对流量进行判别修改，会与其他使用`connmark`的程序发生冲突，例如`mwan3负载均衡`。
- `UA3F`[^5] 是一个`SOCK5`代理服务器，它通过接管通过`SOCK5`客户端转发来的流量，对`HTTP`数据包进行修改。但是针对性能较弱的路由器，可能会出现内存不足等问题。
- 可以直接使用`代理工具`，将流量重定向到`代理服务器`，这一过程是加密的，因此也可以用于绕过UA检测。
## 多终端速率叠加
我校校园网以**高出市场价几倍**的资费进行出售，实在是可恶。一般购买的是 `50M / 2个终端` 套餐，因此可以考虑通过`单线多播`，实现50M速率的叠加。
### `macvlan` 创建虚拟网口 实现单线多播
安装过程不做赘述，可以搜到很多的 `macvlan` 安装教程、

- 通过`ifconfig`得到自己真实的wan口名称，如`eth1`。
- `ip link add link eth1 name macvlan0 type macvlan` 创建一个虚拟网口，如果有两个终端则可以创建两个。
### `mwan3` 实现负载均衡
前面提到已经实现了多个虚拟网口，现在只需要使用`mwan3`对这几个虚拟网口进行负载均衡即可达到速率叠加。
## 自动认证
- 可以编写`shell脚本`, 结合`crontab`实现定期运行
- 使用`nettask`[^7]设置事件触发，同时也需要配合脚本。
## 打包出售的校园网绕过方案
`GSWIFI`[^6], `ASWIFI` 
## 利用已经开放端口，外部架设免流代理，从而无需认证即可访问外网
经测试，`哆点认证`未认证也不会封锁53端口。

[^2] https://www.mi.com/rm2100  
[^3] [自编译Redmi AC2100固件，集成UA2F等，可用于校园网防检测等功能](https://www.right.com.cn/forum/thread-8315729-1-1.html)  
[^4] [校园网路由器多设备伪装指北](https://learningman.top/archives/304)  
[^5] [校园网防检测: UA3F - 新一代 UA 修改方法](https://blog.sunbk201.site/posts/ua3f/)  
[^6] 固件做了正版验证，官方公布的固件刷机后无法使用。https://www.guangsuwifi.com/  
[^7] [luci-app-nettask](https://github.com/lucikap/luci-app-nettask)  