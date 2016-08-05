Hprose 服务器端提供了几个事件，它们分别是：

* `onBeforeInvoke`
* `onAfterInvoke`
* `onSendError`

这三个事件所有的 Hprose 服务器都支持。

* `onSendHeader`

这个事件仅 HTTP 服务器支持。

* `onAccept`
* `onClose`

这两个事件 Socket 和 WebSocket 服务器支持。

这些事件是以属性方式提供的，只需要将事件函数赋值给这些属性即可。

例如：

```php
$server->onBeforeInvoke = function($name, $args, $byref, \stdClass $context) {
    ...
}
$server->onAfterInvoke = function($name, $args, $byref, $result, \stdClass $context) {
    ...
}
$server->onSendError = function($error, \stdClass $context) {
    ...
}
```