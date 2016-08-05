## crossDomain 属性

该属性用于设置是否允许浏览器客户端跨域调用本服务。默认值为 `false`。当设置为 `true` 时，自动开启 CORS 跨域支持。当配合 `addAccessControlAllowOrigin` 和 `removeAccessControlAllowOrigin` 这两个方法时，还可以做细粒度的跨域设置。

## isCrossDomainEnabled 方法

获取 `crossDomain` 的属性值。

## setCrossDomainEnabled 方法

设置 `crossDomain` 的属性值。

## p3p 属性

该属性用于设置是否允许 IE 浏览器跨域设置 Cookie。默认为 `true`。

## isP3PEnabled 方法

获取 `p3p` 的属性值。

## setP3PEnabled 方法

设置 `p3p` 的属性值。

## get 属性

该属性用于设置是否接受 GET 请求。默认为 `true`。如果你不希望用户使用浏览器直接浏览器服务器函数列表，你可以将该属性设置为 `false`。

## isGetEnabled 方法

获取 `get` 的属性值。

## setGetEnabled 方法

设置 `get` 的属性值。

## addAccessControlAllowOrigin 方法

添加允许跨域的地址。

## removeAccessControlAllowOrigin 方法

删除允许跨域的地址。