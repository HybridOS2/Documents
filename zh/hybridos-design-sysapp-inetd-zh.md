# 合璧操作系统网络管理应用的设计

Subject: HybridOS Network Interface Manager  
Version: 2.0  
Author: Vincent Wei  
Category:  Specification  
Creation Date: Nov., 2020  
Last Modified Date: May 15, 2023  
Status: Release Candidate  
Release Name: 硕鼠  
Language: Chinese

**版权声明**

版权所有 &copy; 2020 ~ 2023 北京飞漫软件技术有限公司  
保留所有权利

此文档不受合璧操作系统相关软件开源许可证的管辖。

飞漫软件公开此文档的目标，用于向开发者解释合璧操作系统的设计原理或者相关规范。在未获得飞漫软件书面许可之前，任何人不得复制或者分发本文档的全部或部分内容，或利用本文档描绘的技术思路申请专利、撰写学术论文等。

本文涉及到的飞漫软件或其合作伙伴的注册商标或商标，有关其详细列表，请查阅文档末尾。


**目录**

[//]:# (START OF TOC)

- [1) 一般信息](#1-一般信息)
- [2) 数据总线接口](#2-数据总线接口)
   + [2.1) 远程过程](#21-远程过程)
      * [2.1.1) 打开网络设备](#211-打开网络设备)
      * [2.1.2) 关闭网络设备](#212-关闭网络设备)
      * [2.1.3) 查询网络设备状态](#213-查询网络设备状态)
      * [2.1.4) 开始扫描网络热点](#214-开始扫描网络热点)
      * [2.1.5) 停止网络热点扫描](#215-停止网络热点扫描)
      * [2.1.6) 获取热点列表](#216-获取热点列表)
      * [2.1.7) 连接网络热点](#217-连接网络热点)
      * [2.1.8) 中断网络连接](#218-中断网络连接)
      * [2.1.9) 获得当前网络详细信息](#219-获得当前网络详细信息)
   + [2.2) 可订阅事件](#22-可订阅事件)
      * [2.2.1) 网络设备发生变化](#221-网络设备发生变化)
      * [2.2.2) 网络热点列表发生变化](#222-网络热点列表发生变化)
      * [2.2.3) 连接到热点](#223-连接到热点)
      * [2.2.4) 连接已配置](#224-连接已配置)
      * [2.2.5) 断开热点](#225-断开热点)
      * [2.2.6) 当前网络信号强度发生变化](#226-当前网络信号强度发生变化)
- [3) 错误代码表](#3-错误代码表)
- [附.1) 商标声明](#附1-商标声明)

[//]:# (END OF TOC)

## 1) 一般信息

- 用途：负责管理系统中的网络接口。
- 应用名称：`cn.fmsoft.hybridos.inetd`。
- 行者：
   + `daemon`: 主行者。

架构图如下：

```
 ------------------------------------------------------------
|        APP1        |        APP2        |       APP3       |
 ------------------------------------------------------------
          |                     |                  |
 == ======================== HBDBus =========================
                                |
 ------------------------------------------------------------
|               cn.fmsoft.hybridos.inetd                     |
 ------------------------------------------------------------
|         wpa_supplicant      |      dhclient                |
 ------------------------------------------------------------
|                  Linux Kernel/Drivers                      |
 ------------------------------------------------------------
```

## 2) 数据总线接口

`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon` 提供的远程过程及可订阅事件，如下所述。

### 2.1) 远程过程

强制规定：远程过程的参数，以及执行结果，使用JSON格式字符串。下面仅对每个过程的参数及返回值，进行说明。

#### 2.1.1) 打开网络设备

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/openDevice`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.2) 关闭网络设备

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/closeDevice`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
```json
    {
        "device":"device_name"
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.3) 查询网络设备状态

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/getDeviceStatus`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`*`
- 参数：
   + `device`：网络设备名称，可使用通配符。
```json
    {
        "device":"*"
    }
```
- 返回值：
   + `data`：返回的数据：
     + `device`：网络设备名；
     + `type`：网络设备类型；
     + `status`：网络设备状态；
     + `mac`：网络设备MAC地址；
     + `inet4`：网络设备IPv4地址信息；
     + `inet6`：网络设备IPv4地址信息；
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "data":[
                    {
                        "device":"eth0",
                        "type":"<wifi|wired|mobile|loop>",
                        "status":"<down|up|running>",
                        "hardwareAddr":"AB:CD:EF:12:34:56",
                        "inet4": {
                            "address": "192.168.1.128",
                            "netmask": "255.255.255.0",
                            "broadcastAddr": "192.168.1.255",   /* No P2P link only */
                            "destinationAddr": "192.168.1.255", /* P2P link only */
                            "gateway": "192.168.1.1",
                            "dns": "192.168.1.1",
                        },
                        "inet6": {
                            "address": ":fe80::583a:5e2d:fa3f:14ad",
                            "netmask": "...",
                            "broadcastAddr": "...", /* No P2P link only */
                            "destinationAddr": "...", /* P2P link only */
                            "gateway": "...",
                            "dns": "...",
                        },
                    },
                    {
                         ......
                    }
               ],
        "errCode": 0,
        "errMsg": "OK"
    }
```

如没有查到网络设备，则`data`为空数组。

#### 2.1.4) 开始扫描网络热点

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiStartScanHotspots`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
   + `interval`：两次扫描的间隔时间（单位：秒）。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `data`：当前热点信息数组，各成员包含如下键值：
     + `bssid`：
     + `ssid`：网络名称；
     + `frequency`：网络频率；
     + `signalStrength`：网络信号强度；
     + `capabilities`：可用的加密方式；
     + `isConnected`：当前是否连接。
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "data": [
                    {
                         "bssid": "f0:b4:29:24:18:eb",
                         "ssid": "fmsoft-dev",
                         "frequency": "2427 MHz",
                         "signalStrength": 65,
                         "capabilities": ["WPA-PSK-CCMP+TKIP","WPA2-PSK-CCMP+TKIP","WPS","ESS"],
                         "isConnected":true
                    },
                    {
                         ......
                    }
                ],
        "errCode":0,
        "errMsg":"OK"
    }
```

如没有网络热点，则`data`为空数组。

该过程将返回扫描结果。热点列表根据信号强度从大到小排列。如有当前连接网络，则改网络排在第一个。


#### 2.1.5) 停止网络热点扫描

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiStopScanHotspots`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
```json
    { 
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

HBDInetd 将停止后台进行的定时热点扫描操作，这将导致停止发送 `WIFIHOTSPOTSCHANGED` 事件泡泡。

#### 2.1.6) 获取热点列表

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiGetHotspotList`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `data`：返回的数据，当前热点信息数组，各成员包含如下键值：
     + `bssid`：
     + `ssid`：网络名称；
     + `frequency`：网络频率；
     + `signalStrength`：网络信号强度；
     + `capabilities`：可用的加密方式；
     + `isConnected`：当前是否连接。
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
   + 样例：

```json
    {
        "data": [
                    {
                         "bssid": "f0:b4:29:24:18:eb",
                         "ssid": "fmsoft-dev",
                         "frequency": "2427 MHz",
                         "signalStrength": 65,
                         "capabilities": ["WPA-PSK-CCMP+TKIP","WPA2-PSK-CCMP+TKIP","WPS","ESS"],
                         "isConnected":true
                    },
                    {
                         ......
                    }
                ],
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.7) 连接网络热点

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiConnect`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
   + `ssid`：网络名称；
   + `password`：网络密码；
   + `autoConnect`：网络中断后是否自动连接；
   + `default`：是否设置为默认网络，下次开机时自动连接。
```json
    {
        "device":"device_name",
        "ssid":"fmsoft-dev",
        "password":"hybridos-hibus",
        "autoConnect":true,
        "default":true
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

在当前版本中，没有实现`autoConnect`、`default`对应的功能。


#### 2.1.8) 中断网络连接

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiDisconnect`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称；
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.9) 获得当前网络详细信息

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.inetd/daemon/wifiGetNetworkInfo`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`*`
- 参数：
   + `device`：网络设备名称；
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `data`：返回的数据：
     + `device`：网络设备名称；
     + `bssid`：BSSID值；
     + `ssid`：网络名称；
     + `encryptionType`：加密方式；
     + `signalStrength`：信号强度；
     + `hardwareAddr`：硬件地址；
     + `inet4`：IPv4 地址信息；
     + `inet6`：IPv6 地址信息；
     + `frenquency`：网络信号频率；
     + `bitRate`：网络速度；
   + `errCode`：返回错误编码，见附表；
   + `errMsg`：错误信息。
```json
    {
        "data":{
                    "device":"device_name",
                    "bssid":"0c:4b:54:a5:ec:93",
                    "ssid":"fmsoft-dev",
                    "encryptionType":"WPA2",
                    "hardwareAddr":"AB:CD:EF:12:34:56",
                    "addressMethod:"dhcp",
                    "inet4": {
                        "address": "192.168.1.128",
                        "netmask": "255.255.255.0",
                        "broadcastAddr": "192.168.1.255",   /* No P2P link only */
                        "destinationAddr": "192.168.1.255", /* P2P link only */
                        "gateway": "192.168.1.1",
                        "DNSServers": "192.168.1.1,8.8.8.8",
                    },
                    "inet6": {
                        "address": ":fe80::583a:5e2d:fa3f:14ad",
                        "netmask": "...",
                        "broadcastAddr": "...", /* No P2P link only */
                        "destinationAddr": "...", /* P2P link only */
                        "gateway": "...",
                        "DNSServers": "...",
                    },
                    "frenquency":"5 GHz",
                    "signalStrength":65,
                    "bitRate": 650000000,
                },
        "errCode":0,
        "errMsg":"OK"
    }
```

如没有查到当前网络详细信息，则`data`为空，`errCode` 返回错误原因。

### 2.2) 可订阅事件

#### 2.2.1) 网络设备发生变化

- 泡泡名称：`NETWORKDEVICECHANGED`
- bubbleData：
   + `device`：网络设备名称；
   + `type`：网络类型；
   + `status`：设备状态。
```json
    {
        "device":"device_name",
        "type":"<wifi|wired|mobile|loopback>",
        "status":"<down|up|running>"
    }
```
- 使用描述：
   + 当网络设备工作状态发生变化时，发送此事件。

#### 2.2.2) 网络热点列表发生变化

- 泡泡名称：`WIFIHOTSPOTSCHANGED`
- 泡泡数据：热点数组，每个成员包含如下信息：
   + `bssid`：BSSID值；
   + `ssid`：网络名称；
   + `capabilities`：加密方式；
   + `signalStrength`：信号强度，取值范围在0——100之间。
- 使用描述：
   + 若 WiFi 设备已被打开，HBDInetd 将定时扫描热点，并通过该事件发送给订阅该事件的行者；
   + 如要获得完整的热点列表，则调用过程 `wifiStartScanHotspots`，通过其返回值获得。
- 样例：

```json
        [
            {
                "bssid": "f0:b4:29:24:18:eb",
                "ssid":"fmsoft-dev",
                "capabilities": ["WPA-PSK-CCMP+TKIP", "WPA2-PSK-CCMP+TKIP", "WPS", "ESS"],
                "signalStrength":65
            },
            {
                ......
            }
        ]
```


#### 2.2.3) 连接到热点

- 泡泡名称：`WIFICONNECTED`
- 泡泡数据：
   + `bssid`：BSSID值；
   + `ssid`：网络SSID；
   + `signalStrength`：网络信号强度，取值范围在0——100之间。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
        "signalStrength":65
    }
```

- 使用描述：
   + 当网络连接成功后发送该泡泡。

#### 2.2.4) 连接已配置

- 泡泡名称：`WIFICONFIGURED`
- 泡泡数据：
   + `bssid`：BSSID值；
   + `ssid`：网络SSID；
   + `signalStrength`：网络信号强度，取值范围在0——100之间。
   + `inet4`：IPv4 地址信息。
   + `inet6`：IPv6 地址信息。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
        "signalStrength":65,
        "inet4": { ... },
        "inet6": { ... },
    }
```

- 使用描述：
   + 当网络连接成功且配置成功后发送该泡泡。

#### 2.2.5) 断开热点

- 泡泡名称：`WIFIDISCONNECTED`
- 泡泡数据：
   + `bssid`：BSSID值；
   + `ssid`：网络SSID；
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
    }
```

- 使用描述：
   + 当连接到指定热点的尝试失败或者热点消失时，产生该泡泡。

#### 2.2.6) 当前网络信号强度发生变化

- 泡泡名称：`WIFISIGNALSTRENGTHCHANGED`
- bubbleData：
   + `bssid`：BSSID值；
   + `ssid`：网络SSID；
   + `signalStrength`：网络信号强度，取值范围在0——100之间。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
        "signalStrength":65
    }
```
- 使用描述：
   + 当网络连接成功后，才开始发送该泡泡；当前网络中断后，不会发送该事件。
   + 该事件的发送间隔，由配置文件中的`scan_time`确定。


## 3) 错误代码表

| 宏定义                        | errCode | errMsg                                   | 备注                     |
| ---------------------------   | ------- | ---------------------------------------- | ------------------------ |
| `ERR_OK`                      | 0       | success                                  | 正常                     |
| `ERR_LIBRARY_OPERATION`       | -1      | an error ocures in library operation     | 错误发生在工具层         |
| `ERR_NONE_DEVICE_LIST`        | -2      | can not get devices list.                | 无法获得网络设备列表     |
| `ERR_WRONG_PROCEDURE`         | -3      | wrong procedure name.                    | 错误的远程调用名称       |
| `ERR_WRONG_JSON`              | -4      | wrong Json format.                       | 错误的JSON格式           |
| `ERR_NO_DEVICE_NAME_IN_PARAM` | -5      | can not find device name in param.       | 设备名不在网络设备列表中 |
| `ERR_NO_DEVICE_IN_SYSTEM`     | -6      | can not find device in system.           | 系统中找不到该设备       |
| `ERR_DEVICE_TYPE`             | -7      | invalid network device type.             | 错误的网络设备类型       |
| `ERR_LOAD_LIBRARY`            | -8      | some error in load library.              | 装载动态库失败           |
| `ERR_NOT_WIFI_DEVICE`         | -9      | device is not WiFi device.               | 该设备并非WiFi设备       |
| `ERR_DEVICE_NOT_OPENNED`      | -10     | device has not openned.                  | 网络设备还未打开         |
| `ERR_OPEN_WIFI_DEVICE`        | -11     | an error ocurs in open wifi device.      | 打开WiFi设备错误         |
| `ERR_CLOSE_WIFI_DEVICE`       | -12     | an error ocurs in close wifi device.     | 关闭WiFi设备错误         |
| `ERR_OPEN_ETHERNET_DEVICE`    | -13     | an error ocurs in open ethernet device.  | 打开Ethernet设备错误     |
| `ERR_CLOSE_ETHERNET_DEVICE`   | -14     | an error ocurs in close ethernet device. | 关闭Ethernet设备错误     |
| `ERR_OPEN_MOBILE_DEVICE`      | -15     | an error ocurs in open mobile device.    | 打开Mobile设备错误       |
| `ERR_CLOSE_MOBILE_DEVICE`     | -16     | an error ocurs in close mobile device.   | 关闭Mobile设备错误       |
| `ERR_DEVICE_NOT_CONNECT`      | -17     | device does not connect any network.     | 网络设备未连接           |


## 附.1) 商标声明

本文提到的产品、技术或者术语名称，涉及北京飞漫软件技术有限公司在中国或其他地区注册的如下商标：

1) 飛漫

![飛漫](https://www.fmsoft.cn/application/files/cache/thumbnails/87f47bb9aeef9d6ecd8e2ffa2f0e2cb6.jpg)

2) FMSoft

![FMSoft](https://www.fmsoft.cn/application/files/cache/thumbnails/44a50f4b2a07e2aef4140a23d33f164e.jpg)

3) 合璧

![合璧](https://www.fmsoft.cn/application/files/4716/1180/1904/256132.jpg)
![合璧](https://www.fmsoft.cn/application/files/cache/thumbnails/9c57dee9df8a6d93de1c6f3abe784229.jpg)
![合壁](https://www.fmsoft.cn/application/files/cache/thumbnails/f59f58830eccd57e931f3cb61c4330ed.jpg)

4) HybridOS

![HybridOS](https://www.fmsoft.cn/application/files/cache/thumbnails/5a85507f3d48cbfd0fad645b4a6622ad.jpg)

5) HybridRun

![HybridRun](https://www.fmsoft.cn/application/files/cache/thumbnails/84934542340ed662ef99963a14cf31c0.jpg)

6) MiniGUI

![MiniGUI](https://www.fmsoft.cn/application/files/cache/thumbnails/54e87b0c49d659be3380e207922fff63.jpg)

7) xGUI

![xGUI](https://www.fmsoft.cn/application/files/cache/thumbnails/7fbcb150d7d0747e702fd2d63f20017e.jpg)

8) miniStudio

![miniStudio](https://www.fmsoft.cn/application/files/cache/thumbnails/82c3be63f19c587c489deb928111bfe2.jpg)

9) HVML

![HVML](https://www.fmsoft.cn/application/files/8116/1931/8777/HVML256132.jpg)

10) 呼噜猫

![呼噜猫](https://www.fmsoft.cn/application/files/8416/1931/8781/256132.jpg)

11) Purring Cat

![Purring Cat](https://www.fmsoft.cn/application/files/2816/1931/9258/PurringCat256132.jpg)

12) PurC

![PurC](https://www.fmsoft.cn/application/files/5716/2813/0470/PurC256132.jpg)

[FMSoft Technologies]: https://www.fmsoft.cn
[HybridOS]: https://hybridos.fmsoft.cn
[MiniGUI]: http:/minigui.fmsoft.cn
[HVML]: https://hvml.fmsoft.cn

