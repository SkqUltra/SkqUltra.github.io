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

　以下10月16记

　使用上述blog配置进行测试，输入

	error_logger:error_msg("123").  

　产生一条错误信息，之后进入logs目录下会发现多了两个文件，一个名为index，一个名为1。当你想打开文件1看看报错信息时，便瞬间傻眼了，这尼玛都是什么东西，文件内容节选如下：

	0155 8368 0268 0268 0362 0000 07e0 610a
	6110 6803 610f 611f 610b 6803 6400 0b69
	6e66 6f5f 7265 706f 7274 6764 000d 6e6f
	6e6f 6465 406e 6f68 6f73 7400 0000 2000
	0000 0000 6803 6764 000d 6e6f 6e6f 6465
	406e 6f68 6f73 7400 0000 2300 0000 0000
	6400 0870 726f 6772 6573 736c 0000 0002

　原来erlang还提供了一套rb模块来提取错误信息，简单来说就是可以用rb模块中提供的一些函数，来将上面那段天书提取成我们需要的信息，具体函数这里就不一一介绍，因为并不打算使用 囧rz..

　但是我还是对rb模块将上述天书转变成普通字符的过程感兴趣，查阅相关源码后，可以提取出主要算法：

<pre>
1. {ok, Pid} = file:open("1",  [read]).
2. [Hi, Lo] = io:get_chars(Pid,'',2).
3. Len = ((Hi bsl 8) band 16#ff00) bor (Lo band 16#ff).
4. String = binary_to_term(list_to_binary(io:get_chars(Pid,'',Len))).
</pre>

　在第一步获取到Pid后，循环2至4步骤至文件结束，每一次循环生成的String就是给人类阅读的数据了。
　

　用这套流程来还原下上述十六进制的数据，(节选)结果为：

<pre>
{{'{{{'}}2016,10,16},{15,31,11}},
 {info_report,<0.32.0>,
     {<0.35.0>,progress,
      [{supervisor,{local,sasl_safe_sup}},
       {started,
           [{pid,<0.36.0>},
            {id,alarm_handler},
            {mfargs,{alarm_handler,start_link,[]}},
            {restart_type,permanent},
            {shutdown,2000},
            {child_type,worker}]}]}}}
{{'{{{'}}2016,10,16},{15,31,11}},
 {info_report,<0.32.0>,
     {<0.35.0>,progress,
      [{supervisor,{local,sasl_safe_sup}},
       {started,
           [{pid,<0.37.0>},
            {id,overload},
            {mfargs,{overload,start_link,[]}},
            {restart_type,permanent},
            {shutdown,2000},
            {child_type,worker}]}]}}}
</pre>

　感觉这样操作十分之繁琐，如何才能自定义输出的错误日志呢？摸索ing...