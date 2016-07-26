# 概述

Hprose 2.0 for PHP 客户端相比 1.x 版本有了比较大的改动。

核心版本除了提供了客户端的基本实现的基类以外，还提供了 HTTP 客户端和 Socket 客户端。这两个客户端都可以创建为同步或异步客户端。这两个客户端既可以在命令行环境下使用，也可以在 php-fpm 或其他 PHP 环境下使用。

另外 [[swoole 版本|https://github.com/hprose/hprose-swoole]] 提供了纯异步的 HTTP 客户端，Socket 客户端和 WebSocket 客户端。Swoole 客户端只能在命令行环境下使用。

其中 HTTP 客户端支持跟 HTTP、HTTPS 绑定的 Hprose 服务器通讯。

Socket 客户端支持跟 TCP、Unix Socket 绑定的 Hprose 服务器通讯，并且支持全双工和半双工两种模式。

WebSocket 客户端支持跟 ws、wss 绑定的 Hprose 服务器通讯。

为了清晰对比，这里列出一个表格：

功能列表          |    hprose-php     |    hprose-swoole
:---------------:|:------------------:|:-----------------:
   同步调用       | :white_check_mark: |        :x:  
   异步调用       | :white_check_mark: | :white_check_mark:  
  HTTP 客户端     | :white_check_mark: | :white_check_mark:  
 Socket 客户端    | :white_check_mark: | :white_check_mark: 
WebSocket 客户端  |        :x:         | :white_check_mark:
   命令行环境     | :white_check_mark: | :white_check_mark: 
  非命令行环境    | :white_check_mark: |         :x: 

尽管支持这么多不同的底层网络协议，但除了在对涉及到底层网络协议的参数设置上有所不同以外，其它的用法都完全相同。因此，我们在下面介绍 Hprose 客户端的功能时，若未涉及到底层网络协议的区别，就以 HTTP 客户端为例来进行说明。

# 创建客户端

创建客户端有两种方式，一种是直接使用构造器，另一种是使用工厂方法 create。

## 使用构造器创建客户端

`Hprose\Client` 是一个抽象类，因此它不能作为构造器直接使用。如果你想创建一个具体的底层网络协议绑定的客户端，你可以将它作为父类，至于如何实现一个具体的底层网络协议绑定的客户端，这已经超出了本手册的内容范围，这里不做具体介绍，有兴趣的读者可以参考 `Hprose\Http\Client`、`Hprose\Socket\Client` 和 `Hprose\Swoole\WebSocket\Client` 等底层网络协议绑定客户端的实现源码。

`Hprose\Http\Client`、`Hprose\Socket\Client` 这两个类是可以直接使用的构造器。它们分别对应 HTTP 客户端、Socket 客户端。

创建方式如下：

```php
$client = new \Hprose\Http\Client([$uris = null[, $async = true]]);
```

`[]` 内的参数表示可选参数。

当两个参数都省略时，创建的客户端是未初始化的异步客户端，后面需要使用 `useService` 方法进行初始化，这是后话，暂且不表。

第 1 个参数 `$uris` 是服务器地址，该服务器地址可以是单个的地址字符串，也可以是由多个地址字符串组成的数组。当该参数为多个地址字符串组成的数组时，客户端会从这些地址当中随机选择一个作为服务地址。因此需要保证这些地址发布的都是完全相同的服务。

第 2 个参数 `$async` 表示是否是异步客户端，在 Hprose 2.0 for PHP 中，默认创建的都是异步客户端，这是因为 Swoole 客户端只支持异步，为了可以方便的在普通客户端和 Swoole 客户端之间切换，所以默认设置为异步。异步客户端在进行远程调用时，返回值为一个 `promise` 对象。而同步客户端在进行远程调用时，返回值为实际返回值（或者抛出异常）。客户端创建之后，该类型不能被更改。

例如：

**创建一个同步的 HTTP 客户端**
```php
$client = new \Hprose\Http\Client('http://hprose.com/example/', false);
```

**创建一个同步的 TCP 客户端**
```php
$client = new \Hprose\Socket\Client('tcp://127.0.0.1:1314', false);
```

**创建一个异步的 Unix Socket 客户端**
```php
$client = new \Hprose\Socket\Client('unix:/tmp/my.sock');
```

**创建一个异步的 WebSocket 客户端**
```php
$client = new \Hprose\Swoole\WebSocket\Client('ws://127.0.0.1:8080/');
```

>
注意：如果要使用 swoole 客户端，需要在 composer.json 加入对 `hprose/hprose-swoole` 的引用。且 Swoole 客户端不支持第二个参数。
>

另外，如果创建的是 Swoole 的客户端，还有更简单的方式：

**同样创建一个异步的 WebSocket 客户端**
```php
$client = new \Hprose\Swoole\Client('ws://127.0.0.1:8080/');
```

也就是说，只需要使用 `Hprose\Swoole\Client`，就可以创建所有 Swoole 支持的客户端了，Hprose 可以自动根据服务器地址的 scheme 来判断客户端类型。


## 通过工厂方法 `create` 创建客户端

```php
$client = \Hprose\Client->create($uris = null[, $async = true]);
```

`create` 方法与构造器函数的参数一样，返回结果也一样。但是第一个参数 `$uris` 不能被省略。

使用 `create` 方法更加方便，因此，除非在创建客户端的时候，不想指定服务地址，否则，应该优先考虑使用 `create` 方法来创建客户端。

`\Hprose\Client->create` 支持创建 Hprose 核心库上的客户端，`Hprose\Swoole\Client->create` 支持创建 swoole 的客户端。例如：

**创建一个同步的 HTTP 客户端**
```php
$client = \Hprose\Client->create('http://hprose.com/example/', false);
```

**创建一个同步的 TCP 客户端**
```php
$client = \Hprose\Client->create('tcp://127.0.0.1:1314', false);
```

**创建一个异步的 Unix Socket 客户端**
```php
$client = \Hprose\Client->create('unix:/tmp/my.sock');
```

**创建一个异步的 WebSocket 客户端**
```php
$client = new \Hprose\Swoole\Client-create('ws://127.0.0.1:8080/');
```

## 注册自己的客户端实现类

如果你自己创建了一个客户端实现，你可以通过：
```php
Client::registerClientFactory($scheme, $clientFactory);
```

或者
```php
Client::tryRegisterClientFactory($scheme, $clientFactory);
```

这两个静态方法来注册自己的客户端实现。

注册之后，你就可以使用 `create` 方法来创建你的客户端对象了。

`registerClientFactory` 方法会覆盖原来已注册的相同 `$scheme` 的客户端类，`tryRegisterClientFactory` 方法不会覆盖。

## uri 地址格式

### HTTP 服务地址格式

HTTP 服务地址与普通的 URL 地址没有区别，支持 `http` 和 `https` 两种协议，这里不做介绍。

### WebSocket 服务地址格式

除了协议从 `http` 改为 `ws`（或 `wss`） 以外，其它部分与 `http` 地址表示方式完全相同，这里不再详述。

### TCP 服务地址格式

```
<scheme>://<ip>:<port>
```

`<ip>` 是服务器的 IP 地址，也可以是域名。

`<port>` 是服务器的端口号，hprose 的 TCP 服务没有默认端口号，因此不可省略。

`<scheme>` 表示协议，它可以为以下取值：

* `tcp`
* `ssl`
* `sslv2`
* `sslv3`
* `tls`

`tcp` 表示 tcp 协议，地址可以是 ipv6 地址，也可以是 ipv4 地址。

`ssl`, `sslv2`, `sslv3` 和 `tls` 表示安全的 tcp 协议。如有必要，可设置客户端安全证书。

### Unix Socket 服务地址格式

```
unix:<path>
```

其中 `<path>` 是绝对路径（以 `/` 开头）。例如：

```
unix:/tmp/my.sock
```

# 属性

## onError 属性

该属性为 `callable` 类型，默认值为 `NULL`。

当客户端采用回调方式进行调用时，并且回调函数没有参数，如果发生异常，该属性会被调用。回调函数格式为：

```
function onError($name, $e);
```

$name 是字符串类型，$e 在 PHP 5 中 为 Exception 类型或它的子类型对象，在 PHP 7 中是 Throwable 接口的实现类对象。

## uri 属性

只读属性。字符串类型，表示当前客户端所调用的服务地址。

## uris 属性

只读属性。数组类型，表示当前客户端可以调用的服务器地址列表。

## filters 属性

只读属性。数组类型，表示当前客户端上添加的过滤器列表。

## timeout 属性

整数类型，默认值为 `30000`，单位是毫秒（ms）。表示客户端在调用时的超时时间，如果调用超过该时间后仍然没有返回，则会以超时错误返回。

## failswitch 属性

布尔类型。默认值为 `false`。表示当前客户端在因网络原因调用失败时是否自动切换服务地址。当客户端服务地址仅设置一个时，不管该属性值为何，都不会切换地址。

## idempotent 属性

布尔类型，默认值为 `false`。表示调用是否为幂等性调用，幂等性调用表示不论该调用被重复几次，对服务器的影响都是相同的。幂等性调用在因网络原因调用失败时，会自动重试。如果 `failswitch` 属性同时被设置为 `true`，并且客户端设置了多个服务地址，在重试时还会自动切换地址。

## retry 属性

整数类型，默认值为 `10`。表示幂等性调用在因网络原因调用失败后的重试次数。只有 `idempotent` 属性为 `true` 时，该属性才有作用。

## byref 属性

布尔类型，默认值为 `false`。表示调用是否为引用参数传递。当设置为引用参数传递时，服务器端会传回修改后的参数值（即使没有修改也会传回）。因此，当不需要该功能时，设置为 `false` 会比较节省流量。

## simple 属性

布尔类型，默认值为 `false`。表示调用中所传输的数据是否为简单数据。

简单数据是指：null、数字（包括整数、浮点数）、Boolean 值、字符串、日期时间等基本类型的数据或者不包含引用的数组和对象。当该属性设置为 `true` 时，在进行序列化操作时，将忽略引用处理，加快序列化速度。但如果数据不是简单类型的情况下，将该属性设置为 `true`，可能会因为死循环导致堆栈溢出的错误。

简单的讲，用 `JSON` 可以表示的数据都是简单数据。但是对于比较复杂的 `JSON` 数据，设置 `simple` 为 `true` 可能不会加快速度，反而会减慢，比如对象数组。因为默认情况下，hprose 会对对象数组中的重复字符串的键值进行引用处理，这种引用处理可以对序列化起到优化作用。而关闭引用处理，也就关闭了这种优化。

因为不同调用的数据可能差别很大，因此，建议不要修改默认设置，而是针对某个调用进行单独设置。