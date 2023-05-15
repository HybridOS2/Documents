# 合璧操作系统外壳应用设计

Subject: HybridOS Shell  
Version: 2.0  
Author: Vincent Wei  
Category: Design  
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
   + [2.1) 远程过程调用](#21-远程过程调用)
- [附.1) 商标声明](#附1-商标声明)

[//]:# (END OF TOC)

## 1) 一般信息

- 用途：负责启动系统中各个应用并管理其生命周期。
- 别名：HBDShell。
- 应用名称：`cn.fmsoft.hybridos.shell`。
- 行者:
   + `main`: 主行者。
   + `appagent`: 负责启动应用。
   + `appinstaller`: 应用安装器。

HBDShell 为其他应用的运行提供了基本的运行环境，并对其他应用的生命周期进行管理。除了少数系统服务外，所有应用程序的启动、运行、关闭，都需要使用 `HBDShell` 提供的数据总线接口来完成。

合璧系统启动流程：

1. Linux 内核启动，装载各个模块，并启动 `init` 进程。
1. init 进程启动各项系统服务，最后启动 HBDShell。
1. HBDShell 启动已安装应用登记的守护进程。

## 2) 数据总线接口

### 2.1) 远程过程调用

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.shell/appagent/launchApp`
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

- 过程名称：`edpt://localhost/cn.fmsoft.hybridos.shell/appagent/launchRunner`
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


