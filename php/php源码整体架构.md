## PHP 语言的执行原理

![php执行顺序](https://oushuifa.github.io/doc/image/exec.png)

1. 词法分析将php转换成有意义的Token，此步骤的词法分析器使用 [Re2c](http://re2c.org/) 实现。

2. 语法分析器将Token和符合文法规则的代码生成抽象语法树（简称AST）。语法分析器基于 [Bison](https://www.gnu.org/software/bison/) 实现。语法分析使用了BNF（Backus-NaurForm,巴科斯范式）来表达文法规则，Bison借助状态机、
状态转移表和压栈、出栈等一系列操作，生成抽象语法树。

3. 上步的抽象语法树生成对应的opcode，并被虚拟机执行。

## PHP 内核架构

![php内核架构](https://oushuifa.github.io/doc/image/arch.png)

php架构大致分为四个部分。

1. **Zend 引擎** ：PHP的变量设计、内存管理、进程管理等都在引擎层实现，引擎为PHP提供了基础服务，PHP的可靠性和高性能都依赖引擎的基础支撑

2. **PHP 层**：Zend引擎为php提供基础能力，而来自外部的交互需要通过PHP层处理。

3. **SAPI**：即 `Server API` 包含常见的 `cli SAPI` 和 `fpm SAPI` 。PHP定义好输入输出规范，一句此规范与PHP交互的一方都可以称为 `Server` .

4. **扩展部分**：Zend 引擎提供核心能力和接口规范。再次基础上开发的扩展，为PHP代码的性能和功能的多样性提供了丰富的选项。

## PHP 源码结构初步介绍

PHP主要包含一下源码目录：`sapi` 、`main`、`ext`、`Zend`、`TSRM`。

### sapi 目录源码

`sapi` 是对输入输出层的抽象，是PHP提供对外服务的规范。

例如，命令行模式对应的二进制程序是 `bin/php`；`CGI` 模式对应的是二进制文件是 `bin/cgi`; `FastCGI` 对应的是二进制程序 `sbin/php-fpm` 。同时多个模式抽象出相同的模板（源码中的结构体 `sapi_module_struct`），其定义了模式启动、关闭、激活、失效等多个钩子函数指针。 每个模式将这些函数指针指向自己的函数，实现不同模式之间输入、输出的差异化。
 
### Zend 目录源码

1. 内存管理模块
PHP实现了自己的 [内存管理器](php内存管理器.md) ，主要操作实现在 `zend_alloc_size.h`，`zend_alloc.c`，`zend_alloc.h`三个文件中。

    - `zend_alloc_size.h` ：PHP内存分配策略按照大小有三种规格，分配时会按照实际需要空间选择内存对齐，在进行分配。**small** 内存小于3072B，**large** 内存介于3072B到4KB之间，**huge** 内存大于2MB。
    - `zend_alloc.c` ：定义了内存操作函数的实现以及PHP内存管理器的核心数据结构 `_zend_mm_heap` 等
    - `zend_alloc.h` ：主要是一些内存操作函数的声明。PHP内存管理器在C语言常见的内存操作函数 `molloc()`，`free()` 等之上做了一层封装。

2. 垃圾回收
为了解决循环引用问题，PHP引入了垃圾回收机制。PHP的卡机回收的实现主要包含在 `zend_gc.h` 和 `zend_gc.c0` 中。详见 [垃圾回收](php垃圾回收.md)

3. 数组实现
数组是PHP最常用、最复杂的数据类型之一。经常有人调侃PHP是面向数组编程 😎 。支持数组的底层数据结构 `HashTable` 也经常在扩展开发中内开发者使用。PHP数组的底层设计主要在 `zend_hash.c` 和 `zend_hash.h` 两个文件中实现。详见 [PHP数组](php数组的实现.md)

### main 目录源码

`main` 目录是 `SAPI` 层和 `Zend` 层的粘合剂。起到一个承上启下的作用：承上，解析 `SAPI` 的请求，分析要执行的脚本和参数；启下，调用 `Zend` 引擎之前完成必要的初始化工作（模块初始化，脚本执行等）

### ext 目录源码

`ext` 是PHP扩展相关的目录，常用的 `array`、`str`、`pdo` 等系列函数都在这里定义。详见 [PHP扩展开发](php扩展开发.md)

### TSRM 目录源码介绍

`TSRM` 是 `Thread Safe Resource Manager` 的缩写 ———— 线程安全资源管理器。可在编译时增加编译参数 `enable-maintainer-zts`，以激活 `ZTS` 常量，支持线程安全。 