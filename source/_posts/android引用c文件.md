---
title: android引用c文件实战
---

 *-by 李泽君 2019-12-06*
<br></br>
- #### 项目背景: 智能眼罩项目
- #### 实现功能: 集成C文件解析蓝牙发送数据
- #### 具体需求: 蓝牙服务建立之后, 会发送数据, 经过c文件方法解码之后存到文件里


------------
> 项目本意是引用c文件接口, 之前有个误区, 只要是C文件, 就以为是涉及到JNI开发. 其实不是的,jni开发一般会打好so包, 供其他工程用, 可以起到保密, 安全等作用.封装业务核心逻辑. 如果只是单纯的引用c文件接口, 则不需要太复杂去打so包之类的操作. 两者的共同点是, 需要安装ndk环境, 否则无法引用c文件.

#### 一, jni开发
  "JNI是JAVA标准平台中的一个重要功能，它弥补了JAVA的与平台无关这一重大优点的不足，在JAVA实现跨平台的同时，也能与其它语言（如C、C++）的动态库进行交互，给其它语言发挥优势的机会。"
   关于JNI网上有很多示例文章:我贴一下比较好的几篇, 大家学习的时候可以看一下:
   [JNI详解---从不懂到理解][https://blog.csdn.net/hui12581/article/details/44832651]


[https://blog.csdn.net/hui12581/article/details/44832651]: JNI详解---从不懂到理解 "JNI详解---从不懂到理解"

[JNI 详细使用 基础【步骤】][JNI 详细使用 基础【步骤】]
[JNI 详细使用 基础【步骤】]: https://www.cnblogs.com/baiqiantao/p/5604121.html "JNI 详细使用 基础【步骤】"

实际操作过程中,可能会碰到:生成so包时会生成obj文件夹:obj下的是带符号和调试信息的，lib下的是去掉这些庞大信息后的动态链接库文件. 所以obj可以直接删除掉

* 注意 ``Android.mk Application.mk``文件的编写

#### 二, 单纯引入c文件或者C++库
   "向项目添加C/C++代码分为两种情况，一种是创建支持C/C++代码的新项目，一种是向原先不支持C/C++的已有项目添加C/C++代码。这两种情况分别对应本教程的第一大点和第二大点。"
   
   教程:
   [Android Studio向项目添加C/C++原生代码教程][Android Studio向项目添加C/C++原生代码教程]
[Android Studio向项目添加C/C++原生代码教程]: https://www.cnblogs.com/lsdb/p/9337285.html "Android Studio向项目添加C/C++原生代码教程"

 * 操作之后注意cmake文件处理下
 ![](http://120.24.225.154:4999/server/../Public/Uploads/2019-12-06/5dea198a9c229.png)