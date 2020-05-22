---
title: fido集成
---

*-by 李泽君 2019-10-25*

- #### 项目背景: RCE张总规划未来发展
- #### 实现功能: 集成fido国民认证实现指纹登录
- #### 具体需求: 用户登录手机后, 可以在设置页面打开安全管理选项, 录入指纹开启指纹登录, 然后需要密码登录时, 在本机刷指纹就可以实现登录

---
### 实现过程以及思路:

#### 1 导入Fidodemo里的Fidolibrary
#### 2 asmdescriptors.json文件复制到raw文件夹下
  这个文件是配置fido sdk的网络请求的,
  ```
  {
  "descriptorclass": [
    "com.gmrz.asm.fp.descriptor.FpAuthenticatorDescriptor",
    "com.gmrz.asm.fp.descriptor.FpAttestationAuthenticatorDescriptor",
    "com.gmrz.asm.fp.descriptor.FpSoftwareAttestationAuthenticatorDescriptor"
  ],
  "asmselect": "com.gmrz.fido.control.AsmSelector"
}
  ```
  
  最后一行路径对应fido module项目中的AsmSelector的路径, 要根据项目位置修改填写正确
  
#### 3 preferencefile.txt文件复制到asset文件夹下
  preferencefile主要配置网络请求还有appid等一些参数
  ```
  {
  "ServerNameList": [
    {
      "serverName": "外网",
      "urlMFAS": "http://fido.nationauth.cn:11104/uaf",
      "userName": "uap_in_test_11",
      "policyID": "1103",
      "phoneNumber": "18612523200",
      "transNo": "aaaa-bbbb-cccc-dddd"
    },
    {
      "serverName": "内网",
      "urlMFAS": "http://10.29.183.84:8080/uaf",
      "userName": "uap_out_test_11",
      "policyID": "1103",
      "phoneNumber": "18612523200",
      "transNo": "aaaa-bbbb-cccc-dddd"
    }
  ]
}
  ```
  
  该文件最终在在代码中以io流方式读取之后, 属性存到了sp文件以供备用, 而url地址可以通过Retrofit的baseUrl配置, 其他属性也可以自行找合适的位置直接硬编码到sp文件中
#### 4 初始化fido
  (1)Rceapp中的oncreate方法中添加
  ```
  //初始化fidonet
            FidoNet.getInstance().init(this);
            initURL();
  ```
  (2)登陆成功后在mainactivity中保存用户名, 供注册时调用, mainactivity启动模式为单例, 所以在这里执行初始化一次的代码即可
  ```
   private void initURL() {
        SharePrefeUtil.write(this, "userId", CacheTask.getInstance().getUserId());
        SharePrefeUtil.write(this, "mobile", UserTask.getInstance().getStaffInfoFromDB(CacheTask.getInstance().getUserId()).getMobile());
        SharePrefeUtil.write(this, "username", UserTask.getInstance().getStaffInfoFromDB(CacheTask.getInstance().getUserId()).getMobile()); //目前name存手机  因为登录用手机
        SharePrefeUtil.write(this, "rf1", UserTask.getInstance().getStaffInfoFromDB(CacheTask.getInstance().getUserId()).getName()); //真正的name先存在rf1
        SharePrefeUtil.write(this, "rf2", CacheTask.getInstance().getUserId());
    }
  ```
#### 5 注册,认证, 注销, 查询调用逻辑
```
  FidoConctroller->FidoSDKProcess->FidoNetControl->FidoNet->retrofitBuilder.build().create(API)
```


  
  