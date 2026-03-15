# Serializable vs Parcelable 接口

这两个接口都是 Android 中用于**对象序列化**的，但设计目的、实现方式和性能特点有显著不同。让我详细为您解析：

## 📊 核心对比表

| 维度           | Serializable (Java标准) | Parcelable (Android专用)      |
| -------------- | ----------------------- | ----------------------------- |
| **来源**       | Java 标准库             | Android SDK 特有              |
| **设计目标**   | 通用序列化（磁盘/网络） | 内存高效序列化（IPC）         |
| **性能**       | 慢（反射+IO）           | 快（手动写入内存）            |
| **内存开销**   | 大（创建临时对象）      | 小（直接操作内存）            |
| **实现复杂度** | 简单（自动）            | 复杂（手动）                  |
| **适用场景**   | 存储到文件/网络传输     | Activity/Binder/Intent 间传递 |
| **版本兼容**   | 有（serialVersionUID）  | 无（需手动处理）              |
| **线程安全**   | 是                      | 是                            |



## 🔄 工作原理对比

### Serializable 工作原理

```java
// 1. 写入过程
ObjectOutputStream oos = new ObjectOutputStream(outputStream);
oos.writeObject(obj);  // 通过反射获取所有字段

// 内部流程：
// 1. 递归遍历对象图
// 2. 反射获取字段值
// 3. 写入字节流
// 4. 生成大量临时对象
```

### Parcelable 工作原理

```java
// 1. 写入过程
Parcel parcel = Parcel.obtain();
obj.writeToParcel(parcel, 0);

// 内部流程：
// 1. 直接调用 writeXxx() 方法
// 2. 写入原生内存
// 3. 无反射，无临时对象
// 4. 性能接近直接内存操作
```

## 📱 代码实现对比

### Serializable 实现

```java
// 非常简单，只需实现接口
public class User implements Serializable {
    
    // 强烈建议：版本控制ID
    private static final long serialVersionUID = 1L;
    
    private String name;
    private int age;
    private String email;
    private transient String password;  // 不会被序列化
    
    // Getter/Setter...
    
    // 可选：自定义序列化过程
    private void writeObject(java.io.ObjectOutputStream out)
        throws IOException {
        // 自定义序列化逻辑
        out.defaultWriteObject();  // 默认序列化
        out.writeUTF(name);
    }
    
    private void readObject(java.io.ObjectInputStream in)
        throws IOException, ClassNotFoundException {
        // 自定义反序列化逻辑
        in.defaultReadObject();  // 默认反序列化
        name = in.readUTF();
    }
}
```

### Parcelable 实现

```java
public class User implements Parcelable {
    
    private String name;
    private int age;
    private String email;
    
    // 构造方法
    public User() {}
    
    public User(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    // 从 Parcel 读取的构造方法
    protected User(Parcel in) {
        name = in.readString();
        age = in.readInt();
        email = in.readString();
    }
    
    // 必须实现：创建 Creator
    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }
        
        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };
    
    // 必须实现：描述内容
    @Override
    public int describeContents() {
        return 0;  // 只有包含文件描述符时才返回非0
    }
    
    // 必须实现：写入数据
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeInt(age);
        dest.writeString(email);
    }
    
    // Getter/Setter...
}
```