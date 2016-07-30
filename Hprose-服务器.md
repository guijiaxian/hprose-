# 概述

Hprose 2.0 for PHP 支持多种底层网络协议绑定的服务器，比如：HTTP 服务器，Socket 服务器和 WebSocket 服务器。

其中 HTTP 服务器支持在 HTTP、HTTPS 协议上通讯。

Socket 服务器支持在 TCP、Unix Socket 协议上通讯，并且支持全双工和半双工两种模式。

WebSocket 服务器支持在 ws、wss 协议上通讯。

其中，Hprose 2.0 的核心库提供了 HTTP 服务器和 Socket 服务器，基于 Swoole 的版本提供了 HTTP、Socket、WebSocket 服务器。另外还有 Yii2、Symfony、PSR7 版本的服务器。

为了清晰对比，这里列出一个表格：

功能列表          |    hprose-php     |    hprose-swoole   |     hprose-yii     |    hprose-symfony   |    hprose-psr7
:---------------:|:------------------:|:------------------:|:------------------:|:------------------:|:------------------:
  HTTP 服务器     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: 
 Socket 服务器    | :white_check_mark: | :white_check_mark: |        :x:         |        :x:         |        :x:         
WebSocket 服务器  |        :x:         | :white_check_mark: |        :x:         |        :x:         |        :x:         
   命令行环境     | :white_check_mark: | :white_check_mark: |        :x:         |        :x:         | :white_check_mark: 
  非命令行环境    | :white_check_mark: |         :x:        | :white_check_mark: | :white_check_mark: | :white_check_mark: 
     推送服务     | :white_check_mark: | :white_check_mark: |        :x:         |        :x:         |        :x:         

尽管支持这么多不同的底层网络协议，但除了在对涉及到底层网络协议的参数设置上有所不同以外，其它的用法都完全相同。因此，我们在下面介绍 Hprose 服务器的功能时，若未涉及到底层网络协议的区别，就以 HTTP 服务器为例来进行说明。

