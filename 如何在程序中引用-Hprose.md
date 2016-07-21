如果你正在使用 composer 管理你的项目，那么你不需要做任何特别处理。只要在 composer.json 中的 `require` 段添加了对 `hprose/hprose` 的引用就可以了。

如果你不打算使用 composer 来管理你的项目，那你可以直接把 hprose-php 里面的 src 目录复制到你的项目中，然后改成任何你喜欢的名字，比如改为 hprose。

然后像这样引用它：

```php
<?php
require_once 'hprose/Hprose.php';
...
```