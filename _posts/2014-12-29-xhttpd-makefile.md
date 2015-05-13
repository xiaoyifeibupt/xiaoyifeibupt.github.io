---
layout: post
title:  从零开始实现自己的httpd——Makefile
categories: [http]
---

一个工程中的源文件不计其数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。

###自动化编译



###主要功能

Make工具最主要也是最基本的功能就是通过makefile文件来描述源程序之间的相互关系并自动维护编译工作。而makefile 文件需要按照某种语法进行编写，文件中需要说明如何编译各个源文件并连接生成可执行文件，并要求定义源文件之间的依赖关系。

###规则

让我们先来粗略地看一看Makefile的规则。

target ... : prerequisites ...
command
...
...


目标：依赖
执行指令 ...
target也就是一个目标文件，可以是Object File，也可以是执行文件。还可以是一个标签（Label）。
① prerequisites就是，要生成那个target所需要的文件或是目标。
② command也就是make需要执行的命令。（任意的Shell命令）
这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说，prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行(command一定要以Tab键开始，否者编译器无法识别command），减少重复编译，提高了其软件工程管理效率。
文件定义与命令
Makefile文件作为一种描述文档一般需要包含以下
命令行中执行makefile
命令行中执行makefile
内容:
宏定义
源文件之间的相互依赖关系
可执行的命令
Makefile中允许使用简单的宏指代源文件及其相关编译信息，在Linux中也称宏为变量。在引用宏时只需在变量前加$符号，但值得注意的是，如果变量名的长度超过一个字符，在引用时就必须加圆括号（）。
