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

# 创建服务器

创建服务器有多种方式，我们先从最简单的方式说起。

## 使用 Server 构造器函数

### 创建 HTTP 服务器

```php
use Hprose\Http\Server;

function hello($name) {
    return "Hello $name!";
}

$server = new Server();
$server->addFunction('hello');
$server->start();
```

该服务是基于 php-fpm 或者任何支持运行 PHP 的 Web 服务器（例如 Apache、IIS 等）。把它丢到配置好 PHP 的 Web 服务器上，然后在浏览器端打开该文件，你应该会看到以下输出：

>
```
Fa2{u#s5"hello"}z
```
>

如果你用过 Hprose 1.x，你会发现 2.0 版本多了一个 `u#`，这表示有一个名字叫 `#` 的方法，这是 Hprose 2.0 服务器端一个特殊的方法，用来产生一个唯一编号，客户端可以调用该方法得到一个唯一编号用来标识自己。Hprose 2.0 的客户端在使用推送功能时，如果没有手动指定 `id` 的情况下，会自动调用该方法来获取 `id`。

该方法的默认实现很简单，就是一个自增计数，所以一旦当服务器关闭之后重启，该方法会重新开始从 `0` 开始计数。这种方式可能不适用于某些场合，因此，你可能希望能够使用自己的实现来替换它，这是可以做到的。你只需要在发布方法时，将别名指定为 '#' 就可以覆盖默认实现了。

虽然上面对 '#' 这个方法介绍了这么多，然而该服务器并不支持推送服务。如果你真的需要推送服务，你需要创建一个独立的服务器。例如：

```php
use Hprose\Swoole\Server;

function hello($name) {
    return "Hello $name!";
}

$server = new Server("http://0.0.0.0:8086");
$server->addFunction('hello');
$server->start();
```

这是一个基于 Swoole 的 HTTP 的 Hprose 独立服务器。这代码跟上面的代码几乎一样。只是 `use` 的路径不同，`new Server` 时多了一个服务器地址。其它的都一模一样。

但是这段代码的运行方式跟上面那个服务器却完全不同。要运行该服务器，只需要在命令行中键入：

```
php HelloServer.php
```

上面假设跟这个文件保存为 `HelloServer.php`。你的服务就运行起来了，现在你的命令行会卡在那里不动了。但是你在浏览器中输入：`http://<IP>:8086`，其中 `<IP>` 是你运行这个程序的那台服务器的 IP 地址，如果跟你浏览器是同一台机器，IP 可以是 `127.0.0.1`，如果是不同的机器，你应该比我更清楚是多少，所以我就假设你输入正确了，然后你在浏览器端应该看到跟上面那个程序同样的输出（我就不再重复了）。

如果你的浏览器打不开，请检查你的地址是否输入正确，或者你是否开了防火墙把该服务给屏蔽了，或者你的网线是否插好了，诸如此类的问题我就不再一一列举，并且给出解决方案了。如果你解决不了，我也无能为力，所以请不要费劲地把此类问题提交到 [[issues|https://github.com/hprose/hprose-php/issues]] 中了，我真的帮不了你。

