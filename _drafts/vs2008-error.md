---
layout: post
title: "Today, I choose to be a (vegan) wolf"
description: ""
category: blog
---

在VS2008上运行程序出错：error PRJ0003 : 生成“cmd.exe”错误，这个问题主要是由于系统中的环境变量没有增加相关的路径所知。

刚装完Visual2008，建了个Win32工程，一编译就出现 “项目 : error PRJ0003 : 生成“cmd.exe”时出错。”。
  解决方案：工具—>选项—>项目和解决方案—>VC++目录，在可执行文件栏中加上如下路径：
$(SystemRoot)/System32
$(SystemRoot)
$(SystemRoot)/System32/wbem 
现在运行成功了，输出内容：
------ 已启动生成: 项目: Game, 配置: Debug Win32 ------
正在嵌入清单...
生成日志保存在“file://e:/Visual Studio 2008/Projects/Game/Game/Debug/BuildLog.htm”
Game - 0 个错误，个警告
========== 生成: 成功1 个，失败0 个，最新0 个，跳过0 个==========