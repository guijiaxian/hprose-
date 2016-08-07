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
use Hprose\Client;

$client = Client::create('tcp://127.0.0.1:1143/', false);
$client->addFilter(new LogFilter());
var_dump($client->hello("world"));
```

上面的服务器和客户端代码我们省略了包含路径。请自行脑补，或者直接参见 [[examples 目录|https://github.com/hprose/hprose-php/tree/master/examples/src]]里面的例子。

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

上面输出操作我们用了 `error_log` 函数，并不是因为我们要输出的内容是错误信息，而是因为 Hprose 内部在服务器端过滤了 `echo`、`var_dump `这些用 `ob_xxx` 操作可以过滤掉的输出信息，因为不过滤这些信息，一旦服务器有输出操作，就会造成客户端无法正常运行。所以，我们这里用 `error_log` 函数来进行输出。你也可以换成别的你喜欢的方式，只要不会被 `ob_xxx` 操作过滤掉就可以了。下面的例子我们同样使用这个函数来作为输出。

# 压缩传输

上面的例子，我们只使用了一个过滤器。在本例中，我们展示多个过滤器组合使用的效果。

**CompressFilter.php**
```php
use Hprose\Filter;

class CompressFilter implements Filter {
    public function inputFilter($data, stdClass $context) {
        return gzdecode($data);
    }
    public function outputFilter($data, stdClass $context) {
        return gzencode($data);
    }
}
```

**SizeFilter.php**
```php
use Hprose\Filter;

class SizeFilter implements Filter {
    private $message;
    public function __construct($message) {
        $this->message = $message;
    }
    public function inputFilter($data, stdClass $context) {
        error_log($this->message . ' input size: ' . strlen($data));
        return $data;
    }
    public function outputFilter($data, stdClass $context) {
        error_log($this->message . ' output size: ' . strlen($data));
        return $data;
    }
}
```

**Server.php**
```php
use Hprose\Socket\Server;

$server = new Server('tcp://0.0.0.0:1143/');
$server->addFilter(new SizeFilter('Non compressed'));
$server->addFilter(new CompressFilter());
$server->addFilter(new SizeFilter('Compressed'));
$server->addFunction(function($value) {
    return $value;
}, 'echo');
$server->start();
```

**Client.php**
```php
use Hprose\Client;

$client = Client::create('tcp://127.0.0.1:1143/', false);
$client->addFilter(new SizeFilter('Non compressed'));
$client->addFilter(new CompressFilter());
$client->addFilter(new SizeFilter('Compressed'));

$value = range(0, 99999);
var_dump(count($client->echo($value)));
```

然后分别启动服务器和客户端，就会看到如下输出：

**服务器输出**
>
```
Compressed input size: 216266
Non compressed input size: 688893
Non compressed output size: 688881
Compressed output size: 216245
```
>

客户端输出
>
```
Non compressed output size: 688893
Compressed output size: 216266
Compressed input size: 216245
Non compressed input size: 688881
int(100000)
```
>

在这个例子中，压缩我们使用了 PHP 内置的 gzip 算法，运行前需要确认你开启了这个扩展（一般默认就是开着的）。

加密跟这个类似，这里就不再单独举加密的例子了。