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
      * [2.1.10) 终止](#2110-终止)
   + [2.2) 可订阅事件](#22-可订阅事件)
      * [2.2.1) 网络设备发生变化](#221-网络设备发生变化)
      * [2.2.2) 网络设备已配置](#222-网络设备已配置)
      * [2.2.3) 网络设备配置失败](#223-网络设备配置失败)
      * [2.2.4) 发现热点](#224-发现热点)
      * [2.2.5) 丢失热点](#225-丢失热点)
      * [2.2.6) 热点扫描结束](#226-热点扫描结束)
      * [2.2.7) 热点已连接](#227-热点已连接)
      * [2.2.8) 失败的连接尝试](#228-失败的连接尝试)
      * [2.2.9) 热点已断开](#229-热点已断开)
      * [2.2.10) 无线信号强度发生变化](#2210-无线信号强度发生变化)
- [3) 错误代码表](#3-错误代码表)
- [4) 示例](#4-示例)
   + [附.1) 修订记录](#附1-修订记录)
      * [RC1) 230531](#rc1-230531)
- [附.1) 商标声明](#附1-商标声明)

[//]:# (END OF TOC)

## 1) 一般信息

- 用途：负责管理系统中的网络接口。
- 应用名称：`cn.fmsoft.hybridos.inetd`。
- 行者：
   + `main`: 主行者。
   + `dhclient`: DHCP 客户端行者。执行 DHCP 请求并根据 DHCP 信息设置网络路由和 DNS 服务器等。

架构图如下：

```
 ------------------------------------------------------------
|        APP1        |        APP2        |       APP3       |
 ------------------------------------------------------------
          |                     |                  |
 =========================== HBDBus =========================
                                |
 ------------------------------------------------------------
|          cn.fmsoft.hybridos.inetd (main, dhclient)         |
 ------------------------------------------------------------
|                      wpa_supplicant                        |
 ------------------------------------------------------------
|                  Linux Kernel/Drivers                      |
 ------------------------------------------------------------
```

## 2) 数据总线接口

`edpt://localhost/cn.fmsoft.hybridos.inetd/main` 提供的远程过程及可订阅事件，如下所述。

### 2.1) 远程过程

强制规定：远程过程的参数，以及执行结果，使用JSON格式字符串。下面仅对每个过程的参数及返回值，进行说明。

#### 2.1.1) 打开网络设备

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/openDevice`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.2) 关闭网络设备

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/closeDevice`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name"
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.3) 查询网络设备状态

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/getDeviceStatus`
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
     + `device`：网络设备名。
     + `type`：网络设备类型。
     + `status`：网络设备状态。
     + `mac`：网络设备MAC地址。
     + `inet4`：网络设备IPv4地址信息。
     + `inet6`：网络设备IPv4地址信息。
   + `errCode`：返回错误编码，见附表。
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

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiStartScan`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
   + `waitSeconds`：启动扫描后返回结果前的等待时间（单位：秒）；若小于 0.1，则立即返回当前已有的扫描结果。
```json
    {
        "device":"device_name",
        "waitSeconds":0.5
    }
```
- 返回值：
   + `data`：当前热点信息数组，各成员包含如下键值：
     + `bssid`：
     + `ssid`：网络名称。
     + `frequency`：网络频率。
     + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
     + `capabilities`：热点能力。
     + `isSaved`：是否是已保存热点。
     + `isConnected`：当前是否连接。
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "data": [
                    {
                         "bssid": "f0:b4:29:24:18:eb",
                         "ssid": "fmsoft-dev",
                         "frequency": "2.4 GHz",
                         "signalLevel": -65,
                         "capabilities": "WPA-PSK-CCMP",
                         "isSaved":true,
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

- 备注：
   + 如没有网络热点，则 `data` 为空数组。若要获取完整的热点扫描结果，应订阅 `WiFiScanFinished` 事件。

#### 2.1.5) 停止网络热点扫描

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiStopScan`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

HBDInetd 将停止后台进行的定时热点扫描操作，这将导致停止发送 `WiFiScanFinished` 事件泡泡。

#### 2.1.6) 获取热点列表

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiGetHotspotList`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `data`：返回的数据，当前热点信息数组，各成员包含如下键值：
     + `bssid`：
     + `ssid`：网络名称。
     + `frequency`：网络频率。
     + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
     + `capabilities`：热点能力。
     + `isSaved`：是否是已保存的热点。
     + `isConnected`：当前是否连接。
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
   + 样例：

```json
    {
        "data": [
                    {
                         "bssid": "f0:b4:29:24:18:eb",
                         "ssid": "fmsoft-dev",
                         "frequency": "2.4 GHz",
                         "signalLevel": -65,
                         "capabilities": "WPA-PSK-CCMP",
                         "isSaved":true
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

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiConnect`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
   + `ssid`：热点名称。
   + `bssid`：热点地址；取 `null` 表示未知，此时需指定 `keymgmt`，否则表示该热点来自扫描结果，此时可忽略 `keymgmt` 参数。
   + `keymgmt`：安全性；取 `NONE`、`WEP`、`WPA-EAP`（暂不支持）、`WPA-PSK` 和 `WPA2-PSK` 之一。
   + `passphrase`：`keymgmt` 不为 `NONE` 时，通过此参数指定密语（8 ~ 63 ASCII 字符）。
```json
    {
        "device":"device_name",
        "ssid":"fmsoft-dev",
        "bssid":null,
        "keymgmt":"WPA-PSK",
        "passphrase":"xxxxxxxx"
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

若超时或返回 `ERR_UNCERTAIN_RESULT`，可根据随后的 `WiFiConnected` 或者 `WiFiFailedConnAttempt` 事件确定连接是否成功。

#### 2.1.8) 中断网络连接

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiDisconnect`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

#### 2.1.9) 获得当前网络详细信息

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/wifiGetNetworkInfo`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`*`
- 参数：
   + `device`：网络设备名称。
```json
    {
        "device":"device_name",
    }
```
- 返回值：
   + `data`：返回的数据：
     + `device`：网络设备名称。
     + `bssid`：BSSID值。
     + `ssid`：网络名称。
     + `keyMgmt`：加密方式。
     + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
     + `hardwareAddr`：硬件地址。
     + `inet4`：IPv4 地址信息。
     + `inet6`：IPv6 地址信息。
     + `frenquency`：网络信号频率。
     + `bitRate`：网络速度。
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。
```json
    {
        "data":{
                    "device":"device_name",
                    "bssid":"0c:4b:54:a5:ec:93",
                    "ssid":"fmsoft-dev",
                    "keyMgmt":"WPA2",
                    "hardwareAddr":"AB:CD:EF:12:34:56",
                    "configMethod:"dhcp",
                    "DNSServers": "192.168.1.1,8.8.8.8",
                    "inet4": {
                        "address": "192.168.1.128",
                        "netmask": "255.255.255.0",
                        "broadcastAddr": "192.168.1.255",   /* No P2P link only */
                        "destinationAddr": "192.168.1.255", /* P2P link only */
                        "gateway": "192.168.1.1",
                    },
                    "inet6": {
                        "address": ":fe80::583a:5e2d:fa3f:14ad",
                        "netmask": "...",
                        "broadcastAddr": "...", /* No P2P link only */
                        "destinationAddr": "...", /* P2P link only */
                        "gateway": "...",
                    },
                    "frenquency":"5 GHz",
                    "signalLevel":-65,
                    "bitRate": 650000000,
                },
        "errCode":0,
        "errMsg":"OK"
    }
```

如没有查到当前网络详细信息，则`data`为空，`errCode` 返回错误原因。

#### 2.1.10) 终止

- Procedure URI：`edpt://localhost/cn.fmsoft.hybridos.inetd/main/method/terminate`
- 权限：
   + 允许的主机：`localhost`
   + 允许的应用：`cn.fmsoft.hybridos.*`
- 参数：
   + `afterSeconds`：一个大于零的数值，指定秒数。HBDInetd 将在指定的秒数后终止。零表示立即终止。
- 返回值：
   + `errCode`：返回错误编码，见附表。
   + `errMsg`：错误信息。

下面是一个示例调用参数：

```json
    {
        "afterSeconds": 3,
    }
```

对应的返回值：

```json
    {
        "errCode":0,
        "errMsg":"OK"
    }
```

如没有查到当前网络详细信息，则`data`为空，`errCode` 返回错误原因。

### 2.2) 可订阅事件

#### 2.2.1) 网络设备发生变化

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/DeviceChanged`
- bubbleData：
   + `device`：网络设备名称。
   + `type`：网络类型。
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

#### 2.2.2) 网络设备已配置

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/DeviceConfigured`
- 泡泡数据：
   + `device`：设备接口名称。
   + `method`：配置方法。
   + `inet4`：IPv4 地址信息。
   + `inet6`：IPv6 地址信息。
```json
    {
        "device":"wlan0",
        "method":"dhcp",
        "inet4": { ... },
        "inet6": { ... },
    }
```

- 使用描述：
   + 当网络设备的配置成功后发送该泡泡。

#### 2.2.3) 网络设备配置失败

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/DeviceConfigFailed`
- 泡泡数据：
   + `device`：设备接口名称。
   + `method`：配置方法。
   + `reason`：失败原因。
```json
    {
        "device":"wlan0",
        "method":"dhcp",
        "reason":"timeout"
    }
```

- 使用描述：
   + 当网络设备的配置失败时发送该泡泡。

#### 2.2.4) 发现热点

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiHotspotFound`
- 泡泡数据：
    + `bssid`：BSSID。
    + `ssid`：热点名称。
    + `capabilities`：热点能力。
    + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
    + `isSaved`：是否是已保存的热点。
    + `isConnected`：是否已连接到该热点。
- 使用描述：
   + 若 WiFi 设备已被打开，HBDInetd 将定时扫描热点，当发现新的热点时通过该事件发送给订阅该事件的行者。
   + 如要获得完整的当前热点列表，则调用过程 `wifiGetHotspotList`，通过其返回值获得。
- 样例：

```json
{
    "bssid": "f0:b4:29:24:18:eb",
    "ssid":"fmsoft-dev",
    "capabilities": "WPA-PSK-CCMP",
    "signalLevel":-65,
    "isSaved":true,
    "isConnected":true
}
```

#### 2.2.5) 丢失热点

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiHotspotLost`
- 泡泡数据：
    + `bssid`：BSSID。
- 使用描述：
   + 若 WiFi 设备已被打开，HBDInetd 将定时扫描热点，当发现之前扫描获得的热点丢失时，通过该事件发送给订阅该事件的行者。
   + 如要获得完整的当前热点列表，则调用过程 `wifiGetHotspotList`，通过其返回值获得。
- 样例：

```json
{
    "bssid": "f0:b4:29:24:18:eb",
}
```

#### 2.2.6) 热点扫描结束

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiScanFinished`
- 泡泡数据：
   + `success`：表示扫描成功与否。
   + `hotspots`：若扫描失败，该键值为 `null`。若扫描成功，则包含该键值用于描述热点数组，每个成员包含如下信息：
      + `bssid`：BSSID。
      + `ssid`：热点名称。
      + `capabilities`：热点能力。
      + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
      + `isSaved`：是否是已保存的热点。
      + `isConnected`：当前是否连接。
- 使用描述：
   + 若 WiFi 设备已被打开，HBDInetd 将定时扫描热点，在扫描结束后通过该事件发送给订阅该事件的行者。
   + 如要获得当前的热点列表，则调用过程 `wifiGetHotspotList`，通过其返回值获得。
- 样例：

```json
{
    "success": true,
    "hotspots":[
        {
            "bssid": "f0:b4:29:24:18:eb",
            "ssid":"fmsoft-dev",
            "capabilities": "WPA-PSK-CCMP",
            "signalLevel":-65,
            "isSaved":true,
            "isConnected":true
        },
        {
            ......
        }
    ]
}
```

#### 2.2.7) 热点已连接

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiConnected`
- 泡泡数据：
   + `bssid`：BSSID值。
   + `ssid`：网络SSID。
   + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
        "signalLevel":-65
    }
```

- 使用描述：
   + 当网络连接成功后发送该泡泡。

#### 2.2.8) 失败的连接尝试

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiFailedConnAttempt`
- 泡泡数据：
   + `ssid`：网络SSID。
   + `reason`：失败原因。
```json
    {
        "ssid":"fmsoft-dev",
        "reason":"WRONG-KEY",
    }
```

- 使用描述：
   + 当网络连接尝试失败后发送该泡泡。

#### 2.2.9) 热点已断开

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiDisconnected`
- 泡泡数据：
   + `bssid`：BSSID值。
   + `ssid`：网络SSID。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
    }
```

- 使用描述：
   + 当指定热点的连接断开（如热点消失）时，产生该泡泡。

#### 2.2.10) 无线信号强度发生变化

- Event URI: `edpt://localhost/cn.fmsoft.hybridos.hbdinetd/main/bubble/WiFiSignalLevelChanged`
- bubbleData：
   + `bssid`：BSSID值。
   + `ssid`：网络SSID。
   + `signalLevel`：网络信号强度（dBm，分贝毫）；一般的取值范围：-100 ~ 0，取值越高表示信号越好。
```json
    {
        "bssid":"f0:b4:29:24:18:eb",
        "ssid":"fmsoft-dev",
        "signalLevel":-65
    }
```
- 使用描述：
   + 当网络连接成功后，才开始发送该泡泡。当前网络中断后，不会发送该事件。


## 3) 错误代码表

| 宏定义                        | errCode | errMsg                                   | 备注                         |
| ---------------------------   | ------- | ---------------------------------------- | ---------------------------- |
| `ERR_OK`                      | 0       | Success                                  | 正常                         |
| `ERR_DATA_BUS`                | -1      | Error in data bus                        | 错误发生在数据总线           |
| `ERR_DEVICE_CONTROLLER`       | -2      | Error in device controller.              | 错误发生在设备控制器         |
| `ERR_TWO_MANY_FAILURES`       | -3      | Too many failures.                       | 太多错误                     |
| `ERR_UNCERTAIN_RESULT`        | -4      | Uncertain result; see event.             | 不确定的结果；见相应事件     |
| `ERR_WPA_INVALID_SSID`        | -5      | Invalid SSID.                            | 无效的 SSID（热点名称）      |
| `ERR_WPA_INVALID_PASSPHRASE`  | -6      | Invalid passphrase.                      | 无效密语                     |
| `ERR_WPA_INVALID_KEYMGMT`     | -7      | Invalid key management.                  | 无效的密钥管理方法           |
| `ERR_WPA_WRONG_PASSPHRASE`    | -8      | Wrong passphrase.                        | 错误的密语                   |
| `ERR_WPA_TIMEOUT`             | -9      | Timeout.                                 | 操作超时                     |
| `ERR_UNRESOLVED_ATTEMPT`      | -10     | There already is an unresolved attempt.  | 尚有未决意图                 |

## 4) 示例

```
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main getDeviceStatus {device: '*'}
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main openDevice {device:'wlp0s20f3'}
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main closeDevice {device:'wlp0s20f3'}

# call edpt://localhost/cn.fmsoft.hybridos.inetd/main wifiStartScan { device:'wlp0s20f3' }
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main wifiStopScan { device:'wlp0s20f3' }
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main wifiConnect { device:'wlp0s20f3', ssid:'foo', bssid:null, keymgmt:'NONE', passphrase: 'barbarbar'}
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main wifiDisconnect { device:'wlp0s20f3' }
# call edpt://localhost/cn.fmsoft.hybridos.inetd/main wifiGetNetworkInfo { device:'wlp0s20f3' }
```

### 附.1) 修订记录

发布历史：

- 2023 年 05 月 31 日：发布 V2.0 RC1，标记为 'v1.0-rc1-230531'。

#### RC1) 230531

1. 调整泡泡名称：使用首字母大写的驼峰命名法。
1. 新增过程及泡泡。
1. 调整过程参数和返回值。

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

