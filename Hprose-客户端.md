# 概述

Hprose 2.0 for PHP 客户端相比 1.x 版本有了比较大的改动。

核心版本除了提供了客户端的基本实现的基类以外，还提供了 HTTP 客户端和 Socket 客户端。这两个客户端都可以创建为同步或异步客户端。

另外 [[swoole 版本|https://github.com/hprose/hprose-swoole]] 提供了纯异步的 HTTP 客户端，Socket 客户端和 WebSocket 客户端。

其中 HTTP 客户端支持跟 HTTP、HTTPS 绑定的 Hprose 服务器通讯。

Socket 客户端支持跟 TCP、Unix Socket 绑定的 Hprose 服务器通讯，并且支持全双工和半双工两种模式。

WebSocket 客户端支持跟 ws、wss 绑定的 Hprose 服务器通讯。

尽管支持这么多不同的底层网络协议，但除了在对涉及到底层网络协议的参数设置上有所不同以外，其它的用法都完全相同。因此，我们在下面介绍 Hprose 客户端的功能时，若未涉及到底层网络协议的区别，就以 HTTP 客户端为例来进行说明。