---
layout: post
title:  "UPnP协议通信"
date:   2017-05-15 19:55:39 +0800
categories: 网络
---

# Android WIFI peer-to-peer网络

Wi-Fi peer-to-peer（P2P，对等网络），它允许具备相应硬件的Android 4.0（API level 14）或者更高版本的设备可以直接通过wifi而不需要其它中间中转节点就能直接通信（Android的Wi-Fi P2P框架符合Wi-Fi联盟的Wi-Fi Direct™直连认证标志）。使用这些API，你可以搜索并连接其它同样支持Wi-Fi P2P的设备，然后再通过一个高速的连接进行互相通信，并且这个连接的有效距离要比蓝牙连接的有效距离要长的多。这对于需要在用户之间共享数据的应用程序非常有用，例如多玩家游戏或者照片分享之类应用。

在3G和4G网络下手机端IP是电信供应商服务器的NAT地址，不可能成功的通过端口映射和UDP打洞技术突破电信防火墙的方案来实现P2P网络。WIFI网络类似于本地局域网，目前移动端设备实现的P2P网络仅限于WIFI网络环境下。因此，现在或者将来某一段时间内不可能有实现手机终端P2P的成熟方案。

详细情况请参考：  
[http://blog.csdn.net/qinxiandiqi/article/details/41213535](http://blog.csdn.net/qinxiandiqi/article/details/41213535)
[http://blog.csdn.net/u011913612/article/details/52804299](http://blog.csdn.net/u011913612/article/details/52804299)
