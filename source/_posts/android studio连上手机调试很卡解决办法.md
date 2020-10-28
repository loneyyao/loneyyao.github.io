---
title: Androidstudio 连上手机调试后特别卡
---

 *-by 李泽君 2020-10-23* 

------------
### 解决办法:
1, 猜想可能是instant run之类的导致的跳转页面特变卡和慢
2, studio版本为4.0, 找不到instant run
3, 在这里找到了:
打开设置 -> Build,Execution,Deployment -> Debugger -> HotSwap 取消选中右面的 Enable hot-swap agent for Groovy code
你直接搜Instant Run是搜不到的，用HotSwap代替了
4, 重新调试手机, 发现一点也不卡了















