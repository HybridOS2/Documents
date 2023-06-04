# HybridOS

This repository contains specifications, articles, guides, and documents about HybridOS.

HybridOS is an open source operating system designed for embedded systems and AIoT devices which are based on Linux kernel.

## Goals of HybridOS

[HVML], Hybrid Virtual Markup Language, is a descriptive programming language proposed and designed by [Vincent Wei] during the developement of HybridOS.

HVML provides a data-driven programming model, and one developer can easily write an app with GUI by using HVML like writing a HTML document.
HVML also provides some easy ways to integrate external modules writing in C/C++ or Python programming languages.
Moreover, HVML provides a new and universal app framework for device apps, client apps, and even for cloud servers.

By using HVML, one developer can not only write device apps for HybridOS,
   but also client apps for Linux, Windows, macOS, Android, and iOS operating systems.
Finally, the app programming languages of HybridOS will be reduced to two: HVML and Python,
    no matter on device, mobile phone, computer, or server.
As a result, the development cost of an embedded/AIoT device will be reduced greatly by using HybridOS.

In addition to HVML, HybridOS provides a new software stack for embedded system and AIoT devices.
We design a data bus service (HBDBus) for system modules and apps.
An app can easily interact with various system modules and other apps through HBDBus.
HBDBus exchanges data among apps and the services in JSON, which is friendly for any programming language or other devices on the network.

HybridOS is based on the Linux kernel, making full use of the Linux kernel ecosystem,
         reducing the difficulty of developing various drivers and the difficulty of supporting various protocol stacks, thus reducing development cost.

### Design documents and specifications

## Developer Guides

- [Third-party Software List](ThirdPartySoftwareList.md)

## API References


## Specifications and Design Documents

__NOTE__:  
Most design documents are written in Chinese.

- [合璧操作系统应用管理规范](zh/hybridos-spec-app-management-zh.md)
- [数据总线设计](zh/hybridos-design-data-bus-zh.md)
- [网络接口管理设计](zh/hybridos-design-sysapp-inetd-zh.md)
- [应用外壳设计](zh/hybridos-design-sysapp-shell-zh.md)

## Software Repositories

We use some popular open source software as the common infrastructure of HybridOS.
However, we have done a lot of adjustments and optimizations for some of the key software.
We call the changed versions of the software as derivatives for HybridOS and maintain them through the following public repositories:

- Tailored WebKit:
   + WebKit2HBD: the HybridOS derivative of [WebKit] \(only tarball):
<https://github.com/HybridOS2/WebKit2HBD>

- Graphics Stack:
   + MesaHBD: the HybridOS derivative of [Mesa]:
<https://github.com/HybridOS2/MesaHBD>
   + CairoHBD: the HybridOS derivative of [Cairo]:
<https://github.com/HybridOS2/CairoHBD>
   + HBDDRMDrivers: DRM Drivers for HybridOS:
<https://github.com/HybridOS2/HBDDRMDrivers>
   + MiniGUI: the window system for HybridOS:
<https://github.com/VincentWei/minigui>

- System Servers:
   + HBDBus: the data bus server for HybridOS:
<https://github.com/HybridOS2/HBDBus>

- System Daemons:
   + HBDInetd: the network device managment service:
<https://github.com/HybridOS2/HBDInetd>
   + HBDShell: the app management service:
<https://github.com/HybridOS2/HBDShell>

## Current Status

- June. 2023:  
   FMSoft announces the availability of HybridOS 2.0.0.
- Aug. 2020:  
   FMSoft announces the availability of the key component of HybridOS: WebKit2HBD.
- Mar. 2020:  
   FMSoft announces the availability of MiniGUI 5.0 and the updated graphics stack of HybridOS for device-side.
- Nov. 2019:  
   Initial release of CairoHBD.
- Nov. 2018:  
   Initiate this project and organize specifications and design documents.

## Copying

Copyright (C) 2018 \~ 2023 [Beijing FMSoft Technologies Co., Ltd.]

- For device side and client side, HybridOS uses GPLv3 or LGPLv3.
- For server side, HybridOS uses AGPLv3.
- For documents, GPLv3 applies.

If a component of HybridOS is a derivative of an existing open source software,
   we generally continue to use the original license.

Also note that HybridOS integrates many mature open source software, such as ZLib, LibPNG, LibJPEG, SQLite, FreeType, HarfBuzz, and so on.
For the copyright owners and licenses for these software, please refer to the README or LICENSE files contained in the source tarballs.

---

[HybridOS Architecture]: en/HybridOS-Architecture.md
[HybridOS Code and Development Convention]: en/HybridOS-Code-and-Development-Convention.md

[HVML]: https://hvml.fmsoft.cn
[Beijing FMSoft Technologies Co., Ltd.]: https://www.fmsoft.cn
[FMSoft Technologies]: https://www.fmsoft.cn
[HybridOS Official Site]: https://hybridos.fmsoft.cn

[MiniGUI]: http:/www.minigui.com
[WebKit]: https://webkit.org
[Mesa]: https://mesa3d.org/
[Cairo]: https://www.cairographics.org/

