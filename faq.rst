===============
常见问题
===============


---------------
连不上云平台
---------------

^^^^^^^^^^^^
AT固件
^^^^^^^^^^^^

如果您连不上ESPUSH，请按如下步骤检查您的模块：
#. 检查模块运行状态，是否能正常输入AT指令，若AT指令无应答，则解决之
#. 检查模块当前状态，执行 **AT+CWMODE?** 只有处于STATION模式或STATIONAP模式才能连接到云服务，且当前不能打开SmartConfig
#. 检查模块STATION状态，如是否获得正确的IP地址，执行 **AT+CIPSTA?** 查看
#. 检查模块MAC地址配置是否正确，在刷写模块时会自动生成STATION模式的MAC地址，您可使用手机分享WIFI，模块连接手机时，可在手机端查知模块的STATION的MAC地址，若地址错误，亦将无法连接外部网络，这时您可能需要重新刷写固件。

---------------
关于在线状态
---------------

如果您在Web控制台的在线设备中发现了异常，一般而言有这两种情况

* Web控制台中发现了两个设备ID一模一样的，但最后存活时间不同，这是由于系统的心跳机制引起的bug，您可以手动刷新，真正离线的设备在刷新完毕后即会消失。
* Web控制台没有发现应该在线的设备


