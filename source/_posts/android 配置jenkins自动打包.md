---
title: android配置jenkins
---

*-by 李泽君 2019-10-21* 


- #### 项目背景: RCE以及其他基于RCE项目 后期可以发散到任何项目
- ####  实现功能: 测试人员可以基于jenkins打包生产二维码下载安装
- ####  jenkins地址: http://120.24.225.154:8080/

> 11-19更新:
由于蒲公英文档Api更新, 导致上传失败
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-11-19/5dd36cd207bea.png)
新增安装方式参数为必选:2和3  分别为密码安装和邀请安装  之前为1现在已经失效
密码安装需要设置密码  将密码明文显示到pgyer插件的安装显示之后

### 具体实施步骤:

#### 1, 新建一个自由风格的构建
![新建一个项目](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da829f8c9b4f.png "新建一个项目")
#### 2,配置该项目
![点击配置](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82a1c6c097.png "点击配置")

<font color=red>*以下几点需要注意:*</font>
(1)参数化构建过程中的参数, 最终会显示在构建页面, 供测试人员自定义打包参数, 如:渠道, debug/release类型, 时间, 版本等等.

 ``部分参数如下所示:具体可以参考yuanhe项目``

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82aa5f0cfb.png)

(2) 源码管理, 需要填写对应的仓库地址, 以及该仓库的账号密码登陆, 一般填写登陆gitlab的账户和密码, 前提是需要在gitlab配置了自己本机的ssh

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82bba4cdb3.png)

(3) 构建环境中的``set build name`` 是个插件, 支持上面定义的构建参数的直接引用

```一个例子: #${BUILD_NUMBER} - ${APP_NAME} - ${PRODUCT_FLAVOR_BUILD} - ${BUILD_TYPE} - ${APP_VERSION} - ${BUILD_TIMESTAMP} ```

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82d138250d.png)

配置好之后, 构建时不会显示#1等, 而是显示如下图所示:
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da83115342ed.png)

(4) 构建命令中也可以使用上面的构建参数
``` 一个例子: 
clean
assemble${PRODUCT_FLAVOR_BUILD}${BUILD_TYPE}
```
表示先clean工程, 然后执行构建命令, 动态根据变量值打出对应包

<font color=red size=4>*注意: 要想使上面的构建参数生效, 必须勾选这个选项:</font>
勾选以后(3)中的参数才可以覆盖app工程中的gradle.properties文件中的值, 达到jenkins配置app的效果!!!*

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82e35dca05.png)

(5) 构建后操作:
1> 上传蒲公英是个jenkins插件,需要先下载安装,然后在蒲公英注册账户, 填写插件中的值.
2> set build Description也是一个插件,其中配置的html代码想要展示出来, 必须在jenkins中设置如下:

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da830a7c0d36.png)

然后配置代码段:
```
<div><img src="http://120.24.225.154:8080/job/android_app_yuanhe/ws/apks/qrcode.png"  alt="扫描安装" /></div> <br> <p>扫描下载安装app</p>
```

![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da82f7403600.png)

src地址在这里面可以找到:
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da830145ff1f.jpg)
apks地址为项目gradle中配置的打包地址, 下文会有项目gradle配置,  <font color=red>注意, gradle配置与jenkins配置的参数强关联, 必须对应好.</font>

配置好后, 打包完成时, 会在页面显示生成的二维码
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da831700ce2d.png)
二维码由蒲公英生成, 经过蒲公英插件, 下载到本地配置的工作空间

配置完毕就可以愉快的进行打包了, 打好之后会发送邮件, 邮件由蒲公英去配置, jenkins也可以配置直接发邮件, 但由于我的smtp无法配置好, 只能通过蒲公英发邮件了.

#### android工程配置:
android工程配置应该先与jenkins配置, 为了便于理解, 放在此处讲解.

(1)多渠道打包:
jenkins配置的值必须与gradle配置的这几个值相同, 这样打包命令才会生效. 打包命令格式为:``assemble+渠道名+打包类型(debug/release)``
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da832cc9e4f8.png)

代码中配置该渠道对应的资源文件, 打该渠道时会自动选择此资源文件, 元禾项目通过配置string 实现了不同的渠道包对应不同的server地址:
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da833955137d.png)

(2)配置apk生成路径
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da83420659f0.png)

红框参数为gradle.properties配置, <font color=red>*注意: 必须与jenkins配置的打包参数名称一致,jenkins才可以动态改变这个文件的值.*</font>
![](http://120.24.225.154:4999/server/../Public/Uploads/2019-10-17/5da835e12a15f.png)

``IS_JENKINS``要在jenkins的构建参数中配置, 这样可以通过gradle中的配置, 区分jenkins和本地打包的apk生成路径.
*jenkins的打包路径改变, 需要在蒲公英插件中修改二维码回传路径, 以及``(5) 构建后操作:`` 中的地址,目前二维码地址与apk地址处于同级目录, 若修改gradle配置的打包目录结构, 请同步修改jenkins配置*


## 全剧终
## 2019-10-17 17:41:29 星期四



