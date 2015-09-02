# PHP 5.6.x 版本迁移至 PHP 7.0.x 版本
## 新的函数
### [Closure](http://php.net/manual/en/class.closure.php) 
- [Closure::call()](http://php.net/manual/en/closure.call.php)

### [CSPRNG](http://php.net/manual/en/book.csprng.php) 
- [random_bytes()](http://php.net/manual/en/function.random-bytes.php)
- [random_int()](http://php.net/manual/en/function.random-int.php)

###[错误处理和日志记录](http://php.net/manual/en/book.errorfunc.php)
- [error_clear_last()](http://php.net/manual/en/function.error-clear-last.php)

###[GNU Multiple Precision](http://php.net/manual/en/book.gmp.php)
- [gmp_random_seed()](http://php.net/manual/en/function.gmp-random-seed.php)

###[Math](http://php.net/manual/en/book.math.php)
- [intdiv()](http://php.net/manual/en/function.intdiv.php)

###[PCRE](http://php.net/manual/en/book.pcre.php)
- [preg_replace_callback_array()](http://php.net/manual/en/function.preg-replace-callback-array.php)

###[POSIX](http://php.net/manual/en/book.posix.php)
- posix_setrlimit()

###[反射](http://php.net/manual/en/book.reflection.php)
- [ReflectionParameter::getType()](http://php.net/manual/en/reflectionparameter.gettype.php)
- [ReflectionFunctionAbstract::getReturnType()](http://php.net/manual/en/reflectionfunctionabstract.getreturntype.php)

###[Zip](http://php.net/manual/en/book.zip.php)
- [ZipArchive::setCompressionIndex()](http://php.net/manual/en/ziparchive.setcompressionindex.php)
- [ZipArchive::setCompressionName()](http://php.net/manual/en/ziparchive.setcompressionname.php)

###[Zlib压缩](http://php.net/manual/en/book.zlib.php)
- inflate_add()
- deflate_add()
- inflate_init()
- deflate_init()

## 用户贡献记录
暂无
