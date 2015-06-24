===============
进阶指南
===============

---------------
配置开发环境
---------------

官网文档使用已配置好的VirtualBox虚拟机作为开发环境，但此处建议使用由安信可定制的Eclipse CDT来作为日常开发环境，不需要下载配置虚拟机，简单方便，且带有cygwin环境，可做更多的自定义调整。下载地址在这里 http://www.ai-thinker.com/forum.php?mod=viewthread&tid=650 感谢开发者的努力。


^^^^^^^^^^^^^^^^^
使用Eclipse CDT
^^^^^^^^^^^^^^^^^

^^^^^^^^^^^^^^^^^
使用Cygwin
^^^^^^^^^^^^^^^^^
建议使用mintty

---------------
推送技术原理
---------------

推送的本质原理就是TCP长连接，配合心跳周期定期刷新路由器的Nat表，以实现TCP的稳定连接，但具体实践中也有多种协议的实现，常见的一般都使用自定义的应用层协议，配合如Protobuf等拆包封包，现在随着HTML5的兴起，Websocket也逐渐走入了开发者的视野中，亦不失为一种不错的选择。

ESPUSH使用的是经典TCP长连接而非Websocket，未来有可能在NodeMCU平台上使用Lua实现Websocket的推送平台，以实现不需要刷固件（不需要刷固件，但仍然需要在NodeMCU中写入lua文件）即可使用ESPUSH的功能。

既然使用到TCP长连接心跳保持技术，这便对设备的能耗提出了更多的要求，设备将不得断开网络，这将意味着您无法使用ESP8266的`深度睡眠`功能。同时对设备的网络模式提出了要求，设备将不得处于SoftAP模式，如若处于AP模式则芯片将不能正常访问外部Internet网络，将无法通过云端对其进行实时操控。当您需要退出实时操控时，请调用 **push_unregister** 函数，芯片将断开与云端的连接，同时云端亦将无法再行控制设备，各种服务端SDK对设备的操作如数据推送、云端推送升级等亦将失败。


---------------
与Arduino互通
---------------

---------------
SDK开发库
---------------

首先当然是clone 项目https://github.com/pushdotccgzs/espush_sdk

本质上，我的工作只是在官方sdk上新增了库文件libpush.a，并修改其makefile而已，官方最新版SDK于此：http://bbs.espressif.com/viewtopic.php?f=5&t=481&sid=2b010931ef357b2847f14a6f012e2d84

客户端的使用方式极为简单，克隆此github库https://github.com/pushdotccgzs/esp_push_example，使用如下代码：

.. code-block: shell

    make clean && make BOOT=new APP=1
    #即可编译出user1，同理使用
    make clean && make BOOT=new APP=2
    即可编译出user2.


本质上只是在乐鑫官方库http://bbs.espressif.com/viewtopic.php?f=5&t=321的基础上增加了用于推送的 **libpush** 库，如果开发者使用如安信可IDE等，可直接编译出对应的flash rom。


^^^^^^^^^^^^^^
C API
^^^^^^^^^^^^^^


