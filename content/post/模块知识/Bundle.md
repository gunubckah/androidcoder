# Bundle

- Bundle 是 Android 中**最核心的数据传递容器**，简单来说就是 Android 的 **"数据包"** 或 **"数据字典"**。



## Bundle 的核心本质

```java
public final class Bundle implements Parcelable, Cloneable {
    // 内部使用 ArrayMap 存储键值对
    ArrayMap<String, Object> mMap;
}
```

- **本质是**：一个实现了 Parcelable 的键值对容器，支持跨进程序列化。



## Bundle 的核心作用

![image-20260126142545587](D:\study\Images\image-20260126142545587.png)





- 应用场景
  - Activity 间数据传递
  - Fragment 数据传递
  - 状态保存



## Bundle 支持的数据类型



### 类型转换表

| 数据类型     | Bundle 方法                    | 获取方法                       | 备注            |
| ------------ | ------------------------------ | ------------------------------ | --------------- |
| 基本类型     | putInt, putBoolean...          | getInt, getBoolean...          | 有默认值重载    |
| String       | putString                      | getString                      | 可返回 null     |
| 数组         | putIntArray, putStringArray... | getIntArray, getStringArray... | 返回数组或 null |
| Parcelable   | putParcelable                  | getParcelable                  | 需要 Class 参数 |
| Serializable | putSerializable                | getSerializable                | 性能较差        |
| Bundle       | putBundle                      | getBundle                      | 可嵌套          |



## Bundle 的底层实现原理

### 1. 数据结构

```java
// Bundle 内部结构简化
public class Bundle {
    // 核心存储：ArrayMap（Android 优化过的 Map）
    ArrayMap<String, Object> mMap;
    
    // Parcel 支持
    Parcel mParcel = null;
    boolean mFrozen = false;  // 是否冻结
    
    // 懒加载机制
    ClassLoader mClassLoader;
}
```

### 2. 序列化过程

```java
// 1. 写入过程
writeToParcel(Parcel parcel, int flags) {
    // 先将 mMap 写入 Parcel
    parcel.writeArrayMap(mMap);
}

// 2. 读取过程
readFromParcel(Parcel parcel) {
    // 从 Parcel 读取到 mMap
    mMap = parcel.readArrayMap(mClassLoader);
}
```

### 3. 性能优化特点

```kotlin
// 1. ArrayMap 比 HashMap 更节省内存
// HashMap: 数组+链表+Entry对象
// ArrayMap: 两个数组（mHashes+mArray）

// 2. 延迟创建 Parcel
// 只有在需要序列化时才创建 Parcel 对象

// 3. 内存复用
// 清空时保留数组，不清除引用
bundle.clear()  // 只是清空引用，不释放数组内存
```



## 与Intent的关系

Inten是信使，Bundle是数据包。