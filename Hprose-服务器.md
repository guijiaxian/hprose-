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

# Server 与 Service 的区别

Hprose 的服务器端的实现，分为 `Service` 和 `Server` 两部分。

其中 `Service` 部分是核心功能，包括接收请求，处理请求，服务调用，返回应答等整个服务的处理流程。

而 `Server` 则主要负责启动和关闭服务器，它包括设置服务地址和端口，设置服务器启动选项，启动服务器，接收来自客户端的连接然后传给 `Service` 进行处理。

之所以分开，是为了更方便的跟已有的库和框架结合，例如：swoole, yii、symfony 等版本都是直接继承 `Service` 来实现自己的服务，之后再创建自己的 `Server` 用于服务设置和启动。

分开的另外一个理由是，`Server` 部分的实现是很简单的，有时候开发者可能会希望把 Hprose 服务结合到自己的某个服务器中去，而不是作为一个单独的服务器来运行，在这种情况下，也是直接使用 `Service` 就可以了。

当开发者没有什么特殊需求，只是希望启动一个独立的 Hprose 服务器时，那使用 `Server` 就是一个最方便的选择了。