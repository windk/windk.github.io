Date: 2013-07-04 16:00:27
Status: public
Tags: PHP 数组 运算符

原文地址:[Array Operators in PHP: Interesting but Less Spoken](http://phpmaster.com/array-operators-in-php-interesting-but-less-spoken/)

PHP 的操作符 可以分为7种不同类型， 算术(arithmetic)， 赋值(assignment)，位运算(bitwise)， 比较(comparison)， 错误控制(error control)， 执行(execution)， 递增／递减(incrementing/decrementing)， 逻辑(logical)， 字符串(string)， 数组(array)，和类型操作符(type operators)， 这篇文章着重讲数组运算符号, 不过也稍微提了一下使用数组的时候可以使用的运算符。

## 数组操作符
官方文档仅仅提供了一个简短的数组的操作符的描述，有时候会对产生歧义， 我们先看看各个运算符究竟干了什么，这些运算符都是2元的，就是说一个操作符操作两个数组。

### 数组联合
首先是联合运算符`＋`， 基于数组的key，来计算两个数组联合的结果。 他基于key的松散匹配，如果第一个数组里面已经存在这个key，则直接忽略第二个数组中的这个key。 第二个数组的剩下的key以及key相应的值会被附加到第一个的后面。

```
<?php
$array1 = array('a', 'b', 'c');
$array2 = array('d', 'e', 'f', 'g', 'h', 'i');
print_r($array1 + $array2);
print_r($array2 + $array1);
```
```
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => g
    [4] => h
    [5] => i
)
```
```    
Array
(
    [0] => d
    [1] => e
    [2] => f
    [3] => g
    [4] => h
    [5] => i
)
```
在第一个`print_r()`中， `$array2` 的前三个元素的key 已经存在了`$array1`中， 所以'd', 'e', 'f' 就没有出现在结果集中。
而在第二个`print_r()`中，`$array1` 中的所有的key都出现在了`$array2`中，所以`$array1`中的元素都被忽略了。
基于key的松散匹配得出的结果可能和你想象得完全不一样， 不过也提供了优化代码和开放性代码的可能(这里还是参考原文看看，翻译得不好)。
```
<?php
$array1 = array('0' => 'a', '1' => 'b', '2' => 'c', '3' => 'd');
$array2 = array(false => 'e', 1 => 'f', 2 => 'g', 3 => 'h', 4 => 'i');
print_r($array1 + $array2);
```
```
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => d
    [4] => i
)
```
这个经常被误解为：联合是基于数组的值。 实际上呢，是基于数组的key！对于基于值的数组， 你可以使用`array_merge()`组合 `array_unique()`实现。
```
<?php
$union = array_unique(array_merge($array1, $array2));
print_r($union);
```
```
Array
(
    [0] => a
    [1] => b
    [2] => c
    [3] => d
    [4] => e
    [5] => f
    [6] => g
    [7] => h
    [8] => i
)
```

### 数组相等
相等运算符 `==` 检查两个数组是否相似。 如果第一个数组中所有的键-值对和第二个数组都相等， 这个操作符返回 `true`。 他是松散的匹配， 并且比较的时候会忽略数组项的顺序。
```
<?php
$array1 = array('1' => 1, '2' => 2, '3' => 3, '0' => 0);
$array2 = array(false => '0', 1 => '1', 2 => '2', 3 => '3');
var_dump($array1 == $array2);
```
```
bool(true)
```
两个数组元素的顺序不同，但是每个数组中都有相同的值 还是不能相等的，比如下面的例子：
```
<?php
$array1 = array(1, 2);
$array2 = array(2, 1);
var_dump($array1 == $array2);
bool(false)
```
`!=` 和 `<>` 运算符 检查是否两个数组不相似， 结果与 `==` 运算符正好相反， `==` 运算符返回`false`， 那么这个运算符就返回`true`， 反之亦然。
```
<?php
$array1 = array('1' => 1, '2' => 2, '3' => 3, '0' => 0);
$array2 = array(false => '0', 1 => '1', 2 => '2', 3 => '3');
var_dump($array1 != $array2);
```
```
bool(false)
```
## 数组一致
一致运算符 `===` 检查两个数组是否是同一个数组。 如果满足以下条件， 那么两个数组会被认为是一致的：

*   有相同数量的元素
*   有相同的键-值对
*   元素排列顺序相同
*   两个数组键相同时，键所对应的值有相同的数据类型
但是对于数组的key，如果一个数组的key是整形，而对应的另一个数组中，有经过转换是整形的字符串数字，那么一致预算符会采用松散匹配方式，认为这两个数组中的这两个项目也是相等的。但是对于一个float和string，运算符会很严格， PHP手册没有提及这一点。

```
<?php
// arrays are almost identical but keys have different types
$array1 = array('0' => '0', '1' => '1', '2' => '2', '3' => '3');
$array2 = array(0 => '0', 1 => '1', 2 => '2', 3 => '3');
var_dump($array1 === $array2);
```
```
bool(true)
```
```
<?php
// the sequence of elements in both arrays is not the same
$array1 = array('0' => '0', '1' => '1', '2' => '2', '3' => '3');
$array2 = array(1 => '1', 2 => '2', 3 => '3', 0 => '0');
var_dump($array1 === $array2);
```
```
bool(false)
```
```
<?php
// the key in the first array is a string and in the second is a float
$array1 = array('0.5' => '0');
$array2 = array(0.5 => '0');
var_dump($array1 === $array2);
```
```
bool(false)
```
而 `!==` 非一致性运算符 检查两个数组是否不一致。 这个运算符是和 `===` 完全相反的操作符， 也就是说这个运算符返回的是和 `===` 完全相反的结果。
```
<?php
$array1 = array('0' => '0', '1' => '1', '2' => '2', '3' => '3');
$array2 = array(0 => '0', 1 => '1', 2 => '2', 3 => '3');
var_dump($array1 !== $array2);
```
```
bool(false)
```
## 对数组使用其他运算符
如果对数组使用上面提及的其他运算符PHP表现出不同， 
下面是一个操作符及结果列表：

### Fatal Error: Unexpected Operand Type
如果对数组应用下面的操作符会导致一个 致命错误(fatal error) ：
位非运算符 bitwise-not operator (`~`)
算术负值运算符 arithmetic negation Operator (`-`)
算术减法运算符 arithmetic subtraction operator (`-`)
算术乘法运算符 arithmetic multiplication operator (`*`)
算术除法运算符 arithmetic division operator (`/`)

### 数组当作整数使用
当使用下面的操作符时， 数组会被当作整数对待，一个空数组(无任何元素) 会被当作 `Int(0)`, 一个非空的数组会被当作 `Int(1)`。

逻辑非 Logical-not (`!`) 空数组时返回 true ，当数组有一个或多个值，返回false
与 Bitwise-and (`&`) 两个操作数都非空返回 1， 如果有一个或者两个都是空数组，则返回 0 
或 Bitwise-or (`|`) 连个操作数都是空数组 返回 0， 否则返回1 
异或 Bitwise-xor (`^`) 如果两个操作数一样（都是0或者都是1）返回0， 否则返回1. 
左移 Shifting an array n-steps to the left with the left-shit operator (`<<`) returns the integer equivalent of 1 `<<` n if the array is non-empty. Otherwise it returns 0 (the outcome of `0 << n`).

右移 The right-shift operator (`>>`) has similar behavior as left-shift except that it shifts rightward.
取余 Modulus (`%`) returns true if both of the arrays are non-empty. If the second array is empty, it issues a “Division by Zero” error. If the first array is empty, it returns 0 (the result of `0 % 1`).
逻辑与 Logical-and (`&&` and `AND`) return false if any of the arrays are empty. If both arrays are non empty, it returns true.
逻辑或 Logical-or (`||` and `OR`) return true if any of the operand arrays are non-empty. If both arrays are empty, it returns false.
逻辑异或 If both of the arrays are either empty or non empty, logical-xor (`XOR`) returns false. Otherwise, it returns true if one of the arrays is empty.
Casting an array to bool gives false if the array is empty, otherwise true.

### 数组当作字符串
当使用`.`联接两个数组时，`.`会把数组当作字符串`Array`并联接。
```
<?php
$array1 = array(0, 1, 2, 3);
$array1 = array(0, 1, 2, 3);
var_dump($array1 . $array2);
```
```
string(10) "ArrayArray"
```
### 无作用
递增/递减类的操作符 `++` 和 `--` 对数组无作用。
```
<?php
$array1 = $array2;
var_dump((++$array1) === $array2);
```
```
bool(true)
```
## 结论
PHP缺少操作符如何操作数组的实际文档。
不过你可以查看在数组操作符文档页面的用户提交的评论。
也很欢迎你在[这里(原文)](http://phpmaster.com/array-operators-in-php-interesting-but-less-spoken/)提出问题和评论，作者很高兴进一步解释问题。


