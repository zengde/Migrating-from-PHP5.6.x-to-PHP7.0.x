# PHP5.6.x版本迁移至7.0.x版本
## 新特性
### 标量类型声明
标量类型声明有两种模式：强制（默认）模式和严格模式。现在可以使用下列类型的参数（无论用强制模式还是严格模式）：字符串（[string](http://php.net/manual/en/language.types.string.php)）、整数（int）、浮点数（[float](http://php.net/manual/en/language.types.float.php)）和布尔值（bool）。它们扩充了PHP5中引入的其他类型：类名、接口、[数组](http://php.net/manual/en/language.types.array.php)和[回调类型](http://php.net/manual/en/language.types.callable.php)。

```PHP
<?php
// Coercive mode
function sumOfInts(int ...$ints)
{
    return array_sum($ints);
}

var_dump(sumOfInts(2, '3', 4.1));
```

上述例子输出：
```PHP
int(9)
```

要使用严格模式， 必须在文件顶部声明一个[declare](http://php.net/manual/en/control-structures.declare.php)指令，这意味着严格声明标量是基于文件可配的。这个指令不仅影响参数的类型声明，也影响到函数的返回值声明（参见[返回值类型声明](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)，内置的php函数以及扩展中加载的php函数）。

完整的标量类型声明文档和示例参见[类型声明](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)章节。

### 返回值类型声明
PHP7新增了[返回类型声明](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)，类似于[参数类型声明](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration)，返回类型声明指明了函数返回值的类型。可用的声明类型与参数声明中可用的[类型](http://php.net/manual/en/functions.arguments.php#functions.arguments.type-declaration.types)相同。
```PHP
<?php

function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}

print_r(arraysSum([1,2,3], [4,5,6], [7,8,9]));
```
上述代码返回值为:
```PHP
Array
(
    [0] => 6
    [1] => 15
    [2] => 24
)
```
完整的返回值声明文档和示例代码可以参见[返回值类型声明](http://php.net/manual/en/functions.returning-values.php#functions.returning-values.type-declaration)。

### NULL合并运算符（“??”）
由于日常使用中存在大量需要使用三元表达式与[isset()](http://php.net/manual/en/function.isset.php)结合的情况，php7新增了一个语法糖：NULL合并运算符（“??”）。如果变量存在且不为**NULL**，那么返回第一个操作数的值，否则，返回第二个擦作数的值。
```PHP
<?php
// Fetches the value of $_GET['user'] and returns 'nobody'
// if it does not exist.
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';

// Coalesces can be chained: this will return the first
// defined value out of $_GET['user'], $_POST['user'], and
// 'nobody'.
$username = $_GET['user'] ?? $_POST['user'] ?? 'nobody';
?>
```
### 结合比较运算符 (<=>)
结合比较运算符用于比较两个表达式。当$a小于、等于或大于$b时分别返回一个大于0、等于0、小于0的整数。比较的原则是根据PHP的[类型比较规则](http://php.net/manual/en/types.comparisons.php)执行的。
```PHP
<?php
// Integers
echo 1 <=> 1; // 0
echo 1 <=> 2; // -1
echo 2 <=> 1; // 1

// Floats
echo 1.5 <=> 1.5; // 0
echo 1.5 <=> 2.5; // -1
echo 2.5 <=> 1.5; // 1
 
// Strings
echo "a" <=> "a"; // 0
echo "a" <=> "b"; // -1
echo "b" <=> "a"; // 1
?>
```
### 可以使用[define()](http://php.net/manual/en/function.define.php)来定义常量数组
[Array](http://php.net/manual/en/language.types.array.php)类型的常量可以通过[define()](http://php.net/manual/en/function.define.php)来定义了。在PHP5.6中仅能通过const定义。
```PHP
<?php
define('ANIMALS', [
    'dog',
    'cat',
    'bird'
]);

echo ANIMALS[1]; // outputs "cat"
?>
```

### 匿名类
支持通过new class添加一个匿名类。匿名类可以用在完整的类中定义的一次性的对象。
```PHP
<?php
interface Logger {
    public function log(string $msg);
}

class Application {
    private $logger;

    public function getLogger(): Logger {
         return $this->logger;
    }

    public function setLogger(Logger $logger) {
         $this->logger = $logger;
    }
}

$app = new Application;
$app->setLogger(new class implements Logger {
    public function log(string $msg) {
        echo $msg;
    }
});

var_dump($app->getLogger());
?>
```
上面代码输出：
```PHP
object(class@anonymous)#2 (0) {
}
```
详细文档可以参见[匿名类参考](http://php.net/manual/en/language.oop5.anonymous.php)

### Unicode 代码点转义语法
通过十六进制形式的*代码点*与双引号或heredoc组成的字符串生成UTF-8代码点，可以接受任何有效的*代码点*，并且开头的0是可以省略的。
```PHP
echo "\u{aa}";
echo "\u{0000aa}";
echo "\u{9999}";
```
上面代码输出：
```PHP
ª
ª (same as before but with optional leading 0's)
香
```

### [Closure::call()](http://php.net/manual/en/closure.call.php)
[Closure::call()](http://php.net/manual/en/closure.call.php)有着更好的性能，简便的暂时绑定一个对象到闭包并调用它。
```PHP
<?php
class A {private $x = 1;}

// Pre PHP 7 code
$getXCB = function() {return $this->x;};
$getX = $getXCB->bindTo(new A, 'A'); // intermediate closure
echo $getX();

// PHP 7+ code
$getX = function() {return $this->x;};
echo $getX->call(new A);
```
上述代码输出：
```PHP
1
1
```

### 过滤的 unserialize()
这个特性意在反序列化不可靠的数据时提供更好的安全性。通过白名单类的方式来防止代码注入。
```PHP
<?php

// converts all objects into __PHP_Incomplete_Class object
$data = unserialize($foo, ["allowed_classes" => false]);

// converts all objects into __PHP_Incomplete_Class object except those of MyClass and MyClass2
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);

// default behaviour (same as omitting the second argument) that accepts all classes
$data = unserialize($foo, ["allowed_classes" => true]);
```

### [IntlChar](http://php.net/manual/en/class.intlchar.php)
新增加的IntlChar类意在于暴露出更多的ICU功能。类自身定义了许多静态方法和常理用于操作unicode字符。
```PHP
<?php

printf('%x', IntlChar::CODEPOINT_MAX);
echo IntlChar::charName('@');
var_dump(IntlChar::ispunct('!'));
```
上述代码输出：
```PHP
10ffff
COMMERCIAL AT
bool(true)
```
若要使用此类，请先安装[Intl](http://php.net/manual/en/book.intl.php)扩展。

### 预期
预期是向后兼容以增强旧的assert()方法。在生产代码中启用断言为零成本，并且提供抛出特定异常的能力。<br>
在使用老版本API时，如果第一个参数是一个字符串,那么它将被评估。第二个参数可以是一个简单的字符串(导致AssertionError被触发)，或一个包含一个错误消息的自定义异常对象。
```PHP
<?php

ini_set('assert.exception', 1);

class CustomError extends AssertionError {}

assert(false, new CustomError('Some error message'));
```
上述代码输出：
```PHP
Fatal error: Uncaught CustomError: Some error message
```
这个特性会带来两个PHP.ini设置(以及它们的默认值): 
* [zend.assertions](http://php.net/manual/en/ini.core.php#ini.zend.assertions) = 1
* [assert.exception](http://php.net/manual/en/info.configuration.php#ini.assert.exception) = 0

zend.assertions有三种值：
* 1 = 生成并且执行代码（开发模式）
* 0 = 执行代码并且在运行期间跳跃
* -1 = 不生成任何代码 (0开销, 生产模式)
assert.exception意味着断言失败时抛出异常。默认关闭保持兼容旧的assert()函数。 

### 使用use成组声明
现在可以根据父命名空间成组声明，此举可以消除导入多个相同命名空间的类，函数或者常量时的代码冗长。
```PHP
<?php

// Pre PHP 7 code
use some\namespace\ClassA;
use some\namespace\ClassB;
use some\namespace\ClassC as C;

use function some\namespace\fn_a;
use function some\namespace\fn_b;
use function some\namespace\fn_c;

use const some\namespace\ConstA;
use const some\namespace\ConstB;
use const some\namespace\ConstC;

// PHP 7+ code
use some\namespace\{ClassA, ClassB, ClassC as C};
use function some\namespace\{fn_a, fn_b, fn_c};
use const some\namespace\{ConstA, ConstB, ConstC};
```

### 生成器返回表达式
此特性是建立在PHP5.5中引入的生成器功能之上，使得表达式最终返回的值为发生器内使用的return语句（返回引用是不允许的）。可以通过新的Generator::getReturn() 方法获得返回值, 此方法只能在生成器完成yiel值时使用。
```PHP
<?php

$gen = (function() {
    yield 1;
    yield 2;

    return 3;
})();

foreach ($gen as $val) {
    echo $val, PHP_EOL;
}

echo $gen->getReturn(), PHP_EOL;
```
上述代码输出：
```PHP
1
2
3
```
能够从发生器显示地返回一个最终值有一个很方便的地方，可以通过发生器（也许是某种协程计算）最终返回的值来决定客户端代码如何执行。这比强制客户端代码首先检查最终值是否已经产生了，如果是，再专门负责处理该值简单多了。

### 生成器委派
生成器委派建立在上面说到的生成器返回表达式上，它通过使用*yield from &lt;expr>*的新语法，其中&lt;expr>可以是任何Traversable（可遍历）的对象或数组，&lt;expr>将被迭代直到不再有效，然后将继续向下执行生成器。此功能使yield语句被分解成更小的操作，从而具有更高的可重用性的清洁代码。
```PHP
<?php

function gen()
{
    yield 1;
    yield 2;

    return yield from gen2();
}

function gen2()
{
    yield 3;

    return 4;
}

$gen = gen();

foreach ($gen as $val)
{
    echo $val, PHP_EOL;
}

echo $gen->getReturn();
```
上述代码输出：
```PHP
1
2
3
4
```

### 通过[intdiv()](http://php.net/manual/en/function.intdiv.php)做整除
[intdiv()](http://php.net/manual/en/function.intdiv.php)函数来处理需要返回整数的除法。
```PHP
<?php

var_dump(intdiv(10, 3));
```
上述代码输出：
```PHP
int(3)
```

### [session_start()](http://php.net/manual/en/function.session-start.php) 选项
该特性使session_start()函数接受传入的选项数组，当然这些设置可以在PHP.ini中设置。
```PHP
<?php

session_start(['cache_limiter' => 'private']); // sets the session.cache_limiter option to private
```
这个特性还引入了一个新的php.ini设置(session.lazy_write)，默认情况下为true，表示会话数据仅在变化时重写。

### [preg_replace_callback_array()](http://php.net/manual/en/function.preg-replace-callback-array.php) 函数
这一新功能当使用[preg_replace_callback（）](http://php.net/manual/en/function.preg-replace-callback.php)函数时代码编写更简洁。在PHP7之前，回调需要执行每个正则表达式所需的回调函数（[preg_replace_callback()](http://php.net/manual/en/function.preg-replace-callback.php)的第二个参数）从而会产生很多分支的污染。<br>
现在，可以通过关联数组为每个正则表达式分别注册回调函数，只需将正则表达式作为key，回调函数作为value。
```PHP
<?php

$tokenStream = []; // [tokenName, lexeme] pairs

$input = <<<'end'
$a = 3; // variable initialisation
end;

// Pre PHP 7 code
preg_replace_callback(
    [
        '~\$[a-z_][a-z\d_]*~i',
        '~=~',
        '~[\d]+~',
        '~;~',
        '~//.*~'
    ],
    function ($match) use (&$tokenStream) {
        if (strpos($match[0], '$') === 0) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        } elseif (strpos($match[0], '=') === 0) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        } elseif (ctype_digit($match[0])) {
            $tokenStream[] = ['T_NUM', $match[0]];
        } elseif (strpos($match[0], ';') === 0) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        } elseif (strpos($match[0], '//') === 0) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    },
    $input
);

// PHP 7+ code
preg_replace_callback_array(
    [
        '~\$[a-z_][a-z\d_]*~i' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_VARIABLE', $match[0]];
        },
        '~=~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_ASSIGN', $match[0]];
        },
        '~[\d]+~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_NUM', $match[0]];
        },
        '~;~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_TERMINATE_STMT', $match[0]];
        },
        '~//.*~' => function ($match) use (&$tokenStream) {
            $tokenStream[] = ['T_COMMENT', $match[0]];
        }
    ],
    $input
);
```

### [CSPRNG](http://php.net/manual/en/book.csprng.php) 系列函数
该特性引入两个新的函数，用于生成加密安全的整形与字符串。它提供了简单的API和平台无关性。

函数声明：
```PHP
string random_bytes(int length);
int random_int(int min, int max);
```
两个函数在没有找到足够的随机性源时会报**E_WARNING**错误并且返回false。

### [list()](http://php.net/manual/en/function.list.php)可以拆解实现了[ArrayAccess](http://php.net/manual/en/class.arrayaccess.php)接口的对象

## 用户贡献记录
暂无

