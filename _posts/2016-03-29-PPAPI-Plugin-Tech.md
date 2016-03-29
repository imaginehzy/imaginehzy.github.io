---
layout: post
title: "Chrome浏览器PPAPI插件技术调研"
description: ""
category: articles
tags: [开发]
---

NPAPI Plugin的版本限制
----------------------
Google Chrome 42开始，默认禁止NPAPI Plugin, 但是可以在Chrome://flags 和chrome://plugins 页面打开NPAPI的支持。

Chrome 45+开始，完全不支持NPAPI插件。

其他厂商中Mozilla Firefox也宣称要放弃NPAPI。Safari在windows平台上，在5.1.7后就没有更新，是继续支持NPAPI的



NaCl和 PNaCl是什么
-------------------
Native Client（NaCl)和Portable Native Client (PNaCl)是两套独立的开发SDK。

Native Client 是一个沙盒技术。是目前google主推的使用native code的开发扩展的方式。有了NaCl，可以在Chrome里安全运行Intel x86或ARM原生代码的沙盒技术，方便开发者使用C和C++代码在浏览器里实现高效的应用。

使用NaCl开发的模块是nexe文件，这样的文件是和操作系统无关的，但是还不能做到和硬件架构无关。NaCl开发的nexe只能通过google的web store进行发布，因为web store会帮你识别用户的硬件架构，把合适的硬件相关的nexe安装到用户机器上。（和CPU相关）

为了解决硬件架构兼容性的问题，又出现了PNacl。PNacl的工具链把代码编译为中间码解决了这个问题。chrome加载一个pexe的时候会把pexe翻译为NaCl的代码然后再执行。PNaCl开发的模块得到的文件是pexe。pexe不需要通过Google Web Store来发布。

NaCl要编译成动态链接库，PNaCl只能编译成静态库



PPAPI是什么
-----------
全称为Pepper Plugin API。分为trusted和unstrusted两种。trusted只能通过命令行调用，是在浏览器进程外调用插件，可以调用系统api。unstrusted的插件是运行在NaCl沙盒里面的，所以需要提权的操作都不能执行，包括调用系统api等等。沙盒技术分为NaCl和PNaCl, 所以PPAPI插件分别有三种embed类型实现:

+ “application/x-ppapi”，平台相关，唯一能直接使用win32 api的platfrom（有功能上的限制）。dll，格式不允许通过Chrome web store分发。Flash就是采用第一种类型开发的并且内置在浏览器里面。但是必须用命令行的方式调用，详细见下一节。
+ “application/x-nacl”， 只能通过PPAPI，平台无关，cpu相关。nexe格式，只能通过Chrome web store分发。
+ “application/x-pnacl”，只能通过PPAPI，平台无关，cpu无关。pexe格式，可以不通过Chrome web store分发。  

本质上，PPAPI只是一个提供给c/c++的接口，我们实现这些要求用到的接口里面的功能。



限制
----
相对于Win32来说, NaCl相当于另一个平台(虚拟机).一些操作系统相关的API需要移植. 除此之外, 参考Technical Overview, 还有一些其它的限制:

+ 不支持硬件异常
+ 不支持创建进程
+ 不支持传统的TCP/UDP sockets (有其它方式去实现, RakNet已经有个预览版本)
+ 不支持查询可用内存
+ 内联汇编必须兼容 Native Client 验证器(使用SDK中的 ncval 工具检查) (一些使用汇编优化的代码(如数学库)可能不能使用)
+ Pepper API 必须从主线程调用 (对于引擎的多线程架构有所冲击: IO, OpenGL, Input...)
+ 必须同时支持32位和64位 (很多游戏引擎没有考虑64位, 需要解决一些兼容问题)
+ C runtime只支持GLIBC和newlib (一些操作系统相关的API必须改成标准库实现)
+ 渲染使用OpenGL ES (需要做一个渲染器)
+ Pepper Thread 中不能进行阻塞性的操作 (好多操作都要考虑重新实现, 或者转移到线程)
+ 文件只能访问HTTP服务器或者本地Cache中的 (限制了资源加载来源)

这些限制都是为了保证安全性(想想ActiveX为什么失败了)和跨平台(Win/Linux/OSX使用同一个版本)


PPAPI发布
----------
PPAPI开发的dll可以通过命令chrome --register-pepper-plugins="G:\\example.dll#ppexample##1.0.0;a
pplication/x-ppapi-example" file:///G:/web/index.html 打开chrome执行测试，但是直接打开chrome则插件不可用。这是可信用PPAPI插件的调用方法。

如果要绕过Chrome App Store，只能用前面说的PPAPI+PNaCl的方法。



其他替代方法：Native Messaging
------------------------------
Chrome Extension还支持Native Messaging。我们可以实现一个exe可执行程序，然后在chrome扩展里面配置exe的路径。Chrome会启动一个进程执行这个与exe，然后通过扩展用Native Messaging的方式通讯与exe通讯，让exe执行需要执行的Windows API操作。不过，安装Chrome Extension还是必须通过Chrome App Store，本地的扩展只能通过设置调试模式执行。



参考：
+ https://www.chromium.org/nativeclient/getting-started/getting-started-background-and-basics
+ NaCl主页 https://developer.chrome.com/native-client
+ SO上面的热帖 http://stackoverflow.com/tags/ppapi/hot
+ 一篇中文开发总结 http://blog.csdn.net/u012377333/article/details/50679819
+ 一篇NaCl和PNaCl总结 http://sssa2000.github.io/blog/2015/05/20/chrome-plugin-dev-1/
+ Extension里面的Native Message https://developer.chrome.com/extensions/nativeMessaging#native-messaging-host
