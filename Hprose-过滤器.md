# 简介

有时候，我们可能会希望在远程过程调用中对通讯的一些细节有更多的控制，比如对传输中的数据进行加密、压缩、签名、跟踪、协议转换等等，但是又希望这些工作能够跟服务函数/方法本身可以解耦。这个时候，Hprose 过滤器就是一个不错的选择。

Hprose 过滤器是一个接口，它有两个方法：

```php
interface Filter {
    public function inputFilter($data, stdClass $context);
    public function outputFilter($data, stdClass $context);
}
```

其中 `inputFilter` 的作用是对输入数据进行处理，`outputFilter` 的作用是对输出数据进行处理。

`$data` 参数就是输入输出数据，它是 `string` 类型的。这两个方法的返回值也是 `string` 类型的数据，它表示已经处理过的数据，如果你不打算对数据进行修改，你可以直接将 `$data` 参数作为返回值返回。

`$context` 参数是调用的上下文对象，我们在服务器和客户端的介绍中已经多次提到过它。

# 执行顺序

不论是客户端，还是服务器，都可以添加多个过滤器。假设我们按照添加的顺序把它们叫做 `filter1`, `filter2`, ... `filterN`。那么它们的执行顺序是这样的。

## 在客户端的执行顺序

```
+------------------- outputFilter -------------------+
| +-------+      +-------+                 +-------+ |
| |filter1|----->|filter2|-----> ... ----->|filterN| |---------+
| +-------+      +-------+                 +-------+ |         v
+----------------------------------------------------+ +---------------+
                                                       | Hprose Server |
+-------------------- inputFilter -------------------+ +---------------+
| +-------+      +-------+                 +-------+ |         |
| |filter1|<-----|filter2|<----- ... <-----|filterN| |<--------+
| +-------+      +-------+                 +-------+ |
+----------------------------------------------------+
```

## 在服务器端的执行顺序

```
                  +-------------------- inputFilter -------------------+
                  | +-------+                 +-------+      +-------+ |
        +-------->| |filterN|-----> ... ----->|filter2|----->|filter1| |
        |         | +-------+                 +-------+      +-------+ |
+---------------+ +----------------------------------------------------+
| Hprose Client |                                                     
+---------------+ +------------------- outputFilter -------------------+
        ^         | +-------+                 +-------+      +-------+ |
        +---------| |filterN|<----- ... <-----|filter2|<-----|filter1| |
                  | +-------+                 +-------+      +-------+ |
                  +----------------------------------------------------+
```

# 跟踪调试

有时候我们在调试过程中，可能会需要查看输入输出数据。用抓包工具抓取数据当然是一个办法，但是使用过滤器可以更方便更直接的显示出输入输出数据。

**LogFilter.php**
```php
use Hprose\Filter;

class LogFilter implements Filter {
    public function inputFilter($data, stdClass $context) {
        error_log($data);
        return $data;
    }
    public function outputFilter($data, stdClass $context) {
        error_log($data);
        return $data;
    }
}
```

**Server.php**
```php
require_once 'LogFilter.php';

use Hprose\Socket\Server;

function hello($name) {
    return "Hello $name!";
}

$server = new Server('tcp://0.0.0.0:1143/');
$server->addFunction('hello');
$server->addFilter(new LogFilter());
$server->start();
```

**Client.php**
```php
require_once 'LogFilter.php';

use Hprose\Client;

$client = Client::create('tcp://127.0.0.1:1143/', false);
$client->addFilter(new LogFilter());
var_dump($client->hello("world"));
```

然后分别启动服务器和客户端，就会看到如下输出：

**服务器输出**
>
```
Cs5"hello"a1{s5"world"}z
Rs12"Hello world!"z
```
>

**客户端输出**
>
```
Cs5"hello"a1{s5"world"}z
Rs12"Hello world!"z
string(12) "Hello world!"
```
>