# PHP 5.6.x 版本迁移至 PHP 7.0.x 版本
## 7.0.x版本开始停用的特性
### PHP4风格的构造函数 
在PHP4风格的构造函数（与所在类名同名的方法），这一特性在PHP7中被废弃，若使用此种构造函数会发出 **E_DEPRECATED** 错误。当方法名与类名相同，并且类不在命名空间中，并且PHP5的构造函数（__construct）不存在时，会产生一个**E_DEPRECATED**错误。
```PHP
<?php
class foo {
    function foo() {
        echo 'I am the constructor';
    }
}
?>
```
上述代码输出：
```PHP
Deprecated: Methods with the same name as their class will not be constructors in a future version of PHP; foo has a deprecated constructor in example.php on line 3
```

### [password_hash()](http://php.net/manual/en/function.password-hash.php) salt选项
[password_hash()](http://php.net/manual/en/function.password-hash.php) 函数salt选项已被弃用，以防止开发人员使用自己的（通常是不安全的）salt。函数本身会在没有提供salt选项时生成加密安全的salt，不再需要开发者自定义生成salt了。

## 用户贡献记录
暂无
