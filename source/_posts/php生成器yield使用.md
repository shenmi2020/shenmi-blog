---
title: php生成器yield使用
date: 2019-10-21 10:24:57
categories: [编程]
tags:
 - [php]
---


什么是 "yield"
生成器函数看上去就像一个普通函数， 除了不是返回一个值之外， 生成器会根据需求产生更多的值。
<!-- more -->

``` php
function getValues() {
    return 'value';
}
var_dump(getValues()); // string(5) "value"
 
function getValues() {
    yield 'value';
}
var_dump(getValues()); // class Generator#1 (0) {}
```
a. 使用 yield， 你也可以使用 return
``` php
function getValues() {
   yield 'value';
   return 'returnValue';
}
$values = getValues();
foreach ($values as $value) {}
echo $values->getReturn(); // 'returnValue'
```
b. 返回键值对
``` php
function getValues() {
   yield 'key' => 'value';
}
$values = getValues();
foreach ($values as $key => $value) {
   echo $key . ' => ' . $value;
}
```
实际开发应用
很多PHP开发者不了解生成器，其实主要是不了解应用领域。那么，生成器在实际开发中有哪些应用？

读取超大文件
PHP开发很多时候都要读取大文件，比如csv文件、text文件，或者一些日志文件。这些文件如果很大，比如5个G。这时，直接一次性把所有的内容读取到内存中计算不太现实。

这里生成器就可以派上用场啦。简单看个例子：读取text文件
``` php
<?php
header("content-type:text/html;charset=utf-8");
function readTxt()
{
    # code...
    $handle = fopen("./test.txt", 'rb');
 
    while (feof($handle)===false) {
        # code...
        yield fgets($handle);
    }
 
    fclose($handle);
}
 
foreach (readTxt() as $key => $value) {
    # code...
    echo $value.'<br />';
}
```
通过上图的输出结果我们可以看出代码完全正常。
但是，背后的代码执行规则却一点儿也不一样。使用生成器读取文件，第一次读取了第一行，第二次读取了第二行，以此类推，每次被加载到内存中的文字只有一行，大大的减小了内存的使用。
这样，即使读取上G的文本也不用担心，完全可以像读取很小文件一样编写代码。

PS: 还有js生成器generator需要深入学习一下

案例参考来自https://blog.csdn.net/Csoap2/article/details/87620204