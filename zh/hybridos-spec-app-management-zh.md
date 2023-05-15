# 合璧操作系统的应用管理

Subject: HybridOS App Management  
Version: 2.0  
Author: Vincent Wei  
Category:  Specification  
Creation Date: Nov., 2020  
Last Modified Date: May 12, 2023  
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

- [1) 基本概念及术语](#1-基本概念及术语)
- [2) 应用的安装位置及目录树](#2-应用的安装位置及目录树)
- [3) 应用 Manifest 文件](#3-应用-manifest-文件)
   + [3.1) 窗口和活动的区别](#31-窗口和活动的区别)
- [4) 系统应用](#4-系统应用)
- [5) 启动系统及应用](#5-启动系统及应用)
   + [5.1) 启动指定应用](#51-启动指定应用)
   + [5.2) 启动本应用的指定行者](#52-启动本应用的指定行者)
   + [5.3) 应用代理和窗口实例间的通讯](#53-应用代理和窗口实例间的通讯)
   + [5.4) 终止应用](#54-终止应用)
   + [5.5) 窗口实例的展示](#55-窗口实例的展示)
- [6) 应用组验证](#6-应用组验证)
- [7) 端点权限管理](#7-端点权限管理)
- [附.1) 商标声明](#附1-商标声明)

[//]:# (END OF TOC)

## 1) 基本概念及术语

根据[合璧操作系统设备端数据总线概要设计](hybridos-data-bus-design-zh.md)文档的描述，在合璧操作系统设备端，一个应用（App）可能存在多个并行运行的实体（进程或者线程），每个独立运行的实体称为`行者`（Runner）。

在合璧操作系统中，应用指提供特定功能的程序和数据集合，比如若干个可执行程序，或者 HVML 程序，以及该应用要使用的文件、数据库、图片、字体等资源。应用通常压缩打包成一个文件分发，然后由应用管理器解压到系统中以完成安装。

应用中包含的可执行程序，可连接到 HBDBus，亦可不连接 HBDBus 而独立运行。一个行者可能属于如下类型之一：

* 初始化行者（initializer runner）：指在系统启动时执行的初始化程序，用于初始化网络设备等。
* 精灵行者（daemon runner）：指应用启动时会自动启动并在后台执行的进程或线程，通常用于提供数据查询服务或者外设的控制功能。
* 交互行者（user interface runner）：指负责和最终用户交互的进程或线程，一般情况下是单个或者多个图形用户界面（GUI）进程或线程。在没有屏幕的设备上，可能是语音交互进程或线程。
* 临时行者（occasional runner）：指其他行者临时启动的进程或线程，在完成指定的任务后退出。

应用可分为两种类型：

* 系统应用（sysapp）。这类应用提供系统服务，所以在系统启动后其守护进程会一直保持在运行状态。这类系统应用以 root 身份运行，通常具有最高访问权限。
* 普通应用（app）。指一般的应用，该应用不具有系统最高的访问权限。

和其他常见智能操作系统类似，在 HybridOS 中：

- 每个应用被安装在独立的目录中，且每个应用对应一个唯一的用户账号。通常使用和应用名称一样的用户账号名称（需要将应用名中的 `.` 字符调整为 `-` 字符）。如 `cn.fmsoft.hybridos.hbdbus` 应用，对应的用户为 `cn-fmsoft-hybridos-hbdbus`。
- 每个应用所在目录中包含有一对非对称加密算法使用的私钥。在安装该应用时，对应的公钥应被复制到系统目录（如 `/etc/public-keys/`）中，可用于验证签名。

和其他系统不同的是，在 HybridOS 中，我们引入应用组（app group）的概念：

- 应用组指一组由同一个开发者开发的应用，如果这些应用属于同一个应用组，则这些应用的行者之间可通过 HBDBus 提供过程调用或者事件订阅能力。
- 应用组对应于 Unix 系统的用户组，单个应用可以属于多个应用组。
- 每个应用所属的应用组，在安装时经应用管理器验证并设定。
- 应用管理器通过和云端服务器通讯来完成特定的应用是否可以加入到指定的应用组中。

需要注意的是，应用通常代表的是静态的数据；启动一个应用后，运行在系统中的是该应用的行者或者普通的程序。

应用管理器（app manager）负责应用的安装、卸载以及访问权限的管理；而应用外壳（app shell）负责启动和关闭一个应用，也就是管理运行中的应用之生命周期。

应用外壳可以使用不同的策略来管理一个运行中的应用之生命周期，比如：

- 在小屏幕设备上，一个普通应用的所有交互行者退出后，应用外壳会杀掉该应用启动的其他进程。但在桌面计算机中，应用外壳会保留这些进程。

## 2) 应用的安装位置及目录树

在 HybridOS 中，应用统一安装在 `/app` 目录下，以应用名为目录名，下设若干目录，分别存放可执行程序、动态库以及数据等。

```
cn.fmsoft.hybridos.hbdbus/
├── shared
│   ├── assets
│   └── tmp
├── bin
│   └── hbdbuscl
├── sbin
│   └── hbdbusd
├── hvml
│   └── index.hvml
├── lib
│   └── libhbdbus-plugin.so
├── manifest.json
├── private
│   ├── hmac-cn.fmsoft.hybridos.hbdbus.key
│   └── private-cn.fmsoft.hybridos.hbdbus.pem
├── tmp
└── var
```

上图给出了 HBDBus 应用的目录结构。其中，

- `shared/` 中保存有该应用公开的资源数据，如图标、字体等。
- `bin/` 中保存有该应用的二进制可执行程序。
- `sbin/` 中保存有该应用的系统级二进制可执行程序。
- `hvml/` 中保存该应用的 HVML 程序。
- `lib/` 目录中保存有该应用内部使用的动态库，如引擎、插件等。
- `private/` 目录中保存有仅该应用可访问的私有数据，如非对称加解密使用的私钥。
- `tmp/` 目录中保存有该应用的临时文件。
- `var/` 目录中保存该应用以及该应用所在应用组可访问的数据，如数据库等。

通常，一个应用的名称（也是该应用的全局唯一标识符）具有类似主机域名那样的形式，如 `cn.fmsoft.hybridos.hbdbus`。

## 3) 应用 Manifest 文件

每个应用在其安装根目录中包含有一个 `manifest.json` 文件，其中保存有有关该应用图标、名称、行者入口等的各种信息。如：

```json
{
  "name": "cn.fmsoft.hybridos.settings",
  "group": "cn.fmsoft.hybridos",
  "type": "system",
  "versionCode": 1,
  "versionName": "1.0",
  "minPlatformVersion": 1,
  "label": {
      "en": "Settings",
      "zh_CN": "设置",
      "zh_TW": "設置"
  },
  "description": {
      "en": "The system settings for HybridOS.",
      "zh_CN": "合璧系统设置",
      "zh_TW": "合璧系統設置"
  },
  "icons": {
      "ldpi": {
          "src": "assets/images/appicon72.png",
          "size": "72x72",
      },
      "mdpi": {
          "src": "assets/images/appicon96.png",
          "size": "96x96",
      },
      "hdpi": {
          "src": "assets/images/appicon144.png",
          "size": "144x144",
      },
      "xhdpi":{
          "src": "assets/images/appicon192.png",
          "size": "192x192",
      },
      "xxhdpi":{
          "src":  "assets/images/appicon288.png",
          "size": "288x288",
      },
  },
  "runners": [
    {
      "name": "inetd",
      "type": "exectuable",
      "entry": "bin/inetd",
      "runas": "daemon",
      "visibleProcedures": [
        {
            "methodName": "getNetworkDevices",
            "description": "Get information of all network devices",
        }
      ],
      "visibleEvents": [
        {
            "bubbleName": "INTERNETCHANGED",
            "description": "The internet connection changed",
        }
      ],
    },
    {
      "name": "powerd",
      "type": "exectuable",
      "entry": "sbin/powerd",
      "runas": "daemon, auto",
      "properties": "xxx, yyy, zzz"
      "dependencies": "",
      "visibleEvents": [
        {
            "bubbleName": "LOWPOWER",
            "description": "Low power",
        },
        {
            "bubbleName": "ABOUTTOSHUTDOWN",
            "description": "The device is about to shutdown",
        }
      ],
    },
    {
      "name": "main",
      "type": "hvml",
      "entry": "hvml/index.hvml",
      "runas": "activity, default",
      "properties": "xxx, yyy, zzz",
      "dependencies": ""
    },
    {
      "name": "wifi",
      "type": "hvml",
      "entry": "hvml/wifi/index.hvml",
      "runas": "activity",
      "properties": "xxx, yyy, zzz",
      "dependencies": ""
    },
    {
      "name": "bluetooth",
      "type": "hvml",
      "entry": "hvml/bluetooth/index.hvml",
      "runas": "activity",
      "properties": "xxx, yyy, zzz",
      "dependencies": ""
    },
  ]
}
```

如上所示，在 `manifest.json` 文件中，我们描述了应用的信息，主要包括：

- `name`：全局唯一的应用名称/标识符（字符串，符合合璧应用名称规范）。
- `group`：应用所在的应用组（字符串）。
- `type`：应用类型（字符串），可取 `system`，`normal` 等。设备启动时，系统应用会被自动启动。
- `versionCode`：应用的版本号（正整数）。
- `versionName`：应用的版本名（字符串）。
- `minPlatformVersion`：对合璧平台的最低版本号要求（正整数）。
- `label`：应用标签（字符串或对象；使用对象时，给出了特定 locale 下的应用标签）。
- `description`：应用描述（字符串或对象；使用对象时，给出了特定 locale 下的应用标签）。
- `icons`：应用图标（对象，给出了不同像素密度，如 `ldpi` 或 `hdpi` 下的图标文件及其大小）。
- `runners`：应用的行者，使用对象数据描述，其中，
   * `name`：表示行者名称（字符串，符合合璧应用行者名称规范）。
   * `type`：表示行者的类型，可取如下三者之一：
      - `executable` 或 `exec`，表示可执行程序，包括 Python 等脚本。
      - `hvml`，表示 HVML 程序。
   * `entry`：表示该行者的入口，一般给出相对路径，如 `sbin/inetd` 等。
   * `runas`：表示该行者的运行模式，可取如下值之一：
      - `intializer`：表示该行者是该应用的系统初始化行者，通常在启动系统时执行一次。如果附加有 `critical` 属性，则该行者初始化错误时，将不再执行其后的系统初始化工作。
      - `daemon`：表示该行者以守护进程的形式运行。如果附加有 `auto` 属性，则在启动该应用时启动；如果附加有 `ondemond` 属性，则在需要启动时启动。所有的精灵行者，在应用生命周期结束时，由 HBDShell 杀掉。
      - `program`：表示该行者以普通进程的形式运行。如果附加有 `multiple` 属性，则该行者可启动多个实例。
      - `activity`：表示该交互行者以活动的形式运行。如果附加有 `default` 属性，则该活动为该应用的默认活动。
      - `window`：表示该交互行者以窗口的形式运行。如果附加有 `auto` 属性，则该窗口在启动该应用时自动启动。
   * `properties`：表示该行者的属性列表，可取多个值，以逗号分割。具体属性待定。
   * `dependencies`：表示该行者依赖的其他行者。启动该行者时，需首先启动的其他应用的精灵行者，或执行过其他应用的初始化行者。
   * `visibleProcedures`：空对象（`null`）、空数组（`[]`）或以对象数组形式定义，声明该行者可以提供给所有应用的公共过程。
   * `visibleEvents`：空对象（`null`）、空数组（`[]`）或以对象数组形式定义，声明该行者可以提供给所有应用订阅的公共事件。
   * `windowBoxStyles`：若使用窗口运行交互行者，则该字段定义窗口的盒子样式。

注意，一个应用中不需要由 HBDShell 管理的行者，可不列入。

### 3.1) 窗口和活动的区别

当一个交互行者以活动的形式运行时，该行者所属的应用所有活动将公用一个全屏的窗口实例来渲染所有的活动。当一个交互行者以窗口的形式运行时，每个窗口有一个自己的窗口实例。

当交互行者以窗口形式运行时，可以在 manifest 文件中使用 `windowLayoutStyles` 和 `windowBoxStyles` 设置窗口的布局和盒子样式。窗口的布局样式和盒子样式是 CSS 的子集，如：

```json
{
  "windowLayoutStyles": "display: block;"
  "runners": [
    {
      "name": "wifi",
      "type": "hvml",
      "entry": "hvml/wifi/index.hvml",
      "runas": "window autostart",
      "windowBoxStyles": "postion: relative; width: 100%; height: 50%;"
    },
    {
      "name": "bluetooth",
      "type": "hvml",
      "entry": "hvml/bluetooth/index.hvml",
      "runas": "window autostart"
      "windowBoxStyles": "postion: relative; width: 100%; height: 50%;"
    },
  ]
}
```

HBDShell 会根据当前应用的所有窗口的大小和位置计算新窗口的大小和位置，创建对应的窗口。

## 4) 系统应用

HybridOS 中存在如下系统应用：

- `cn.fmsoft.hybridos.settings`：设置应用；其中包含若干个系统设置活动。
- `cn.fmsoft.hybridos.hbdshell`：应用外壳应用；其中包含若干普通程序，如 `mginit`、`wallpaper`，以及一个行者 `appagent`。
- `cn.fmsoft.hybridos.appmanager`：用来安装、卸载和管理应用权限的应用，包含一个精灵行者和若干活动。
- `cn.fmsoft.hybridos.inetd`：用来管理网络设备的守护进程，包含一个精灵行者或若干活动。
- `cn.fmsoft.hybridos.hbdbus`：数据总线 HBDBus 应用；其中包含两个行者：`daemon`（合璧数据总线服务器）和 `cmdline`（合璧数据总线命令行）。

其他规划中的系统应用：

- `cn.fmsoft.hybridos.sec`：系统安全应用，包含一个精灵行者。
- `cn.fmsoft.hybridos.syslog`：系统日志应用，包含一个精灵行者和一个活动。
- `cn.fmsoft.hybridos.power`：用来监视电源状态的应用，包含一个精灵行者和一个活动。
- `cn.fmsoft.hybridos.http`：HTTP 应用，为外部设备或者本地应用提供 Web 服务。

## 5) 启动系统及应用

合璧操作系统的内核启动之后，会首先启动 HBDShell 应用，HBDShell 应用按如下顺序启动其他系统应用：

1. `cn.fmsoft.hybridos.sec`：系统安全应用定义的精灵行者。
1. `cn.fmsoft.hybridos.syslog`：系统日志应用定义的精灵行者。
1. `cn.fmsoft.hybridos` 应用组中各应用定义的初始化行者及精灵行者：
   + `cn.fmsoft.hybridos.inet`：网络设备管理。
   + `cn.fmsoft.hybridos.power`：电源管理。
   + `cn.fmsoft.hybridos.hbdbus`：数据总线服务器。
   + `cn.fmsoft.hybridos.http`：HTTP 服务器。

以上应用启动无误后，启动 HBDShell 包含的墙纸进程以及 `appagent` 行者等。

HBDShell 的 `appagent` 行者为其他行者提供如下的 HBDBus 过程，用来启动一个应用或者一个活动。

### 5.1) 启动指定应用

任何应用，可向 HBDShell 调用如下过程来启动一个应用。

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.hbdshell/appagent/launchApp`
- 参数：
   + `appName`：字符串，应用名称；
   + `runnerName`：字符串，交互行者的名称，可选。若未定义，则启动默认活动/窗口，或 manifest 文件中定义的第一个交互行者。
   + `intent`：字符串，用于意图，可以是转义后的 JSON 表达。
   + 示例如下：
```json
    {
        "appName": "cn.fmsoft.hybridos.settings",
        "runnerName": "wifi",
        "intent": "...",
    }
```
- 返回值：
   + `errMsg`：错误信息；
   + `errCode`：返回错误编码，0 表示执行成功。可能的错误编码有：
     + `ENOENT`：没有指定的应用或者行者。
     + `EINVAL`：无效参数，尤指意图。
     + `ENOMEM`：内存不足。
```json
    {
        "errCode": 0,
        "errMsg": "OK"
    }
```

HBDShell 的 `appagent` 行者在收到上述请求后，若行者对应一个活动，则：

1. 检查指定应用及活动对应的窗口实例是否已经启动，若启动，则向该窗口所属进程发送请求，同时传递意图参数。活动收到该请求后，切换活动对应的窗口实例到Z序顶部，并根据意图做相应的处理。
1. 如果应用已启动，但没有创建活动对应的窗口实例，则创建一个全屏窗口，并通过命令行传递意图参数。
1. 如果应用未启动，则检查应用是否已安装，指定的活动是否存在。
1. 如果一切就绪，则启动应用的精灵行者，然后启动指定的交互行者，通过命令行传递意图参数。

若指定的行者对应一个窗口，则：

1. 检查指定应用及行者对应的窗口是否已经启动，若启动，则发送请求，同时传递意图参数。窗口收到该请求后，切换行者对应的窗口实例到Z序顶部，并根据意图做相应的处理。
1. 如果应用已启动，但没有创建行者对应的窗口实例，则根据窗口布局样式，确定新窗口的大小和位置，创建一个窗口，并通过命令行发送意图参数。
1. 如果应用未启动，则检查应用是否已安装，指定的行者是否存在。
1. 如果一切就绪，则启动应用的精灵行者，然后启动指定的交互行者，并通过命令行传递意图参数。

若 `runnerName` 为空字符串，则：

1. 确保已启动该应用的所有精灵行者。
1. 确保已启动该应用的默认活动。
1. 确保已启动该应用的所有自动启动窗口。
1. 传递意图参数。

### 5.2) 启动本应用的指定行者

任何应用，可调用 HBDShell 的如下过程来启动一个该应用的某个行者。

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.hbdshell/appagent/launchRunner`
- 参数：
   + `runnerName`：字符串，应用名称；
   + `intent`：字符串，用于意图，可以是转义后的 JSON 表达。
   + 示例如下：
```json
    {
        "runnerName": "wifi",
        "intent": "...",
    }
```
- 返回值：
   + `errMsg`：错误信息；
   + `errCode`：返回错误编码，0 表示执行成功。可能的错误编码有：
     + `ENOENT`：没有指定或者行者。
     + `EINVAL`：无效参数，尤指意图。
     + `ENOMEM`：内存不足。
```json
    {
        "errCode": 0,
        "errMsg": "OK"
    }
```

HBDShell 的 `appagent` 行者在收到上述请求后，若行者对应一个活动，则：

1. 检查指定的窗口实例是否已经启动，若启动，则向该窗口所属进程发送请求，同时传递意图参数。窗口收到该请求后，切换活动对应的窗口实例到Z序顶部，并根据意图做相应的处理。
1. 如果对应应用尚未创建活动对应的窗口实例，则创建一个全屏窗口，并通过命令行发送意图参数。
1. 异常情形：收到该请求时，应用可能已经终止，即没有任何该应用的实例。

若行者对应一个窗口，则：

1. 检查行者对应的窗口是否已经启动，若启动，则发送请求，同时传递意图参数。窗口收到该请求后，切换行者对应的窗口实例到Z序顶部，并根据意图做相应的处理。
1. 如果尚未创建行者对应的窗口实例，则根据窗口布局样式，确定新窗口的大小和位置，创建一个窗口，并通过命令行发送意图参数。
1. 异常情形：收到该请求时，应用可能已经终止，即没有任何该应用的实例。

### 5.3) 应用代理和窗口实例间的通讯

所有应用的窗口实例需要连接到 HBDShell `appagent` 创建的 Unix Domain Socket 上，并通过该套接字发送请求或者接受请求。注意，为避免冲突，降低复杂性，对此通讯，我们不使用 HBDBus。

当窗口实例重新装载了一个新的行者或者重新装载了当前的行者后，应发送 `runnerReloaded` 请求给 hiSHell 的 `appagent`。传递给 `appagent` 的参数需要包括（以 JSON 描述）：

```json
{
    "windowHandle": "<the_unique_handle_of_the_window>",
    "appName": "<the_current_app_name>",
    "runnerName": "<the_current_runner_name>",
}
```

当 `appagent` 要求一个窗口实例装载一个新的行者或者重新初始化当前行者时，应发送 `reloadRunner` 请求给 HBDShell 的 `appagent`。传递给 `appagent` 的参数需要包括（以 JSON 描述）：

```json
{
    "fromApp": "<the_app_name_emit_this_request>",
    "fromRunner": "<the_runner_name_emit_this_request>",
    "runnerName": "<the_target_runner_name>",
    "intent": "...",
}
```

当用户关闭一个窗口实例时，应发送 `runnerClosed` 请求给 HBDShell 的 `appagent`。传递给 `appagent` 的参数需要包括（以 JSON 描述）：

```json
{
    "windowHandle": "<the_unique_handle_of_the_window>",
    "appName": "<the_current_app_name>",
    "runnerName": "<the_current_runner_name>",
}
```

### 5.4) 终止应用

当一个普通（非系统）应用的所有窗口实例被关闭后，HBDShell 的 `appagent` 将杀掉该应用的所有其他进程，包括精灵行者，正在运行的其他普通进程等。

### 5.5) 窗口实例的展示

以活动形式运行的窗口，切换到前台时，不需要做特殊处理，将对应窗口至于顶部即可。

以窗口形式运行的交互行者，当切换到前台时，应做特殊处理，从该应用所有窗口的 Z 序底部开始，依次调用底层窗口系统（MiniGUI）的切换到顶部函数（`SetActiveMainWindow`）函数。

## 6) 应用组验证

应用组的验证，需要连入中心化的验证服务器如 `wss://hybridos.fmsoft.cn` 进行，通过非对称加密的签名验证执行。

应用开发者通过在网站上注册应用组，获得分配的私钥和公钥。在安装应用时，由应用管理器连接到验证服务器执行相应的身份验证操作。

（细节待续）

## 7) 端点权限管理

如 HBDBus 协议所描述，一个应用是否可以调用某个特定的过程或者订阅某个特定的事件，可以通过 `forHost` 和 `forApp` 两个参数来指定。

可使用如下的语法来使用通配符（`*` 和 `?`）指定多个主机，亦可用逗号（`,`）指定多个匹配项，称为模式（pattern）列表，定义了特定方法和事件的访问控制列表（ACL）。如：

```
    cn.fmsoft.hybridos.*, cn.fmsoft.app.*, com.example.a??
```

另外，HBDBus 还支持如下特定的匹配模式指定方法：

1. 使用 `$` 前缀时，视作变量，将使用上下文信息替代相应的变量。目前支持两个变量：
   - `$self`：表示该过程或事件的创建者所在主机名。
   - `$owner`：表示创建该过程或者事件的应用名。
   - `$granted`：表示通过询问用户获得授权的访问列表清单（新增）。
1. 使用 `!` 前缀时，表示排除给定的匹配模式。

如下面的访问控制列表，用于匹配创建当前方法或者事件的应用，以及 HBDBus 应用本身：

```
    $owner, cn.fmsoft.hybridos.hbdbus
```

如下模式列表，用于匹配所有不以 `cn.fmsoft.hybridos.` 打头的应用：

```
    !cn.fmsoft.hybridos.*, *
```

注意，使用通配符时，隐含了对特定应用组的访问权限的定义。

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

