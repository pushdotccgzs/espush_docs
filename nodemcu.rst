
================
NodeMCU固件
================

如果你熟悉了使用AT固件的方式，那使用NodeMCU固件连上ESPUSH云，就和 **把大象关进冰箱** 一样容易。

----------------
基础使用
----------------

与AT固件类似，克隆ESPUSH官方github库并直接编译即可，参考以下命令：

.. code-block:: shell

    git clone https://github.com/pushdotccgzs/espush_nodemcu.git
    cd espush_nodemcu
    make clean && make

NodeMCU固件由于源码量巨大，编译时间较久，可能持续几分钟甚至十多分钟，请耐心等待，第一次编译完成，后续更改代码后再编译将变得迅速。关于编译环境搭建，参考 **进阶指南** 部分。

使用NodeMCU固件在espush.cn操作步骤与AT固件完全一致，帐号不必重复注册，甚至设备类别也不用重复添加，使用AT固件现有的即可，下文主要阐述NodeMCU的 Lua API。

----------------
NodeMCU Lua API
----------------

最简单的使用Espush for NodeMCU的示例代码如下，寥寥数行代码即可完成所有工作，最大程度简化代码量

.. code-block:: lua

    -- 首先使用NodeMCU Lua语法，将模块置为STATION模式，并连接路由器，使之能连接外部网络
    wifi.setmode(wifi.STATION)
    -- 记得将下面的SSID与密码修改为你自己的
    wifi.sta.config("YOUR SSID","YOURSSIDPWD")
    -- 只需要一行代码即可连入Espush云环境
    push.regist(1234, "25b28f0ffb9711e4a96d446d579b49a1", function(msg)print('RECV: ' .. msg)end)
    -- 如果你想查询当前连接状态，请使用如下指令，返回2时代表连接成功 
    print(push.get_status())


**push.regist(appid, appkey, data_rcv_cb)**，主要的，最重要的函数，连入espush云环境，请指定正确的appid与appkey，错误的值将导致模块不断重试。参数 *data_rcv_cb* 为最终收到推送指令时的回调函数，推送Lua指令时将不会调用此回调，只有推送文本指令与十六进制数据指令时方能被调用。

**push.unregist()** 断开与云的连接，断开后将不能再次发送数据，也无法收到云端推送的任何指令，断开后可重连，重连时可指定不同的设备分类appid与appkey。

**push.get_status()** 或者当前与云端的连接状态，与AT固件定义相同。

**push.pushmsg(ms)** 主动推送数据，用于传感器将自身数据推送到云端，开发者可使用服务端接口同步数据。

