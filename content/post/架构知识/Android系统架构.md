# Android系统架构

## 引用文章

![image-20260107135402442](D:\study\Images\image-20260107135402442.png)

> [一篇文章讲清楚Android中的MVC、MVP、MVVM架构 （附实现代码）MVC、MVP、MVVM大体上都是把系统划分 - 掘金](https://juejin.cn/post/7197230639144828988?searchId=202601071401168C58F83E75D45494ECD3)



> [Android系统架构1. Applications (应用程序） 用户就是通过这些程序来操控Android设备，这些应 - 掘金](https://juejin.cn/post/6844903543623647245?searchId=202601071401168C58F83E75D45494ECD3)

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2018/1/1/160b08f2a285693f~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

> [掌握 Android 系统架构，看这一篇就够了！-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1415759)



## 架构图

![1.png](https://p6-xtjj-sign.byteimg.com/tos-cn-i-73owjymdk6/06edbcf1854945769ff92b4c5784687f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgUm9ja0J5dGU=:q75.awebp?rk3s=f64ab15b&x-expires=1770073049&x-signature=5t0zNg2Kqbu2XIzovGjjUGUSb7k%3D)



## 包含JNI的架构

应用层 (Java/Kotlin)
     ↓
框架层 (Framework)
     ↓
**JNI 层 (Java Native Interface)**
     ↓
本地层 (Native C/C++)
     ↓
硬件抽象层 (HAL)
     ↓
Linux 内核



