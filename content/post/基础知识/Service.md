# Service

# Android Service 生命周期详解

## 一、Service 生命周期概览

![image-20260116112729183](D:\study\Images\image-20260116112729183.png)



## 二、总结

**Service 生命周期核心要点**：

| 方法                 | 调用时机               | 调用次数 | 作用              |
| -------------------- | ---------------------- | -------- | ----------------- |
| **onCreate()**       | Service 创建时         | 1次      | 一次性初始化      |
| **onStartCommand()** | 每次 startService() 时 | 多次     | 处理启动请求      |
| **onBind()**         | 每次 bindService() 时  | 多次     | 返回 IBinder 接口 |
| **onUnbind()**       | 所有客户端解绑时       | 1次      | 清理绑定相关资源  |
| **onRebind()**       | 客户端重新绑定时       | 多次     | 处理重新绑定      |
| **onDestroy()**      | Service 销毁时         | 1次      | 清理所有资源      |







