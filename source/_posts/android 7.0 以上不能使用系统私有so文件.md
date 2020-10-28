---
title: android 7.0 以上不能使用系统私有so文件
---
<h4 align = "center"> *-by 李泽君 2020-10-28* </h4>

<br></br>

> 最近一次运行rce 1.6.6标准版时, 发现无法登陆, 连不上IMserver, 期初怀疑有二:
  1, 是运维部署版本有问题没有配置好
  2, 是客户端包没打好
  
* 后来经过逐一排查, 发现还是包的问题. 首先说明原因:

> 错误日志如下：

```
11-02 03:33:12.533 4855-4855/? E/RongLog: [ RongExceptionHandler ] uncaughtException

  3   java.lang.UnsatisfiedLinkError: dlopen failed: library "libsqlite.so" not found
  4   at java.lang.Runtime.loadLibrary0(Runtime.java:977)
  5   at java.lang.System.loadLibrary(System.java:1530)
  6   at io.rong.imlib.NativeObject.<clinit>(NativeObject.java:8)
  7   at io.rong.imlib.NativeClient.init(NativeClient.java:133)
  8   at io.rong.imlib.LibHandlerStub.<init>(LibHandlerStub.java:46)
  9   at io.rong.imlib.ipc.RongService.onBind(RongService.java:31)
 10   at android.app.ActivityThread.handleBindService(ActivityThread.java:3189)
 11   at android.app.ActivityThread.-wrap3(ActivityThread.java)
 12   at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1555)
 13   at android.os.Handler.dispatchMessage(Handler.java:102)
 14   at android.os.Looper.loop(Looper.java:154)
 15   at android.app.ActivityThread.main(ActivityThread.java:6077)
 16   at java.lang.reflect.Method.invoke(Native Method)
 17   at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:865)

```

解答：

原因是7.0以后，Andorid不允许直接访问系统的私有so文件了。

两种解决方案：

1. 把targetSdkVersion改小于24。

2. apk中带上需要的so文件，这儿是libsqlite.so

下载附件中的so文件，放到对应的文件夹下，如libs/armeabi-v7a/libsqlite.so，并注意你的gradle是否引用了这个目录，如:
``
sourceSets {
        main {
            jni.srcDirs = []
            jniLibs.srcDirs = ['libs']
        }
    }
``
编译测试。

通过Android Studio的Analyze APK功能[Build -> Analyze APK...], 或直接解开apk包，查看是apk的lib目录下是否已包含libsqlite.so文件

收获如下:
刚开始时候找不到错误日志, 所以定位不到错误问题. 后来发现日志截取不全, 只看了应用日志没看到系统日志.

### 全量日志获取方法:
* adb logcat -v time
* android studio日志过滤时, 不要仅选择``show only selected application``
![](http://120.24.225.154:4999/server/../Public/Uploads/2020-10-28/5f98dd26b88d4.png)

### libsqlite.so文件是干嘛用的:
虽然Android提供了访问 sqlite的Java接口，但现在需要在ndk中使用 c 语言访问sqlite。
方法有二：

第一种：使用源码 sqlite3.h, sqlite3.c。
从android源码网站下载sqlite.git库，找到这两个文件，放到项目源码中去，进行ndk编译即可。

优点：简单，很容易想到
缺点：不能有效利用 /system/lib/libsqlite.so文件，导致编译得到的库文件过大。

改进方法二：
根据 liblog.so的使用方法推测得到该方法。
1、先adb pull /system/lib/libsqlite.so得到 libsqlite.so文件
2、把libsqlite.so文件放到 $NDK/platforms/android-3/arch-arm/usr/lib目录下。
3、把 sqlite3.h放到 $NDK/platforms/android-3/arch-arm/usr/include/android目录下。
4、在Android.mk 文件中加入语句： LOCAL_LDLIBS := -lsqlite
5、进行ndk编译

所以这个so文件是你用到的一些三方库(我们项目中是imlib中的c文件用到了这个so文件).