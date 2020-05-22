---
title: android引入c文件之后, debug时间过长的问题
---

 *-by 李泽君 2019-12-26*

- #### 项目背景: 智能眼罩

>项目引入c文件编译时, 增加了ndk库, ndk一般会引入lldb 去让Androidstudio可以debugc文件

##### 问题
> 引入lldb后每次使用进程断点调试时会加载LLDB调试c文件的进程, 时间长达两分钟左右, 实际上可能并不需要去debug C文件

##### 解决办法:

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-12-26/5e042ffa9a98d.png)

*如图所示, 如果只需要debug java文件, 则选择java Debugger , 选择Auto的话, 则项目中含有C文件或者ndk时, 会自动加载LLDB, 导致每次debug进程都是漫长的等待.*

问题虽然简单, 但是百度不到, 试了好多次才解决~ 记录一下~