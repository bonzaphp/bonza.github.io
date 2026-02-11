---
layout: post
title:  "PHP项目编码规范"
date:   2026-02-11 02:53:00 +0800
categories: php
tags: php 编码规范 代码风格 PSR-4
author: 来财
---


* content
{:toc}


# PHP项目编码规范




本文详细介绍了PHP项目的编码规范，包括基本约定、代码样式风格、控制结构、注释规范、数据表命名规范以及PSR-4自动加载规范，为PHP开发团队提供统一的编码标准。

# PHP项目编码规范

- - -

## 一、基本约定

### 1、源文件

（1）、 纯PHP代码源文件只使用 <?php 标签，省略关闭标签 ?> ；

（2）、源文件中PHP代码的编码格式必须是无BOM的UTF-8格式；

（3）、使用 Unix LF(换行符)作为行结束符；

（4）、一个源文件只做一种类型的声明，即，这个文件专门用来声明Class, 那个文件专门用来设置配置信息，别混在一起写；

### 2、缩进

使用Tab键来缩进，每个Tab键长度设置为4个空格；

### 3、行

一行推荐的是最多写120个字符，多于这个字符就应该换行了，一般的编辑器是可以设置的。

### 4、关键字 和 True/False/Null

PHP的关键字，必须小写，boolean值：true，false，null 也必须小写。

下面是PHP的"关键字"，必须小写：

```php 

'__halt_compiler', 'abstract', 'and', 'array', 'as', 'break', 'callable', 'case', 'catch', 'class', 'clone', 'const', 
'continue', 'declare', 'default', 'die', 'do', 'echo', 'else', 'elseif', 'empty', 'enddeclare', 'endfor', 'endforeach',
 'endif', 'endswitch', 'endwhile', 'eval', 'exit', 'extends', 'final', 'for', 'foreach', 'function', 'global', 'goto', 'if', 
 'implements', 'include', 'include_once', 'instanceof', 'insteadof', 'interface', 'isset', 'list', 'namespace', 'new', 'or', 'print', 'private', 'protected', 'public', 'require', 'require_once', 'return', 'static', 'switch', 'throw', 'trait',
  'try', 'unset', 'use', 'var', 'while', 'xor'

```



### 5、命名

（1）、类名 使用大驼峰式（StudlyCaps）写法；

（2）、（类的）方法名 使用小驼峰（cameCase）写法；

（3）、函数名使用 小写字母 + 下划线 写法，如 function http_send_post()； 

（4）、变量名 使用小驼峰写法，如 $userName；

### 6、代码注释标签

如 函数注释、变量注释等，常用标签有 [@package](https://phpdoc.org/docs/latest/references/phpdoc/tags/package.html)、[@var](https://phpdoc.org/docs/latest/references/phpdoc/tags/var.html)、[@param](https://phpdoc.org/docs/latest/references/phpdoc/tags/param.html)、[@return](https://phpdoc.org/docs/latest/references/phpdoc/tags/return.html)、[@author](https://phpdoc.org/docs/latest/references/phpdoc/tags/author.html)、[@todo](https://phpdoc.org/docs/latest/references/phpdoc/tags/todo.html)、[@throws](https://phpdoc.org/docs/latest/references/phpdoc/tags/throws.html)

必须遵守 phpDocument 标签规则

### 7、业务模块

（1）、涉及到多个数据表 更新/添加 操作时，最外层要用事务，保证数据库操作的原子性；

（2）、Model层，只做简单的数据表的查询；

（3）、业务逻辑统一封装到 Logic层；

（4）、控制器只做URL路由，不要当作 业务方法 调用；

（5）、 控制器层不能出现SQL操作语句，如 ThinkPHP框架的 where()、order() 等模型方法，

即，控制器中，不要出现类似这样的SQL语句：D('XXX')->where()->order()->limit()->find();  

where()、order()、limit() 等SQL方法只能出现在 Model层、业务层！

## 二、代码样式风格

### 1、命名空间(Namespace) 和 导入(Use)声明

先简单文字描述下：

1. 命名空间(namespace)的声明后面必须有一行空行；

2. 所有的导入(use)声明必须放在命名空间(namespace)声明的下面；

3. 一句声明中，必须只有一个导入(use)关键字；

4. 在导入(use)声明代码块后面必须有一行空行；

用代码来说明下：

![](https://blog.bonza.cn/assets/php-coding/php-namespace-use-declaration.png)

namespace下空一行，才能使用use，再空一行，才能声明class

![](https://blog.bonza.cn/assets/php-coding/php-class-declaration-format.png)

 ### 2、类(class)，属性(property)和方法(method)

（1）、继承(extends) 和实现(implement) 必须和 class name 写在一行。

![](https://blog.bonza.cn/assets/php-coding/php-class-extends-implement.png)

（2）、属性(property)必须声明其可见性，到底是 public 还是 protected 还是 private，不能省略，也不能使用var, var是php老版本中的什么方式，等用于public。

![](https://blog.bonza.cn/assets/php-coding/php-property-visibility.png)

（3）、方法(method)，必须 声明其可见性，到底是 public 还是 protected 还是 private，不能省略。如果有多个参数，第一个参数后紧接"|" ，再加一个空格：function_name ($par, $par2, $pa3), 如果参数有默认值，"="左右各有一个空格分开。

![](https://blog.bonza.cn/assets/php-coding/php-method-parameters.png)

（4）、当用到抽象(abstract)和终结(final)来做类声明时，它们必须放在可见性声明 （public 还是protected还是private）的前面。而当用到静态(static)来做类声明时，则必须放在可见性声明的后面。

直接上代码：

![](https://blog.bonza.cn/assets/php-coding/php-abstract-final-static.png)

### 3、控制结构

控制接口，就是 if else while switch等。这一类的写法规范也是经常容易出现问题的，也要规范一下。

（1）、if，elseif，else写法，直接上规范代码吧：

![](https://blog.bonza.cn/assets/php-coding/php-if-elseif-else-format.png)

（2）、switch，case 注意空格和换行，还是直接上规范代码：

![](https://blog.bonza.cn/assets/php-coding/php-switch-case-format.png)

（3）、while，do while 的写法也是类似，上代码：

![](https://blog.bonza.cn/assets/php-coding/php-while-do-while-format.png)

（4）、for的写法

![](https://blog.bonza.cn/assets/php-coding/php-for-loop-format.png)

（5）、foreach的写法

![](https://blog.bonza.cn/assets/php-coding/php-foreach-format.png)

6）、try catch的写法

![](https://blog.bonza.cn/assets/php-coding/php-try-catch-format.png)

### 4、注释

（1）、行注释

// 后面需要加一个空格；

如果 // 前面有非空字符，则 // 前面需要加一个空格；

（2）、函数注释

参数名、属性名、标签的文本 上下要对齐；

在第一个标签前加一个空行；

### 5、数据表和字段

*   数据表和字段采用小写加下划线方式命名，并注意字段名不要以下划线开头，例如 think_user 表和 user_name字段，不建议使用驼峰和中文作为数据表字段命名。

![](https://blog.bonza.cn/assets/php-coding/php-database-naming.png)

 5、空格

（1）、赋值操作符（=，+= 等）、逻辑操作符（&&，||）、等号操作符（==，!=）、关系运算符（<，>，<=，>=）、按位操作符（&，|，^）、连接符（.） 左右各有一个空格；

（2）、if，else，elseif，while，do，switch，for，foreach，try，catch，finally 等 与 紧挨的左括号"("之间有一个空格；

（3）、函数、方法的各个参数之间，逗号（","）后面有一个空格；

6、空行

（1）、所有左花括号 { 都不换行，并且 ｛ 紧挨着的下方，一定不是空行；

（2）、同级代码（缩进相同）的 注释（行注释/块注释）前面，必须有一个空行；

（3）、各个方法/函数 之间有一个空行；

（4）、namespace语句、use语句、clase语句 之间有一个空行；

（5）、return语句

如果 return 语句之前只有一行PHP代码，return 语句之前不需要空行；

如果 return 语句之前有至少二行PHP代码，return 语句之前加一个空行；

（5）、if，while，switch，for，foreach、try 等代码块之间 以及 与其他代码之间有一个空行；

【参考示例 汇总】

参考1：

![](https://blog.bonza.cn/assets/php-coding/php-code-example-1.png)

参考2:

![](https://blog.bonza.cn/assets/php-coding/php-code-example-2.png)

参考3：

![](https://blog.bonza.cn/assets/php-coding/php-code-example-3.png)

参考4：

![](https://blog.bonza.cn/assets/php-coding/php-code-example-4.png)

![示例](/assets/php-coding/php-code-example-summary.png)

总结：所有除类，方法以外的左花括号 { 都不换行，并且 ｛ 紧挨着的下方，一定没有空行！

书写原则：做到 代码紧凑 而又不失 小模块化 ！

## ==PSR-4 规范==

PSR-4规范是刚出没多久的一条新的规范，它也是规范 自动加载(autoload)的，是对PSR-0的修改，属于补充规范，

### 简单说下，主要是以下几点： 

1. 废除了PSR-0中_就是目录分割符的写法，_下划线在完全限定类名中是没有特殊含义了。 

2.  类文件名要以 .php 结尾。 

3. 类名必须要和对应的文件名要一模一样，大小写也要一模一样。