---
layout:  post
title:    Erlang错误日志
subtitle:   雄关漫道真如铁，而今迈步从头越
author:  "半桶水"
header-img: "img/post-bg-2016-1.jpg"
tags:
 - Erlang
---

　错误日志就像痛觉神经，能帮助我们很快的定位到报错发生的位置。同时我们可以根据具体的需求或习惯，对错误日志进行一系列的配置。

　在介绍错误日志配置前，先简单介绍下“错误”，根据产生来源可分为两种：

- __人为产生__  
- __自动产生__

　何为人为产生呢，就是码农在写代码时灵光一闪，“诶，这里我想让代码报一条hello girlfriend 的错误”，于是便可以调用Erlang错误日志的相关api主动让代码写入一条错误信息。

　顾名思义，自动产生呢就是Erlang自己产生的报错信息了，如以下代码

	GirlFriendName = xiaohong.
	GirlFriendName = xiaoli.

　Elang这种一心一意的语言可不像其他那些妖艳花心的语言，Erlang可不会允许你拥有两个女朋友，如果你想拥有，不好意思，会报错:-) 

　除人为调用api产生的错误信息外，Erlang内置的错误处理模块会记录以下三种类型的信息：

-  __监管报告__  　　　监管进程启动，停止被监管的进程时，产生报告
-  __进程报告__　　　  OTP监管进程启动或停止时，产生的报告
-  __崩溃报告__ 　　　 被监管进程退出，且退出原因不是normal或shutdown时

　一般情况下，我们只关注错误信息，而忽视掉进程信息，不然错误日志可能就会boomboom。

　下面列出常用的错误日志配置

	erl -boot start_sasl -config elog

　这句命令，用于告诉erlang的内置错误处理模块使用elog配置来处理错误信息。

　elog配置采用Erlang程序设计中推荐的产品化环境配置：

<pre>
[{sasl, [   
		{sasl_error_logger, false},    %%错误信息显示点
		{errlog_type, error},   %%日志错误类型
		{error_logger_mf_dir, "./logs"},     %% 文件目录
		{error_logger_mf_maxbytes, 1048760}, %% 文件大小   
		{error_logger_mf_maxfiles, 10}       %% 最大可用文件个数
]}].   
</pre>

　未完待续，接下来计划贴上源码对于这块内容的相关处理。
