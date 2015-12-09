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
<table><tbody><tr><th bgcolor='#C4C9DF'>场景</th>
<th bgcolor='#C4C9DF'>新的级别/行为</th>
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

<table><tbody><tr><th bgcolor='#C4C9DF'>表达式</th>
<th bgcolor='#C4C9DF'>PHP 5 解析器</th>
<th bgcolor='#C4C9DF'>PHP 7 解析器</th>
</tr>
</tbody>
<tbody><tr><td>$$foo['bar']['baz']</td>
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
[list\(\)](http://php.net/manual/en/function.list.php) 不再允许拆解[字符串](http://php.net/manual/en/language.types.string.php)变量，而应该使用[str_split](http://php.net/manual/en/function.str-split.php)函数。

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
> array(2) {
>   ["b"]=>
>   &int(1)
>   ["a"]=>
>   &int(1)
> }

PHP7中的输出
> array(2) {
>   ["a"]=>
>   &int(1)
>   ["b"]=>
>   &int(1)
> }


#### [global](http://php.net/manual/en/language.variables.scope.php#language.variables.scope.global) 仅支持简单的变量
[可变变量](http://php.net/manual/zh/language.variables.variable.php)将不能再使用[global](http://php.net/manual/zh/language.variables.scope.php#language.variables.scope.global)标记。如果真的需要，可以用花括号来间隔开写，例如下面代码：
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
作为一个基本原则，不提倡这样的变量套变量和[global](http://php.net/manual/zh/language.variables.scope.php#language.variables.scope.global)标记的使用方式。

#### 围绕函数参数中的括号不再影响的变更
在PHP5中，通过引用传递的函数参数使用多余的括号包围函数时，可以不出现严格标准警告。但在PHP7开始，都会发出该警告。
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
> Notice: Only variables should be passed by reference in /tmp/test.php on line 13


### [foreach](http://php.net/manual/zh/control-structures.foreach.php) 的变化
对 [foreach](http://php.net/manual/zh/control-structures.foreach.php) 控制结构的行为作了细微的更改，主要是围绕数组遍历时的内部数组指针和迭代时修改数组的处理。

#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 遍历期间不再修改内部数组指针
在PHP7之前，当数组通过[foreach](http://php.net/manual/zh/control-structures.foreach.php)迭代时，数组指针会移动。现在开始，不再如此，见下面代码：
```PHP
<?php
$array = [0, 1, 2];
foreach ($array as &$val) {
    var_dump(current($array));
}
?>
```
PHP5中的输出
> int(1)
> int(2)
> bool(false)

PHP7中的输出
> int(0)
> int(0)
> int(0)


#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 通过值遍历时，操作的值为数组的副本
当使用默认的通过值遍历数组时，[foreach](http://php.net/manual/zh/control-structures.foreach.php)实际操作的是数组的迭代副本，而非数组本身。这就意味着，在迭代中数组的变化不会修改原数组的值。

#### [foreach](http://php.net/manual/en/control-structures.foreach.php) 通过引用遍历时，有更好的迭代特性
当使用引用遍历数组时，现在[foreach](http://php.net/manual/zh/control-structures.foreach.php)在迭代中更好的跟踪数组变化。例如，在迭代中添加一个值到数组中也会迭代添加的值，例如下面代码：
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
> int(0)

在PHP7的输出为：
> int(0)
> int(1)

#### [non-Traversable](http://php.net/manual/en/class.traversable.php) 对象的遍历
迭代一个[非Traversable](http://php.net/manual/en/class.traversable.php)对象将会与迭代一个引用数组的行为相同。 这将导致在对象添加或删除属性时，如同[foreach 通过引用遍历](http://php.net/manual/en/migration70.incompatible.php#migration70.incompatible.foreach.by-ref)时一样，有更好的迭代特性。

### [整数](http://php.net/manual/zh/language.types.integer.php)处理上的变更
#### 无效的八进制文本
此前，包含无效数字的八进制文本会自动被截断（0128被截为012）。现在，将会引发解析错误。

#### 负位移
负数的位移将抛出一个 [ArithmeticError](http://php.net/manual/en/class.arithmeticerror.php) 
```PHP
<?php
var_dump(1 >> -1);
?>
```
PHP5中的输出：
> int(0)

PHP7中的输出：
> Fatal error: Uncaught ArithmeticError: Bit shift by negative number in /tmp/test.php:2
> Stack trace:
> \#0 {main}
>   thrown in /tmp/test.php on line 2

#### 超出范围的位移
位移（任一方向）超出一个[整数](http://php.net/manual/zh/language.types.integer.php)的位宽度会得到0。以前，这种位移的处理是依赖运行环境所在的硬件结构。

#### 除数是0时的变更
以前，当0被用作 除（/）或取模（％）操作符的除数时，会引发**E\_WARNING**警告并返回 **false** 。现在，除法操作符返回 浮点数+INF，-INF，或NAN。取模操作符不会引发**E\_WARNING**警告，而是抛出 **DivisionByZeroError** 异常。
```PHP
<?php
var_dump(3/0);
var_dump(0/0);
var_dump(0%0);
?>
```

> 在php5中的输出：
> Warning: Division by zero in %s on line %d
> bool(false)
> 
> Warning: Division by zero in %s on line %d
> bool(false)
> 
> Warning: Division by zero in %s on line %d
> bool(false)

在php7中的输出：
> Warning: Division by zero in %s on line %d
> float(INF)
> 
> Warning: Division by zero in %s on line %d
> float(NAN)
> 
> PHP Fatal error:  Uncaught DivisionByZeroError: Modulo by zero in %s line %d

### [string](http://php.net/manual/zh/language.types.string.php)处理上的调整
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
> bool(true)
> bool(true)
> int(15)
> string(2) "oo"

在PHP7中的输出：
> bool(false)
> bool(false)
> int(0)
> 
> Notice: A non well formed numeric value encountered in /tmp/test.php on line 5
> string(3) "foo"

[filter_var()](http://php.net/manual/en/function.filter-var.php) 函数可以用于检查一个 [string](http://php.net/manual/zh/language.types.string.php) 是否含有十六进制数字,并将其转换为[integer](http://php.net/manual/zh/language.types.integer.php):

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

#### \u{ 可能引起错误
由于新的[Unicode codepoint 转译语法](http://php.net/manual/zh/migration70.new-features.php#migration70.new-features.unicode-codepoint-escape-syntax)， 紧连着无效序列并包含\u{ 的字串可能引起致命错误。 为了避免这一报错，应该转义开头的反斜杠。

### 被移除的函数
#### [call_user_method\(\)](http://php.net/manual/en/function.call-user-method.php) 与 [call_user_method_array\(\)](http://php.net/manual/en/function.call-user-method-array.php)
这些函数从PHP4.1.0开始因为新增的 [call_user_func\(\)](http://php.net/manual/zh/function.call-user-func.php) 和 [call_user_func_array\(\)](http://php.net/manual/zh/function.call-user-func-array.php)被废弃。你也可以考虑使用 [可变函数](http://php.net/manual/zh/functions.variable-functions.php)或者[...](http://php.net/manual/zh/functions.arguments.php#functions.variable-arg-list.new) 操作符。

#### [mcrypt](http://php.net/manual/en/book.mcrypt.php) 别名
已废弃的 [mcrypt_generic_end()](http://php.net/manual/zh/function.mcrypt-generic-end.php) 函数已被移除，请使用[mcrypt_generic_deinit()](http://php.net/manual/zh/function.mcrypt-generic-deinit.php)代替。

此外，已废弃的 [mcrypt_ecb()](http://php.net/manual/zh/function.mcrypt-ecb.php), [mcrypt_cbc()](http://php.net/manual/zh/function.mcrypt-cbc.php), [mcrypt_cfb()](http://php.net/manual/zh/function.mcrypt-cfb.php) 和 [mcrypt_ofb()](http://php.net/manual/zh/function.mcrypt-ofb.php) 函数已被移除，请使用[mcrypt_decrypt()](http://php.net/manual/zh/function.mcrypt-decrypt.php)配合恰当的**MCRYPT\_MODE\_*** 常量来代替。

#### [intl](http://php.net/manual/en/book.intl.php) 别名
已废弃的 [datefmt_set_timezone_id()](http://php.net/manual/zh/intldateformatter.settimezoneid.php) 和 [IntlDateFormatter::setTimeZoneID()](http://php.net/manual/zh/intldateformatter.settimezoneid.php) 函数已被移除，请使用 [datefmt_set_timezone()](http://php.net/manual/zh/intldateformatter.settimezone.php) 与 [IntlDateFormatter::setTimeZone()](http://php.net/manual/zh/intldateformatter.settimezone.php)代替。

#### [set_magic_quotes_runtime\(\)](http://php.net/manual/en/function.set-magic-quotes-runtime.php)
[set_magic_quotes_runtime\(\)](http://php.net/manual/zh/function.set-magic-quotes-runtime.php)与它的别名函数[magic_quotes_runtime\(\)](http://php.net/manual/zh/function.magic-quotes-runtime.php)都在PHP7中删除了。他们在PHP5.3.0中已经被废弃，并且在[PHP5.4.0中移除](http://php.net/manual/zh/migration54.incompatible.php)也由于魔术引号的废弃而失去功能。

#### [set_socket_blocking\(\)](http://php.net/manual/en/function.set-socket-blocking.php)
[stream_set_blocking\(\)](http://php.net/manual/zh/function.stream-set-blocking.php)的别名函数[set_socket_blocking\(\)](http://php.net/manual/zh/function.set-socket-blocking.php)已被移除。

#### [dl\(\)](http://php.net/manual/en/function.dl.php) 在PHP-FPM中
[dl()](http://php.net/manual/zh/function.dl.php)在 PHP-FPM 不再可用，在 CLI 和 embed SAPIs 中仍可用。

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
* [always_populate_raw_post_data](http://php.net/manual/zh/ini.core.php#ini.always-populate-raw-post-data)
* [asp_tags](http://php.net/manual/zh/ini.core.php#ini.asp-tags)

#### xsl.security_prefs
xsl.security_prefs指令已被删除。相反，该[xsltprocessor::setsecurityprefs\(\)](http://php.net/manual/en/xsltprocessor.setsecurityprefs.php)方法用于控制每个处理器的安全性偏好。

### 其他向后兼容相关的变更
#### new 操作符创建的对象不能以引用方式赋值给变量
[new](http://php.net/manual/zh/language.oop5.basic.php#language.oop5.basic.new) 语句创建的对象不能 以引用的方式赋值给变量。

```PHP
<?php
class C {}
$c =& new C;
?>
```
PHP5中的输出：
> Deprecated: Assigning the return value of new by reference is deprecated in /tmp/test.php on line 3

PHP7中的输出：
> Parse error: syntax error, unexpected 'new' (T_NEW) in /tmp/test.php on line 3

#### 无效的类、接口以及 trait 命名
不能以下列名字来命名类、接口以及 trait：
* [bool](http://php.net/manual/zh/language.types.boolean.php)
* [int](http://php.net/manual/zh/language.types.integer.php)
* [float](http://php.net/manual/zh/language.types.float.php)
* [string](http://php.net/manual/zh/language.types.string.php)
* **NULL**
* **TRUE**
* **FALSE**

此外，也不要使用下列的名字来命名类、接口以及 trait。虽然在 PHP 7.0 中， 这并不会引发错误， 但是这些名字是保留给将来使用的。
* [resource](http://php.net/manual/zh/language.types.resource.php)
* [object](http://php.net/manual/zh/language.types.object.php)
* [mixed](http://php.net/manual/zh/language.pseudo-types.php#language.types.mixed)
* numeric

#### 移除了 ASP 和 script PHP 标签
使用类似 ASP 的标签，以及 script 标签来区分 PHP 代码的方式被移除。 受到影响的标签有：
<table>
    <caption>
        <span style="font-weight: bolder;">被移除的 ASP 和 script 标签</span>
    </caption>
    <thead>
        <tr>
            <th bgcolor='#C4C9DF'>开标签</th>
            <th bgcolor='#C4C9DF'>闭标签</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td >
                <code class="code">&lt;%</code>
            </td>
            <td>
                <code class="code">%&gt;</code>
            </td>
        </tr>
        <tr>
            <td>
                <code class="code">&lt;%=</code>
            </td>
            <td>
                <code class="code">%&gt;</code>
            </td>
        </tr>
        <tr>
            <td>
                <code class="code">&lt;script language=&quot;php&quot;&gt;</code>
            </td>
            <td>
                <code class="code">&lt;/script&gt;</code>
            </td>
        </tr>
    </tbody>
</table>

#### 移除从不匹配的上下文发起调用
在不匹配的上下文中以静态方式调用非静态方法， [在 PHP 5.6 中已经废弃](http://php.net/manual/zh/migration56.deprecated.php#migration56.deprecated.incompatible-context)， 但是在 PHP 7.0 中， 会导致被调用方法中未定义 $this 变量，以及此行为已经废弃的警告。
```PHP
<?php
class A {
    public function test() { var_dump($this); }
}

// 注意：并没有从类 A 继承
class B {
    public function callNonStaticMethodOfA() { A::test(); }
}

(new B)->callNonStaticMethodOfA();
?>
```
在PHP5中会输出：
> Deprecated: Non-static method A::test() should not be called statically, assuming $this from incompatible context in /tmp/test.php on line 8
> object(B)#1 (0) {
> }

在PHP7中会输出：
> Deprecated: Non-static method A::test() should not be called statically in /tmp/test.php on line 8
> 
> Notice: Undefined variable: this in /tmp/test.php on line 3
> NULL

#### [yield](http://php.net/manual/zh/language.generators.syntax.php#control-structures.yield) 变更为右联接运算符
在使用 [yield](http://php.net/manual/zh/language.generators.syntax.php#control-structures.yield) 关键字的时候，不再需要括号， 并且它变更为右联接操作符，其运算符优先级介于 print 和 => 之间。 这可能导致现有代码的行为发生改变：

```PHP
<?php
echo yield -1;
// 在之前版本中会被解释为：
echo (yield) - 1;
// 现在，它将被解释为：
echo yield (-1);

yield $foo or die;
// 在之前版本中会被解释为：
yield ($foo or die);
// 现在，它将被解释为：
(yield $foo) or die;
?>
```
可以通过使用括号来消除歧义。

#### 函数定义不可以包含多个同名参数
在函数定义中，不可以包含两个或多个同名的参数。 例如，下面代码中的函数定义会触发 **E\_COMPILE\_ERROR** 错误：

```PHP
<?php
function foo($a, $b, $unused, $unused) {
    //
}
?>
```

#### Switch 语句不可以包含多个 default 块
在 switch 语句中，两个或者多个 default 块的代码已经不再被支持。 例如，下面代码中的 switch 语句会触发 **E\_COMPILE\_ERROR** 错误：
```PHP
<?php
switch (1) {
    default:
    break;
    default:
    break;
}
?>
```

#### [$HTTP_RAW_POST_DATA](http://php.net/manual/zh/reserved.variables.httprawpostdata.php) 被移除
不再提供 [$HTTP_RAW_POST_DATA](http://php.net/manual/zh/reserved.variables.httprawpostdata.php) 变量。 请使用 [*php://input*](http://php.net/manual/zh/wrappers.php.php#wrappers.php.input) 作为替代。

#### INI 文件中 \# 注释格式被移除
在 INI 文件中，不再支持以 \# 开始的注释行， 请使用 ;（分号）来表示注释。 此变更适用于 php.ini 以及用 [parse_ini_file()](http://php.net/manual/zh/function.parse-ini-file.php) 和 [parse_ini_string()](http://php.net/manual/zh/function.parse-ini-string.php) 函数来处理的文件。

#### JSON 扩展已经被 JSOND 取代
JSON 扩展已经被 JSOND 扩展取代。 对于数值的处理，有以下两点需要注意的： 第一，数值不能以点号（.）结束 （例如，数值 34. 必须写作 34.0 或 34）。 第二，如果使用科学计数法表示数值，e 前面必须不是点号（.） （例如，3.e3 必须写作 3.0e3 或 3e3）。

#### 在数值溢出的时候，内部函数将会失败
将浮点数转换为整数的时候，如果浮点数值太大，导致无法以整数表达的情况下， 在之前的版本中，内部函数会直接将整数截断，并不会引发错误。 在 PHP 7.0 中，如果发生这种情况，会引发 E_WARNING 错误，并且返回 **NULL**。

#### 自定义会话处理器的返回值修复
在自定义会话处理器中，如果函数的返回值不是 **FALSE**，也不是 -1， 会引发致命错误。现在，如果这些函数的返回值不是布尔值，也不是 -1 或者 0，函数调用结果将被视为失败，并且引发 E_WARNING 错误。