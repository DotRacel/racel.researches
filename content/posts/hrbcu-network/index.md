---
title: "哈尔滨商业大学 校园网络优化深入浅出"
description: "记录我在不断折腾的过程中，所遇到的各种问题和得到的解决方案。"
date: "2024-4-24"
tags: ["校园网", "openwrt", "路由器", "计算机网络"]
categories: ["中文"]
---

# 面对的几个主要问题
- 学校的学生寝室楼的宽带运营商处于垄断状态，均为中国联通，以`80元/月 50Mbps/2终端`的价格提供给学生，昂贵且难用。
- 学校寝室楼的移动流量信号差，速率低，延迟高且有流量限制，属于勉强可以使用的程度。  
-----------------------------
实际上，针对第二个问题，我曾经尝试过使用 CPE(5G模块为`RM520N-GL`)，但难以忍受高延迟和低带宽，最后放弃使用。  
这样的话，我只能尝试采用学校联通所提供的宽带，想办法尽可能优化。

## 优化方案
1. A校区的寝室楼宽带网络提供了**光纤入寝**，租用宽带的同时也可以抵押 300 元得到校园网定制的中国联通光猫。虽然抵押的光猫有WIFI功能，但是上游下发注册会自动关闭管理后台和WIFI功能。因此，可以外接无线路由器，实现同寝室网络共享。
2. 上述基础的网络共享实际上只使用了一个终端，而通过单线多播可以将多终端的速率叠加，提高带宽。

# 优化过程
## 同寝室网络共享
### 硬件要求
| 设备      | 用途           | 
| --------- | -------------- | 
| 光猫      | 光信号调解      | 
| 无线路由器 | 供终端设备无线连接 | 

我购置了一台`红米 RM2100`，价格低廉，OpenWRT 固件数量多，但缺点就是内存太小，单设备无法承担单线多播的责任  
无线路由器既可以选择付费固件（往往捆绑硬件出售），也可以自行购买，安装自定义固件  
但是，安装的固件**至少**需要有`UA2F`插件，用于防止被上游上网行为管理系统的多终端检测阻断  
这里的路由设置教程遵循`OpenWRT`系统
### 路由设置
1. 将路由器的 WAN 口连接到光猫的 LAN 口
2. 如果你知道路由器的默认SSID和密钥，你可以使用终端连接到路由器；如果你不知道，你可以将你有 RJ45 网口的电脑连接到路由器的任意一个 LAN 口。OpenWRT 的默认登录地址即为网关，默认网关地址和管理页面的账户一般在固件下载地址有所提供。这时，登录 OpenWRT 的管理页面。 
3. 然后在路由器中将 WAN 口设置为 DHCP 客户端协议。这时，在`接口`中可以看到`WAN`已经获取到了 IPv4 地址[^1]
4. 勾选 `UA2F` 的全部选项并启动
5. 在`无线`选项中，设置好 SSID 和密钥，并启用。  
-------------------------------------
经过上述的简单设置，此时可以访问校园网的[终端登录页面](http://172.17.100.10/a79.htm)，登录后即可得到公网访问权限。  
并且只有在 24H 内没有过连接，登录才会过期。因为，晚上断电，但是早晨路由器自启动就会维持这个登录状态。

[^1]: 校园网截至现在还没有支持 IPv6
## 单线多播提高带宽
这是最麻烦的一步，因为单线多播带来了很多不兼容性和硬件要求  
概括来说就是，使用 OpenWRT 的 MWAN3(负载均衡) 和 Syncdial(多播) 插件来实现单线多播，以及多播后的负载均衡。 
但是 MWAN3 因为使用了 iptables 的标记功能，会与同时使用这个功能的 UA2F 发生冲突，因为我们也无法继续使用 UA2F 了。这样，我们只能使用 UA3F[^2] 搭配 Clash 使用了。这要求路由器的内存不能过小，但是我的 RM2100 内存过小，因此我购买了 `FriendlyElec R5s` 软路由，充当路由器的上游，仅让 RM2100 作为 AP 使用。  
首先需要将 RM2100 配置为 AP，操作较为简单这里不做赘述。  
### 固件 & 插件安装使用
- 我给我的 R5s 安装了 QWRT 闭源固件，也可以选择其他固件，差别不大
- 需要安装多播(`luci-app-syncdial`)和负载均衡(`luci-app-mwan3`)插件
- 设置好多播线路数量，多播插件会自动配置负载均衡，可以根据需要自行修改策略
- 自动配置的负载均衡接口是 PPPoE，需要手动将这些接口修改为 DHCP 客户端
- 同时根据 UA3F 给出的 OpenWRT Luci 安装教程，安装 UA3F
- UA3F 还需要 Clash 来进行流量代理，可以安装 OpenClash 或者 ShellClash，这里我使用了 OpenClash，安装和设置起来可以直接通过网页界面进行，操作简便[^3]
- (可选) 安装 AdGuard Home，建立私人 DNS，防止劫持，进行广告过滤。
-----------------------------------
由于具体的安装过程在网络上都可以搜索得到，这里只给出我踩过的坑。
- 多播的设置里，不要勾选`绑定物理接口`
- 高版本的 MWAN3 在设置好后，在每次设备启动后，可能会出现接口状态被禁用的情况，这时候需要手动执行`mwan3 restart`。来彻底解决这个问题，可以在启动项脚本中添加`mwan3 restart`这条指令，实现在每次重启后自动对 MWAN3 进行重启
- OpenClash 中不要开启绕过中国大陆，否则有部分流量将不会经过 UA3F，可能会造成网络阻断

### 负载均衡登录
如果需要手动登录，可以使用`curl --interface <URL>`  
当然，也可以借助负载均衡提供的事件响应脚本，在每次被强制登出后，自动执行重新登录  
下面给出我的 python 脚本
```python
#!/usr/bin/python3
import requests
import re
import random
import socket
import sys
import subprocess

INIT_URL = "http://172.17.100.10/"
HRBCU_PORTAL_API = "http://172.17.100.10:801/eportal/portal/"

interface = sys.argv[1]
device = sys.argv[2]
action = sys.argv[3]


class HTTPAdapterWithSocketOptions(requests.adapters.HTTPAdapter):
    def __init__(self, *args, **kwargs):
        self.socket_options = kwargs.pop("socket_options", None)
        super(HTTPAdapterWithSocketOptions, self).__init__(*args, **kwargs)

    def init_poolmanager(self, *args, **kwargs):
        if self.socket_options is not None:
            kwargs["socket_options"] = self.socket_options
        super(HTTPAdapterWithSocketOptions,
              self).init_poolmanager(*args, **kwargs)


adapter = HTTPAdapterWithSocketOptions(socket_options=[(
    socket.SOL_SOCKET, socket.SO_BINDTODEVICE, device.encode('utf-8'))])
session = requests.session()
session.mount("http://", adapter)
session.mount("https://", adapter)

def logging(str):
    print("[*] " + str)
    subprocess.call(
        [f'logger "[*] ' + str + '"'], shell=True
    )

def init():
    logging("initializing")
    global ip, randnum
    init_res = session.get(INIT_URL)
    # print(init_res.content)
    if 'hsydxka' in str(init_res.content):
        ip = re.search("v4ip='(.*?)'", init_res.text).group(1)
    else:
        ip = re.search("v46ip='(.*?)'", init_res.text).group(1)
    randnum = str(random.randint(1000, 9999))
    logging("client_ip:" + ip)
    logging("randint:" + randnum)


def login():
    params = {
        'callback': "dr1003",
        'login_method': 1,
        'user_account': username,
        'user_password': password,
        'wlan_user_ip': ip,
        'wlan_user_mac': '000000000000',
        'wlan_ac_ip': '',
        'wlan_ac_name': '',
        'jsVersion': 4.1,
        'terminal_type': 1,
        'v': randnum,
        'lang': "zh"
    }
    res = session.get(HRBCU_PORTAL_API + "login", params=params)
    logging("response: " + res.text)
    if '认证成功' in res.text:
        logging('logged in successfully')
    elif '已经在线' in res.text:
        logging('login failed')


if __name__ == '__main__':
    if action == "disconnected" and device.find("macvlan") != -1:
        global username, password

        username = ""
        password = ""

        try:
            init()
            login()
        except Exception:
            subprocess.call(
                # [f"ifdown {interface} && sleep 2 && ifup {interface} && sleep 2"], shell=True
                [f"sleep 30"], shell=True
            )

            init()
            login()

```  
同时在事件响应脚本内追加一行``/usr/bin/python3 /root/hrbcu-net.py $INTERFACE $DEVICE $ACTION``即可


[^2]: [校园网防检测: UA3F - 新一代 UA 修改方法](https://blog.sunbk201.site/posts/ua3f/)  
[^3]: [UA3F 与 Clash 从零开始的部署教程](https://sunbk201public.notion.site/UA3F-Clash-16d60a7b5f0e457a9ee97a3be7cbf557)

### AdGuard Home 配置
