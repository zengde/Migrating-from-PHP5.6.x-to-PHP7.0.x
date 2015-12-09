# PHP5.6.x版本迁移至7.0.x版本
## 不向后兼容的变更
### 错误和异常处理相关的变更
在 PHP 7 中，很多致命错误以及可恢复的致命错误，都被转换为异常来处理了。 这些异常继承自 [**Error**](http://php.net/manual/en/class.error.php) 类，此类实现了 [**Throwable**](http://php.net/manual/en/class.throwable.php) 接口 （所有异常都实现了这个基础接口）。<br>

这也意味着，当发生错误的时候，以前代码中的一些错误处理的代码将无法被触发。 因为在 PHP 7 版本中，已经使用抛出异常的错误处理机制了。 （如果代码中没有捕获 [**Error**](http://php.net/manual/en/class.error.php) 异常，那么会引发致命错误）。

PHP 7 中的错误处理的更完整的描述，请参见 [PHP 7 错误处理](http://php.net/manual/zh/language.errors.php7.php)。 本迁移指导主要是列出对兼容性有影响的变更。

#### 当内部构造器失败的时候，总是抛出异常
在之前版本中，如果内部类的构造器出错，会返回 **NULL** 或者一个不可用的对象。 从 PHP 7 开始，如果内部类构造器发生错误， 那么会同一些用户类一样抛出[Exception异常](http://php.net/manual/en/class.exception.php)。

#### 解析错误会抛出 [ParseError](http://php.net/manual/en/class.parseerror.php) 异常
解析错误会抛出 [ParseError](http://php.net/manual/en/class.parseerror.php) 异常。对于 [eval()](http://php.net/manual/zh/function.eval.php) 函数，需要将其包含到一个 [catch](http://php.net/manual/zh/language.exceptions.php#language.exceptions.catch) 代码块中来处理解析错误。

#### E_STRICT 警告级别变更
所有 **E\_STRICT** 警告都被迁移到其他级别。**E\_STRICT**常量依然保留，所以调用 *error_reporting\(E\_ALL|E\_STRICT\)*不会引发错误。<br>

变更情况如下表
<table><tbody><tr style><th style="background-color:#C4C9DF;">场景</th>
<th style="background-color:#C4C9DF;">新的级别/行为</th>
</tr>
</tbody>
<tbody><tr><td>将资源类型的变量用作键来进行索引</td>
<td><span style="font-weight:bolder;">E_NOTICE</span></td>
</tr>
<tr><td>抽象静态方法</td>
<td>不再警告，会引发错误</td>
</tr>
<tr><td>重复定义构造器函数</td>
<td>不再警告，会引发错误</td>
</tr>
<tr><td>在继承的时候，方法签名不匹配</td>
<td><span style="font-weight:bolder;">E_WARNING</span></td>
</tr>
<tr><td>在两个 trait 中包含相同的（兼容的）属性</td>
<td>不再警告，会引发错误</td>
</tr>
<tr><td>以非静态调用的方式访问静态属性</td>
<td><span style="font-weight:bolder;">E_NOTICE</span></td>
</tr>
<tr><td>变量应该以引用的方式赋值</td>
<td><span style="font-weight:bolder;">E_NOTICE</span></td>
</tr>
<tr><td>变量应该以引用的方式传递（到函数参数中）</td>
<td><span style="font-weight:bolder;">E_NOTICE</span></td>
</tr>
<tr><td>以静态方式调用实例方法</td>
<td><span style="font-weight:bolder;">E_DEPRECATED</span></td>
</tr>
</tbody>
</table>

### 变量处理的变更
PHP7开始使用抽象语法树来解析PHP文件。许多的改进由于老版本的PHP解析器的局限性而不可能实现。导致了在一些特殊情况出现了向后不兼容的问题。本节详细介绍这些特殊情况。

#### 处理间接变量、属性和方法的变更 
间接的使用变量、属性和方法，现在开始严格按照从左到右的顺序执行，与以前的特殊情况的混合形式相对。下表列出了赋值顺序的变更。

<table><tbody><tr><th style="background-color:#C4C9DF;">表达式</th>
<th style="background-color:#C4C9DF;">PHP 5 解析器</th>
<th style="background-color:#C4C9DF;">PHP 7 解析器</th>
</tr>
</tbody>
<tbody class="tbody"><tr><td>$$foo['bar']['baz']</td>
<td>${$foo['bar']['baz']}</td>
<td>($$foo)['bar']['baz']</td>
</tr>
<tr><td>$foo-&gt;$bar['baz']</td>
<td>$foo-&gt;{$bar['baz']}</td>
<td>($foo-&gt;$bar)['baz']</td>
</tr>
<tr><td>$foo-&gt;$bar['baz']()</td>
<td>$foo-&gt;{$bar['baz']}()</td>
<td>($foo-&gt;$bar)['baz']()</td>
</tr>
<tr><td>Foo::$bar['baz']()</td>
<td>Foo::{$bar['baz']}()</td>
<td>(Foo::$bar)['baz']()</td>
</tr>
</tbody>
</table>

以前使用的从右到左的方式的代码需要通过花括号来明确顺序（见上表中间列），以使代码同时向前兼容PHP7.x，并向后兼容PHP5.x。

#### [list\(\)](http://php.net/manual/en/function.list.php) 函数处理的变更
##### [list\(\)](http://php.net/manual/en/function.list.php) 不再按照相反的顺序赋值变量
[list\(\)](http://php.net/manual/en/function.list.php)函数将按照变量定义的顺序赋值，而不是相反的顺序。一般来说，这只会影响[list\(\)](http://php.net/manual/en/function.list.php)函数结合数组的\[\]符号时的情况。举例如下：
```PHP
<?php
list($a[], $a[], $a[]) = [1, 2, 3];
var_dump($a);
?>
```
上述例子在PHP5中的输出为：
```PHP
array(3) {
  [0]=>
  int(3)
  [1]=>
  int(2)
  [2]=>
  int(1)
}
```
而在PHP7中的输出为：
```php
array(3) {
  [0]=>
  int(1)
  [1]=>
  int(2)
  [2]=>
  int(3)
}
```
在一般情况下，不建议依靠 [list\(\)](http://php.net/manual/en/function.list.php) 函数赋值的顺序，因为这个实现细节有可能会在未来改变。

##### 使用空的[list\(\)](http://php.net/manual/en/function.list.php) 函数参数被移除
[list\(\)](http://php.net/manual/en/function.list.php) 构造时不再允许参数为空的情况，下列情况将不再支持！
```PHP
<?php
list() = $a;
list(,,) = $a;
list($x, list(), $y) = $a;
?>
```

##### [list\(\)](http://php.net/manual/en/function.list.php) 函数不能解包字符串
[list\(\)](http://php.net/manual/en/function.list.php) 不再允许拆解[字符串](http://php.net/manual/en/language.types.string.php)变量为字母，而应该使用[str_split](http://php.net/manual/en/function.str-split.php)函数。

#### 在数组中的元素通过引用方式创建时，数组顺序会被改变
数组中的元素在通过引用方式创建时，其数组顺序会被自动的改变。例如：
```PHP
<?php
$array = [];
$array["a"] =& $array["b"];
$array["b"] = 1;
var_dump($array);
?>
```
PHP5中的输出：
```PHP
array(2) {
  ["b"]=>
  &int(1)
  ["a"]=>
  &int(1)
}
```
PHP7中的输出
```PHP
array(2) {
  ["a"]=>
  &int(1)
  ["b"]=>
  &int(1)
}
```

#### [global](http://php.net/manual/en/language.variables.scope.php#language.variables.scope.global) 仅支持简单的变量
[可变变量](http://php.net/manual/en/language.variables.variable.php)将不能再使用[global](http://php.net/manual/en/language.variables.scope.php#language.variables.scope.global)标记。如果真的需要，可以用花括号来间隔开写，例如下面代码：
```PHP
<?php
function f() {
    // Valid in PHP 5 only.
    global $$foo->bar;

    // Valid in PHP 5 and 7.
    global ${$foo->bar};
}
?>
```
作为一个基本原则，这样的变量套变量的使用方式，在[global](http://php.net/manual/en/language.variables.scope.php#language.variables.scope.global)这种场景下是不被提倡的。

#### 函数参数中的括号不再影响的行为
在PHP5中，函数通过引用传递的参数使用多余的括号包围，可以不出现严格标准警告。但在PHP7开始，都会发出该警告。
```PHP
<?php
function getArray() {
    return [1, 2, 3];
}

function squareArray(array &$a) {
    foreach ($a as &$v) {
        $v **= 2;
    }
}

// Generates a warning in PHP 7.
squareArray((getArray()));
?>
```
上述示例代码会输出：
```PHP
Notice: Only variables should be passed by reference in /tmp/test.php on line 13
```

### [foreach](http://php.net/manual/en/control-structures.foreach.php) 的改变
对 [foreach](http://php.net/manual/en/control-structures.foreach.php) 控制结构的行为作了细微的更改，主要是围绕处理数组遍历时的内部数组指针和迭代时对数组的修改。

#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 遍历期间不再修改数组指针
在PHP7之前，当数组通过[foreach](http://php.net/manual/en/control-structures.foreach.php)迭代时，数组指针会被修改。现在开始，不再如此，见下面代码：
```PHP
<?php
$array = [0, 1, 2];
foreach ($array as &$val) {
    var_dump(current($array));
}
?>
```
PHP5中的输出
```PHP
int(1)
int(2)
bool(false)
```
PHP7中的输出
```PHP
int(0)
int(0)
int(0)
```

#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 通过值遍历时，操作的值为数组的副本
当使用默认的通过值遍历数组时，[foreach](http://php.net/manual/en/control-structures.foreach.php)实际操作的是数组的迭代副本，而非数组本身。这就意味着，在迭代中数组的变化不会影响原数组的值。

#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 通过引用遍历时，有更好的迭代特性
当使用引用遍历时，现在[foreach](http://php.net/manual/en/control-structures.foreach.php)在迭代中更好的跟踪数组变化。例如，在迭代中添加一个值到数组中也会迭代添加的值，例如下面代码：
```PHP
<?php
$array = [0];
foreach ($array as &$val) {
    var_dump($val);
    $array[1] = 1;
}
?>
```
在PHP5的输出为：
```PHP
int(0)
```
在PHP7的输出为：
```PHP
int(0)
int(1)
```

#### [non-Traversable](http://php.net/manual/en/class.traversable.php) 对象的遍历
[non-Traversable](http://php.net/manual/en/class.traversable.php) 对象的遍历与宾利通过引用的数组相似，具有相同的行为特性，[对遍历期间修改数组的行为的改进](http://php.net/manual/en/migration70.incompatible.php#migration70.incompatible.foreach.by-ref)将同样应用到遍历对象时添加或删除对象的属性。

### [整形](http://php.net/manual/en/language.types.integer.php)处理上的调整
#### 无效的八进制文本
此前，八进制中包含无效数据会自动被截断（0128被当做为012）。现在，一个无效的八进制文本会造成解析错误。

#### 负位移
负数的位移将抛出一个 [ArithmeticError](http://php.net/manual/en/class.arithmeticerror.php) 
```PHP
<?php
var_dump(1 >> -1);
?>
```
PHP5中的输出：
```PHP
int(0)
```
PHP7中的输出：
```PHP
Fatal error: Uncaught ArithmeticError: Bit shift by negative number in /tmp/test.php:2
Stack trace:
#0 {main}
  thrown in /tmp/test.php on line 2
```
#### 超出范围的位移
位移（任一方向）超出一个整数的位宽度会得到0。以前，这种转变的行为是依赖于运行环境的机器架构。

### [字符串](http://php.net/manual/en/language.types.string.php)处理上的调整
#### 十六进制字符串不再被认为是数字
含十六进制数的字符串不再被认为是数字。例如：
```PHP
<?php
var_dump("0x123" == "291");
var_dump(is_numeric("0x123"));
var_dump("0xe" + "0x1");
var_dump(substr("foo", "0x1"));
?>
```
在PHP5中的输出：
```PHP
bool(true)
bool(true)
int(15)
string(2) "oo"
```
在PHP7中的输出：
```PHP
bool(false)
bool(false)
int(0)

Notice: A non well formed numeric value encountered in /tmp/test.php on line 5
string(3) "foo"
```
[filter_var\(\)](http://php.net/manual/en/function.filter-var.php) 函数可以用于检查一个字符串中是否包含十六进制数，同时也可以转换一个字符串为十六进制数。
```PHP
<?php
$str = "0xffff";
$int = filter_var($str, FILTER_VALIDATE_INT, FILTER_FLAG_ALLOW_HEX);
if (false === $int) {
    throw new Exception("Invalid integer!");
}
var_dump($int); // int(65535)
?>
```

#### \u{ 可能触发错误
由于添加了新的[Unicode Codepoint Escape Syntax](http://php.net/manual/en/migration70.new-features.php#migration70.new-features.unicode-codepoint-escape-syntax)，字符串中含有 **\\u{ **后面跟着无效的序列 时会触发Fatal错误。为了避免这一报错，应该转义开头的反斜杠。

### 被移除的函数
#### [call_user_method\(\)](http://php.net/manual/en/function.call-user-method.php) 与 [call_user_method_array\(\)](http://php.net/manual/en/function.call-user-method-array.php)
这些函数从PHP4.1.0开始已经停用，分别使用 [call_user_func\(\)](http://php.net/manual/en/function.call-user-func.php) 和 [call_user_func_array\(\)](http://php.net/manual/en/function.call-user-func-array.php)代替。你也可以考虑使用 [可变函数](http://php.net/manual/en/functions.variable-functions.php)或者其他的选择。

#### [mcrypt](http://php.net/manual/en/book.mcrypt.php) 相关的
移除已废弃的[mcrypt_generic_end\(\)](http://php.net/manual/zh/function.mcrypt-generic-end.php) 函数，请使用 [mcrypt_generic_deinit\(\)](http://php.net/manual/zh/function.mcrypt-generic-deinit.php) 。
此外，已废弃的[mcrypt_ecb\(\)](http://php.net/manual/zh/function.mcrypt-ecb.php)，[mcrypt_cbc\(\)](http://php.net/manual/zh/function.mcrypt-cbc.php)，[mcrypt_cfb\(\)](http://php.net/manual/zh/function.mcrypt-cfb.php)和[mcrypt_ofb\(\)](http://php.net/manual/zh/function.mcrypt-ofb.php)函数被彻底删除，请使用[mcrypt_decrypt()](http://php.net/manual/zh/function.mcrypt-decrypt.php)与适当的[MCRYPT_MODE_*]()常量来代替。

#### [intl](http://php.net/manual/en/book.intl.php) 相关的
[datefmt_set_timezone_id\(\)](http://php.net/manual/zh/intldateformatter.settimezoneid.php)与[IntlDateFormatter::setTimeZoneID\(\)](http://php.net/manual/zh/intldateformatter.settimezoneid.php)被删除，分别使用[datefmt_set_timezone\(\)](http://php.net/manual/zh/intldateformatter.settimezone.php)与[IntlDateFormatter::setTimeZone\(\)](http://php.net/manual/zh/intldateformatter.settimezone.php)。

#### [set_magic_quotes_runtime\(\)](http://php.net/manual/en/function.set-magic-quotes-runtime.php)
[set_magic_quotes_runtime\(\)](http://php.net/manual/zh/function.set-magic-quotes-runtime.php)与它的别名函数[magic_quotes_runtime\(\)](http://php.net/manual/zh/function.magic-quotes-runtime.php)都在PHP7中删除了。他们在PHP5.3.0中就①被废弃，并且在[PHP5.4.0中移除](http://php.net/manual/zh/migration54.incompatible.php)的魔术引号下是无效的。

#### [set_socket_blocking\(\)](http://php.net/manual/en/function.set-socket-blocking.php)
[stream_set_blocking\(\)](http://php.net/manual/zh/function.stream-set-blocking.php)的别名函数[set_socket_blocking\(\)](http://php.net/manual/zh/function.set-socket-blocking.php)已被移除。

#### [dl\(\)](http://php.net/manual/en/function.dl.php) 在PHP-FPM中
[dl\(\)](http://php.net/manual/zh/function.dl.php)函数不能在PHP-FPM中使用了，它的功能仍然在CLI、嵌入SAPIs中保留。

#### [GD](http://php.net/manual/en/book.image.php) Type1 函数
PostScript Type1字体的支持已经从GD扩展删除，涉及的删除的函数有：
* [imagepsbbox()](http://php.net/manual/zh/function.imagepsbbox.php)
* [imagepsencodefont()](http://php.net/manual/zh/function.imagepsencodefont.php)
* [imagepsextendfont()](http://php.net/manual/zh/function.imagepsextendfont.php)
* [imagepsfreefont()](http://php.net/manual/zh/function.imagepsfreefont.php)
* [imagepsloadfont()](http://php.net/manual/zh/function.imagepsloadfont.php)
* [imagepsslantfont()](http://php.net/manual/zh/function.imagepsslantfont.php)
* [imagepstext()](http://php.net/manual/zh/function.imagepstext.php)

建议使用TrueType字体和其相关的函数代替。

### 移除的INI配置
#### 删除的特性
下面的INI指令以及相关的特性被删除：
* [always_populate_raw_post_data](http://php.net/manual/en/ini.core.php#ini.always-populate-raw-post-data)
* [asp_tags](http://php.net/manual/en/ini.core.php#ini.asp-tags)

#### xsl.security_prefs
xsl.security_prefs指令已被删除。相反，该[xsltprocessor::setsecurityprefs\(\)](http://php.net/manual/en/xsltprocessor.setsecurityprefs.php)方法用于控制每个处理器的安全性偏好。

### 其他向后不兼容的变更
#### 不能赋值引用的**New**对象
[New](http://php.net/manual/en/language.oop5.basic.php#language.oop5.basic.new)语句的结果不再能通过引用赋值给一个变量，如下代码：
```PHP
<?php
class C {}
$c =& new C;
?>
```
PHP5中的输出：
```PHP
Deprecated: Assigning the return value of new by reference is deprecated in /tmp/test.php on line 3
```
PHP7中的输出：
```PHP
Parse error: syntax error, unexpected 'new' (T_NEW) in /tmp/test.php on line 3
```

#### 无效的类、接口和trait名
下面的名称不能被用来类、接口、trait的名称：
* [bool](http://php.net/manual/zh/language.types.boolean.php)
* [int](http://php.net/manual/zh/language.types.integer.php)
* [float](http://php.net/manual/zh/language.types.float.php)
* [string](http://php.net/manual/zh/language.types.string.php)
* **NULL**
* **TRUE**
* **FALSE**

此外，下列名称不应该被使用。虽然他们不会在PHP 7中发生错误，他们是保留供将来使用，应认为已过时。
* [resource](http://php.net/manual/zh/language.types.resource.php)
* [object](http://php.net/manual/zh/language.types.object.php)
* [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed)
* numeric

#### ASP语法标记、Script PHP语法标记被移除
使用ASP脚本标签，或者Script标签定界的PHP代码，已被删除。受影响的标签是：
![image](https://cloud.githubusercontent.com/assets/1308846/9438212/bdeec078-4a8e-11e5-91b5-5e6b92e4019d.png)

#### 禁止从不兼容的上下文调用
[之前PHP5.6已废止的特性中](http://php.net/manual/en/migration56.deprecated.php#migration56.deprecated.incompatible-context)，静态调用上下文不一致的非静态方法将会导致调用中的$this 变量未定义，并引发废止警告。
```PHP
<?php
class A {
    public function test() { var_dump($this); }
}

// Note: Does NOT extend A
class B {
    public function callNonStaticMethodOfA() { A::test(); }
}

(new B)->callNonStaticMethodOfA();
?>
```
在PHP5中会输出：
```PHP
Deprecated: Non-static method A::test() should not be called statically, assuming $this from incompatible context in /tmp/test.php on line 8
object(B)#1 (0) {
}
```
在PHP7中会输出：
```PHP
Deprecated: Non-static method A::test() should not be called statically in /tmp/test.php on line 8

Notice: Undefined variable: this in /tmp/test.php on line 3
NULL
```
#### [yield](http://php.net/manual/zh/language.generators.syntax.php#control-structures.yield) 现在开始作为右结合运算符
yield 不再需要括号，可以作为一个右结合运算符，优先级别介于 **print** 与 ** => **之间，这可能会导致行为的改变：
```PHP
<?php
echo yield -1;
// Was previously interpreted as
echo (yield) - 1;
// And is now interpreted as
echo yield (-1);

yield $foo or die;
// Was previously interpreted as
yield ($foo or die);
// And is now interpreted as
(yield $foo) or die;
?>
```
可以用括号来消除歧义的情况。

#### 函数不能有相同名称的参数
不允许在函数中定义相同名称的参数。例如下列代码，将会触发 **E_COMPILE_ERROR** 。
```PHP
<?php
function foo($a, $b, $unused, $unused) {
    //
}
?>
```

#### [$HTTP_RAW_POST_DATA](http://php.net/manual/en/reserved.variables.httprawpostdata.php) 被移除
[$HTTP_RAW_POST_DATA](http://php.net/manual/zh/reserved.variables.httprawpostdata.php) 不再被支持。 应使用 [php://input](http://php.net/manual/zh/wrappers.php.php#wrappers.php.input) 流数据来代替。

#### \# 注释已被移除
INI文件中以\#符号为前缀的注释支持已被移除，**;**符号将代替**\#**，此更改适用于PHP.ini文件，以及由[parse_ini_file()](http://php.net/manual/zh/function.parse-ini-file.php)和[parse_ini_string()](http://php.net/manual/zh/function.parse-ini-string.php)处理的文件。

## 用户贡献说明
暂无
