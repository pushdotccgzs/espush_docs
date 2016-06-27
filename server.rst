服务端接口
==========

ESPUSH云服务，是针对ESP8266EX芯片开发的便捷远控服务，目前主要服务于云端设备控制。现提供此份服务端接口给需要的开发者，方便自行开发第三方应用。API的使用形式与其他推送平台如\ `信鸽 <%5Bhttp://xg.qq.com/>`__\ 等类似，简述如下。

协议整体描述
------------

请求URL结构为：\*\*\ `http://接口域名/openapi/接口命令/?接口参数\*\*，其中各字段取值如下 <http://接口域名/openapi/接口命令/?接口参数**，其中各字段取值如下>`__\ ：

-  接口域名，域名，当前取值espush.cn 使用https://协议
-  openapi，固定取值，区分主站资源
-  接口命令，接口命令资源地址，详见下文
-  接口参数，使用HTTP GET方式传递的参数，包括通用参数、接口指定参数等

通用参数
--------

各接口URL都需要在URL中传递通用参数，参数列表简单摘要如下：

+-------------+--------------+----------+---------------------------------------------+
| 字段        | 值类型       | 必填项   | 示例                                        |
+=============+==============+==========+=============================================+
| appid       | 整形         | 是       | 在https://espush.cn/webv2/apps/中进行管理   |
+-------------+--------------+----------+---------------------------------------------+
| timestamp   | 整形         | 是       | 形如1434447718                              |
+-------------+--------------+----------+---------------------------------------------+
| sign        | 32位字符串   | 是       | 生成方式参考下文                            |
+-------------+--------------+----------+---------------------------------------------+

说明：

-  **espush.cn的服务端API并不认证用户名与密码，只针对APPID与APPKEY，所以保护你的APPKEY，切勿泄露**\ 。
-  请求请定义您自己的UserAgent，目前平台无限制，但为了做简要区分，请尽量不要使用任何带有robot字样的UserAgent。
-  目前接口的调用频次无限制，但针对部分请求，频繁调用可能会导致设备重置。

通用返回结果
------------

-  接口使用非常简单的HTTP JSON
   API形式，成功的接口依据内容的不同返回内容也有不同，但都有共同的字段
   msg，用于表明此次请求的返回说明，成功的请求使用200作为其返回码.
-  使用HTTP返回码，如
   5XX代表服务端的错误，特别的，当与设备连接的网关服务器出错时，多返回502或504，并在msg中告知具体原因。4XX代表客户端的错误，通常401为sign取值错误，400是参数缺失，404是对应资源找不到。
-  服务端要求任何资源的请求末尾字符为
   **/**\ ，\ **？**\ 后的GET参数不计在内，如不应该请求
   ``https://espush.cn/openapi/apps?timestamp=1466598336&sign=a34e0360a05c739a08dd9ab404aec66b&appid=1234``
   而是应该发出\ ``https://espush.cn/openapi/apps/?timestamp=1466598336&sign=a34e0360a05c739a08dd9ab404aec66b&appid=1234``\ 这样的请求，区别在于
   apps 后是否有结束符 **/**\ ，前者将会收到一个301的重定向返回。

签名认证方式
------------

内容签名sign的计算方法如下：

-  提取请求的方法本身的小写字母表示，如 **get**,
   **post**\ 等，记作字符串A
-  提取请求的其他参数（GET与POST参数均需要，COOKIES参数不计入）但不包括sign，并按Key-Value的形式并转换为小写字母表示（未做urlencode），再按Key的降序排列（排序时，Value未参与，只对Key的小写字母表示做字母表的降序排列），组成如下形式：\*\ *key3=v3&key2=v4&key1=v1*\ \*，此处记作字符串B
-  提取APPKEY，记字符串C
-  Sign的值为 S = lower(MD5(A+B+C))

| 如POST请求，现有如下信息：APPID为1234，APPKEY为25b28f0ffb9711e4a96d446d579b49a1，无其他额外的请求参数，则只有通用参数appid、timestamp与sign，则字符串S应为:
  **gettimestamp=1433814203&appid=123425b28f0ffb9711e4a96d446d579b49a1**\ ，对其进行MD5运算，取运算后的\ **小写字母**\ 表示，即得到最终sign值:
| **9f0b613de12d5bb0451c556900a39559**

接口详细定义
------------

获取指定APPID的设备详情信息
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    接口地址 apps/

-  请求方法 GET
-  请求参数 无其他参数，使用公共参数， appid，timestamp，sign，使用HTTP
   GET方式请求即可
-  请求示例（以下请求示例以Win32平台下的cURL为准，\*Nix平台请替换双引号为单引号；且以下所有示例请求，使用appid为\ ``1234``\ ，appkey为\ ``25b28f0ffb9711e4a96d446d579b49a1``\ ，设备芯片编号为\ ``8454703``\ ，这并不是真实存在的设备分类，请不要使用。）：

   .. code:: bash

       curl "https://espush.cn/openapi/apps/?timestamp=1466598336&sign=a34e0360a05c739a08dd9ab404aec66b&appid=1234"

-  返回示例，为了开发方便，此处直接返回了在线设备，如下示例：

   .. code:: json

       {"name": "DevMine", "online": [{"chipid": 8454703, "name": 8454703, "devid": 8454703, "appid": 1234, "vertype": 5, "devkey": "AT_DEV_ANONYMOUS", "latest": "2016-06-22 20:20:06"}]}

-  功能说明 返回指定设备分类名称
-  注意事项，用于扫描二维码后，
   显示设备分类名称同时用于校验二维码是否有效，（是否已经被用户删除了该分类）。有效的方能被返回。

获取在线设备列表
~~~~~~~~~~~~~~~~

::

    接口地址 devices/list/

-  请求方法 GET
-  请求参数 无其他参数，使用公共参数， appid，timestamp，sign，使用HTTP
   GET方式请求即可
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/devices/lists/?timestamp=1466598934&sign=0597370ae9f426225f1c9bdae08593dc&appid=1234"

-  返回示例，返回数组里包裹对象，对象为单个设备的信息，包括其所属的设备分类APPID、最后检测到的存活时间等。

   .. code:: json

       [{"chipid": 8454703, "name": null, "devid": 8454703, "appid": 1234, "vertype": 5, "devkey": "AT_DEV_ANONYMOUS\u0000\u0000\u0000\u0000INVALID CONF", "latest": "2016-06-22 20:20:06"}]

-  功能说明
   返回指定设分备类名称，返回某分类下所有在线的终端，等同于web控制台的在线设备功能

向指定设备推送数据或指令
~~~~~~~~~~~~~~~~~~~~~~~~

::

    接口地址 dev/push/message/

-  功能：向指定的单个设备推送指令或消息，当前使用模块的芯片chipid，后期可能增加使用者自定义的ID。此接口是面向单个设备的消息、指令推送，故返回结果需要等待设备的返回，若设备返回较慢，则此接口同理。客户端需要自行处理超时，一般10秒后若未收到客户端的返回则可认为客户端已下线。
-  请求方法 ：POST
-  参数：

   -  devid，devid亦即芯片编号，\ https://espush.cn/webv2/devices/
      上所示的芯片ID，必须为整数
   -  message，需要推送的内容，如内容较长，建议置入POST区域
   -  format，指令类型，可选类型有：'MSG', 'HEX', 'AT', 'LUA',
      'B64'，分别代表
      文本数据、二进制数据文本方式、远程AT指令、远程LUA文件、Base64数据（同二进制数据方式，只是会进行BASE64解码后再行推送）

-  请求示例：

   .. code:: bash

       curl "https://espush.cn/openapi/dev/push/message/?format=MSG&timestamp=1466599588&devid=8454703&sign=56dd0c5a5dd7f1752d9ae31c077a0fa8&appid=1234&message=HELLO" -X POST

-  返回示例：

   .. code:: json

       {"msg": "OK"}

-  注意事项，参数可以使用URL的方式即GET传递，也可以置入POST请求HTTP
   BODY，推荐后者。

批量推送数据或指令
~~~~~~~~~~~~~~~~~~

::

    接口地址：app/push/message/

-  功能说明：面向指定的设备分类群推消息，目前的接口不会缓存消息，只有该分类下的终端/模块在线时方能推送。且发出此接口后，接口会立即返回，并不会等待所有的模块接收到指令后方返回，故生效可能存在延迟。
-  参数参数：message，需要推送的指令或数据。
-  请求示例

   .. code:: bash

       curl -X POST "https://espush.cn/openapi/app/push/message/?timestamp=1466639683&message=HELLO&format=MSG&sign=849423a3ded5d5e55deae614c00f4b1c&appid=1234"

-  返回示例

   .. code:: json

       {"msg": "success"}

平台数据同步
~~~~~~~~~~~~

::

    接口地址：up_messages/dev/:chipid/

-  功能说明
   查看模块/终端推送到云平台的数据，多为AT固件使用\ ``AT+PUSHMSG=``\ 传递到平台的数据
-  请求参数，无，使用通用参数
-  请求方法，GET
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/up_messages/dev/8454703/?timestamp=1466643867&sign=7a9cca6a064413b0e25c10f8099f09d1&appid=1234"

-  返回示例

   .. code:: json

       [{"body": "IM HERE", "app": "DevMine", "create_time": "2016-06-23 01:10:16", "id": 418335, "dev": "8454703"}]

平台指令历史
~~~~~~~~~~~~

::

    接口地址：push_messages/dev/:chipid/

-  功能说明，平台下发的指令或数据历史记录
-  请求参数，无，使用通用参数
-  请求方法，GET
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/push_messages/dev/8454703/?timestamp=1466643753&sign=7622715527bae95b9d785b5bd11c4eb0&appid=1234"

-  返回示例

   .. code:: json

       [{"body": "48454c4c4f", "create_time": "2016-06-22 12:54:17", "msgtype": "\u63a8\u9001\u6d88\u606f", "id": "10750", "dev": "8454703"}]

按设备类型与起止日期批量数据同步
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    接口地址：sync/:begin_date/:end_date/

-  功能说明，数据同步，用以同步从模块上传到云端的数据。
-  请求参数，无，使用通用参数；begin\_date与end\_date形如20140501,
   20151231 这种形式。
-  请求方法，GET
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/sync/20151201/20160623/?timestamp=1466643634&sign=cd00c34376e49b220faaeead23a4fac8&appid=1234"

-  返回示例

   .. code:: json

       [{"body": "hello", "recv_time": "2016-06-07 01:13:57", "app": "DevMine", "chipid": "470103", "create_time": "2016-06-07 01:13:57", "id": 393869}]

实时状态回调
~~~~~~~~~~~~

::

    接口地址：rt_status/:chipid/

-  功能说明，存在这样一种非常常见的场景，第三方应用想通过平台通道知道模块处于何种状态，但这种业务层的状态却智能自己由代码实现，如得知当前某个传感器的值等，此时可于芯片固件内实现回调函数使其返回对应值即可。平台专属开发板，云端实时获取DHT11的读数，即使用了此项接口。此API目前在NodeMCU固件使用最为方便。
-  请求参数，key，即模块内注册的回调键
-  请求方法，GET
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/rt_status/8454703/?timestamp=1466643470&sign=b1ba99a27c2414798179237d42066b60&key=KEY&appid=1234"

-  返回示例，此请求返回对应回调函数的返回，各有不同难以一一示例。

获取模块各GPIO口简要信息
~~~~~~~~~~~~~~~~~~~~~~~~

::

    接口地址：gpio_status/:chipid/

-  功能说明，获取设备GPIO口电平状态，此API与下一接口，主要用于首页App，远控之用。
-  请求参数，无，使用通用参数
-  请求方法，GET
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/gpio_status/8454703/?timestamp=1466642946&sign=4d37cfc0233fc13b4858f0a35a48926a&appid=1234"

-  返回示例

   .. code:: json

       {"result": "\u0001\u0000\u0001\u0000\u0001\u0000\u0000\u0000\u0000\u0000\u0000\u0000"}

获取模块各GPIO口的状态

简单设置模块GPIO口电平信号
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

    接口地址：set_gpio_edge/:chipid/:pin/:edge/

-  功能说明，设置指定模块的GPIO口电平态，本处只限简单使用，输出高低电平，内部未做任何上拉使能、下拉使能处理，如需要输出pwm波形等，建议自行完成。
-  请求参数，无，使用通用参数；URL中，pin、edge等均为数字，其中edge仅限
   0、1，分别代表 低水位电平、高水位电平
-  请求方法，POST
-  请求示例

   .. code:: bash

       curl -X POST "https://espush.cn/openapi/set_gpio_edge/8454703/5/1/?timestamp=1466643023&sign=58415f36b76cede2e85c7fabcec7538d&appid=1234"

-  返回示例

   .. code:: json

       {"result": "\u0000"}

手动刷新设备存活状态
~~~~~~~~~~~~~~~~~~~~

::

    接口地址：manual_refresh/:chipid/

-  功能说明，强制刷新设备是否在线，刷新原理为服务端推送等待响应，设定10秒超时。故此请求，可能最迟需要10秒才能返回，但当设备在线时，一般可以较迅速返回。
-  请求方法，GET
-  请求参数，无，使用通用参数
-  请求示例

   .. code:: bash

       curl "https://espush.cn/openapi/manual_refresh/8454703/?timestamp=1466642071&sign=9e43e558aa6a9937fe396b3b7ad574e4&appid=1234"

-  返回示例

   .. code:: json

       {"status": "online", "msg": "OK"}

云端重启设备
~~~~~~~~~~~~

::

    接口地址：reboot/dev/:chipid/

-  功能说明，远程控制重置设备

-  请求参数，无，使用通用参数
-  请求方法，POST
-  请求示例

   .. code:: bash

       curl -X POST "https://espush.cn/openapi/reboot/dev/8454703/?timestamp=1466641791&sign=ff42b5f6861e079855fab5abf8173661&appid=1234"

-  返回示例

   .. code:: json

       {"result": "\u0000"}

WebSocket接口
=============

当设备向平台推送数据时，第三方应用可使用WebSocket连接到平台，以获取设备的实时通知。应用应该使用WebSocket连接到以下URL以获取通知：

.. code:: text

    wss://espush.cn/noticed/peer?appid=1234&timestamp=1466642512648&chipid=8454703&
    sign=e54844ce4d29aff9920ccbec32be2e54

**注意**

-  WebSocket接口同样也使用了HTTPS，所以连接协议为 ``wss://``
-  末尾无需加 **/**
-  sign计算方式类同上文，计method为GET

应用在WebSocket上可能会收到的数据包括：

.. code:: json

    {
    "appid": "1234",
    "body": "VEVTVEFHQUlODQo=",
    "chipid": "8454703",
    "timestamp": "1466644524",
    "type": "data_upload"
    }

应用应该手动维持心跳以防止僵死连接，定时（建议每5分钟）发送以下请求：

.. code:: json

    {type: "heartbeat"}

如果需要向设备推送数据，使用如下格式的JSON即可：

.. code:: json

    {
    type: "msgpush",
    body: "SEVMTE8="
    }

留意将body串进行\ **base64编码后**\ 置于json中，且目前暂不支持指令直接推送。
