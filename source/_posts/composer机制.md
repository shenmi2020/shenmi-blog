---
title: composer 提供的 autoload 机制
date: 2022-04-26 10:20:04
categories: [编程]
tags:
 - [php]
---
composer 提供的 autoload 机制使得我们组织代码和引入新类库非常方便，但是也使项目的性能下降了不少 。

composer autoload 慢的主要原因在于来自对 PSR-0 和 PSR-4 的支持，加载器得到一个类名时需要到文件系统里查找对应的类文件位置，这导致了很大的性能损耗，当然这在我们开发时还是有用的，这样我们添加的新的类文件就能即时生效。 但是在生产模式下，我们想要最快的找到这些类文件，并加载他们。

因此 composer 提供了几种优化策略，下面说明下这些优化策略。

<!-- more -->

## 第一层级(Level-1)优化： 生成 classmap

### 如何运行：

执行命令 `composer dump-autoload -o` （`-o 等同于 --optimize`）

### 原理：

这个命令的本质是将 PSR-4/PSR-0 的规则转化为了 classmap 的规则， 因为 classmap 中包含了所有类名与类文件路径的对应关系，所以加载器不再需要到文件系统中查找文件了。可以从 classmap 中直接找到类文件的路径。

### 注意事项

建议开启 opcache , 这样会极大的加速类的加载。\
php5.5 以后的版本中默认自带了 opcache 。

这个命令并没有考虑到当在 classmap 中找不到目标类时的情况，当加载器找不到目标类时，仍旧会根据PSR-4/PSR-0 的规则去文件系统中查找

## 第二层级(Level-2/A)优化：权威的（Authoritative）classmap

### 执行命令：

执行命令 `composer dump-autoload -a` （`-a 等同于 --classmap-authoritative`）

### 原理

执行这个命令隐含的也执行了 Level-1 的命令， 即同样也是生成了 classmap，区别在于当加载器在 classmap 中找不到目标类时，不会再去文件系统中查找（即隐含的认为 classmap 中就是所有合法的类，不会有其他的类了，除非法调用）

### 注意事项

如果你的项目在运行时会生成类，使用这个优化策略会找不到这些新生成的类。

## 第二层级(Level-2/B)优化：使用 APCu cache

### 执行命令：

执行命令 `composer dump-autoload --apcu`

### 原理：

使用这个策略需要安装 apcu 扩展。\
apcu 可以理解为一块内存，并且可以在多进程中共享。\
这种策略是为了在 Level-1 中 classmap 中找不到目标类时，将在文件系统中找到的结果存储到共享内存中， 当下次再查找时就可以从内存中直接返回，不用再去文件系统中再次查找。

在生产环境下，这个策略一般也会与 Level-1 一起使用， 执行`composer dump-autoload -o --apcu`, 这样，即使生产环境下生成了新的类，只需要文件系统中查找一次即可被缓存 ， 弥补了Level-2/A 的缺陷。

## 如何选择优化策略？

要根据自己项目的实际情况来选择策略，如果你的项目在运行时不会生成类文件并且需要 composer 的 autoload 去加载，那么使用 Level-2/A 即可，否则使用 Level-1 及 Level-2/B是比较好的选择。

## 几个提示

-   Level-2的优化基本都是 Level-1 优化的补充，Level-2/A 主要是决定在 classmap 中找不到目标类时是否继续找下去的问题，Level-2/B 主要是在提供了一个缓存机制，将在 classmap 中找不到时，将从文件系统中找到的文件路径缓存起来，加速后续查找的速度。
-   在执行了 Level-2/A 时，表示在 classmap 中找不到不会继续找，此时 Level-2/B 是不会生效的。
-   不论那种情况都建议要开启 opcache， 这会极大的提高类的加载速度，我目测有性能提升至少 10倍。
转自http://www.dahouduan.com/2018/03/16/composer-autoload-optimize/