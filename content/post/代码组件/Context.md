# Context

> https://juejin.cn/post/7582039917217988608

# Context（上下文）核心知识点与应用场景

## 一、Context 的类型与层级关系

| 类型                          | 获取方式                   | 生命周期                                | 内存泄漏风险       | 典型使用场景                           |
| ----------------------------- | -------------------------- | --------------------------------------- | ------------------ | -------------------------------------- |
| **Application Context**       | `getApplicationContext()`  | 应用全局（从启动到终止）                | 低                 | 全局单例、数据库、工具类、资源访问     |
| **Activity Context**          | `this`（在Activity中）     | Activity生命周期（onCreate到onDestroy） | 高（引用Activity） | UI操作、启动Activity、对话框、主题资源 |
| **Service Context**           | `this`（在Service中）      | Service生命周期                         | 中                 | 后台任务、绑定服务、通知               |
| **BroadcastReceiver Context** | `context`参数（onReceive） | 短暂（onReceive期间）                   | 低                 | 广播处理（注意不能用于UI）             |

**继承关系**：`ContextWrapper ← Context` → `ContextThemeWrapper`（Activity）→ `Activity`

## 二、Context 的核心能力

| 能力类别     | 具体方法/功能                                          | 说明与注意事项                            |
| ------------ | ------------------------------------------------------ | ----------------------------------------- |
| **资源访问** | `getResources()`, `getAssets()`                        | 访问res、assets目录资源                   |
| **组件启动** | `startActivity()`, `startService()`, `sendBroadcast()` | Activity/Service/广播的启动               |
| **系统服务** | `getSystemService()`                                   | 获取系统服务（Window、LayoutInflater等）  |
| **文件访问** | `getFilesDir()`, `getCacheDir()`, `openFileOutput()`   | 访问应用私有目录                          |
| **权限检查** | `checkPermission()`                                    | 运行时权限验证                            |
| **主题样式** | `getTheme()`, `setTheme()`                             | Activity特有，Application Context无此能力 |
| **UI相关**   | `getWindow()`, `getLayoutInflater()`                   | **仅Activity可用**，其他Context调用会异常 |

## 三、应用场景决策表

| 你需要做什么         | 推荐使用的 Context                      | 原因与注意事项                                             |
| -------------------- | --------------------------------------- | ---------------------------------------------------------- |
| **创建单例或工具类** | Application Context                     | 生命周期长，避免内存泄漏                                   |
| **弹出 Toast**       | Application Context 或 Activity Context | 两者均可，但某些定制Toast需Activity                        |
| **启动 Activity**    | Activity Context                        | Application Context需加`FLAG_ACTIVITY_NEW_TASK`标志        |
| **显示 Dialog**      | **必须用 Activity Context**             | Application Context会导致`WindowManager$BadTokenException` |
| **布局填充**         | 对应组件的 Context                      | 保持主题一致性，通常用Activity                             |
| **绑定 Service**     | Activity Context                        | 便于生命周期管理                                           |
| **访问资源**         | 任意 Context                            | Application Context通常足够                                |
| **发送广播**         | 任意 Context                            | 根据广播范围选择                                           |
| **访问数据库**       | Application Context                     | 生命周期匹配数据库实例                                     |

## 四、常见错误与最佳实践

### ❌ 常见错误
```java
// 错误1：在Application Context中创建Dialog
Dialog dialog = new Dialog(getApplicationContext()); // 崩溃！

// 错误2：静态成员持有Activity引用
public class Utils {
    private static Context sContext; // 可能内存泄漏
}

// 错误3：混淆Context类型使用
```

### ✅ 最佳实践

| 场景             | 正确做法                                            |
| ---------------- | --------------------------------------------------- |
| **工具类初始化** | 使用Application Context并注意内存管理               |
| **Adapter中**    | 传递Activity Context或使用`getApplicationContext()` |
| **Fragment中**   | 优先使用`requireContext()`（非空保证）              |
| **长期任务**     | 使用Application Context或清理引用                   |
| **匿名内部类**   | 使用弱引用或Application Context                     |

## 五、内存泄漏与解决方案

| 泄漏场景             | 原因                      | 解决方案                |
| -------------------- | ------------------------- | ----------------------- |
| **单例持有Activity** | 生命周期不匹配            | 改用Application Context |
| **静态View引用**     | View绑定Activity Context  | 及时置空或使用弱引用    |
| **Handler泄漏**      | Handler隐式持有外部类引用 | 使用静态Handler+弱引用  |
| **Thread/Runnable**  | 持有外部Context           | 传递Application Context |

## 六、API兼容性要点

| Context方法                             | 最低API | 注意           |
| --------------------------------------- | ------- | -------------- |
| `createDeviceProtectedStorageContext()` | API 24  | 设备保护存储   |
| `isDeviceProtectedStorage()`            | API 24  | 检查存储类型   |
| `getNoBackupFilesDir()`                 | API 21  | 非备份文件目录 |
| `getCodeCacheDir()`                     | API 21  | 代码缓存目录   |

## 七、快速参考决策流程图

```
开始使用Context
      ↓
需要UI相关操作？ → 是 → 使用Activity Context
      ↓否
操作与特定界面绑定？ → 是 → 使用对应组件的Context
      ↓否
生命周期需匹配应用？ → 是 → 使用Application Context
      ↓否
短期/临时使用？ → 是 → 使用当前可用Context
      ↓否
默认选择：Application Context（最安全）
```

## 八、一句话总结

**Application Context 用于全局和生命周期长的任务，Activity Context 用于UI相关操作，根据组件生命周期选择以避免泄漏和崩溃。**

> 提示：在不确定时，优先考虑**生命周期匹配**和**内存安全**，多数工具类场景使用`getApplicationContext()`是最安全的选择。





# ContextWrapper

# ContextWrapper 核心总结

## 一、本质与设计目的

| 维度         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| **本质**     | Context的**包装器**（Wrapper/Decorator模式）                 |
| **设计目的** | **不实现具体功能，只提供委托机制**，让子类可以重写/增强特定功能 |
| **关键特点** | 所有方法都委托给内部`mBase` Context对象                      |
| **自身实现** | **几乎为零**，完全依赖被包装的Context                        |

## 二、类关系与结构

```
Context（抽象接口）
    ↑
ContextWrapper（包装器，委托所有方法给mBase）
    ↑
ContextThemeWrapper（添加主题支持）
    ↑
Activity/Service/Application（具体实现）
```

## 三、核心方法机制

```java
public class ContextWrapper extends Context {
    Context mBase;  // 被包装的真实Context
    
    // 所有方法都简单委托
    public Resources getResources() {
        return mBase.getResources();
    }
    
    public void startActivity(Intent intent) {
        mBase.startActivity(intent);
    }
    
    // ... 其他60+方法全部类似委托
}
```

## 四、实际应用场景

| 场景           | 使用方式                     | 目的/好处                         |
| -------------- | ---------------------------- | --------------------------------- |
| **Mock测试**   | 创建测试用的ContextWrapper   | 模拟真实Context行为，隔离依赖     |
| **功能增强**   | 继承并重写特定方法           | 添加日志、监控、权限检查等AOP功能 |
| **兼容性处理** | 包装旧Context，提供新API实现 | 在不修改原有代码情况下增加新功能  |
| **安全性包装** | 限制某些方法的访问           | 创建受限Context给第三方组件使用   |

## 五、具体使用示例

### 1. **监控增强型Wrapper**
```java
public class MonitoringContextWrapper extends ContextWrapper {
    public MonitoringContextWrapper(Context base) {
        super(base);
    }
    
    @Override
    public void startActivity(Intent intent) {
        // 添加监控逻辑
        Log.d("ContextMonitor", "启动Activity: " + intent.getComponent());
        Analytics.track("activity_start", intent);
        
        // 调用原始功能
        super.startActivity(intent);
    }
    
    @Override
    public ComponentName startService(Intent service) {
        // 服务启动监控
        Log.d("ContextMonitor", "启动Service: " + service.getComponent());
        return super.startService(service);
    }
}
```

### 2. **权限限制型Wrapper**
```java
public class RestrictedContextWrapper extends ContextWrapper {
    private List<String> mAllowedPermissions;
    
    public RestrictedContextWrapper(Context base, List<String> allowedPermissions) {
        super(base);
        mAllowedPermissions = allowedPermissions;
    }
    
    @Override
    public int checkPermission(String permission, int pid, int uid) {
        // 只允许特定权限
        if (!mAllowedPermissions.contains(permission)) {
            return PackageManager.PERMISSION_DENIED;
        }
        return super.checkPermission(permission, pid, uid);
    }
    
    @Override
    public void startActivity(Intent intent) {
        // 限制只能启动特定包名的Activity
        if (!intent.getComponent().getPackageName().equals("com.trusted.app")) {
            throw new SecurityException("不允许启动外部应用");
        }
        super.startActivity(intent);
    }
}
```

### 3. **单元测试Mock Wrapper**
```java
public class MockContextWrapper extends ContextWrapper {
    private Map<String, Object> mMockValues = new HashMap<>();
    
    public MockContextWrapper(Context base) {
        super(base);
    }
    
    // 设置模拟返回值
    public void mockGetSystemService(String name, Object service) {
        mMockValues.put("service:" + name, service);
    }
    
    @Override
    public Object getSystemService(String name) {
        // 返回模拟的系统服务
        String key = "service:" + name;
        if (mMockValues.containsKey(key)) {
            return mMockValues.get(key);
        }
        return super.getSystemService(name);
    }
    
    @Override
    public File getFilesDir() {
        // 返回测试用的临时目录
        return new File("/data/data/test/files");
    }
}
```

## 六、与直接继承Context的对比

| 方式                        | 优点                                                         | 缺点                                                 | 适用场景                |
| --------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- | ----------------------- |
| **继承ContextWrapper**      | 1. 只需重写关注的方法<br>2. 自动获得完整Context功能<br>3. 符合开闭原则 | 1. 无法改变基类行为（只能包装）<br>2. 多一层调用开销 | **功能增强、AOP、测试** |
| **直接实现Context接口**     | 1. 完全控制所有实现<br>2. 无委托开销                         | 1. 需要实现60+方法<br>2. 工作量大易出错              | **特殊需求、框架开发**  |
| **组合模式（持有Context）** | 1. 更灵活的控制<br>2. 可动态更换被包装对象                   | 1. 需要手动转发方法<br>2. 代码量较大                 | **动态代理、复杂包装**  |

## 七、在Android系统中的应用

| 系统类                  | 包装目的                   | 实现方式                                  |
| ----------------------- | -------------------------- | ----------------------------------------- |
| **ContextThemeWrapper** | 添加主题/样式支持          | 包装Context + 主题相关方法                |
| **MOCK Context**        | 测试框架提供模拟环境       | 包装并重写资源、服务等方法                |
| **TintContextWrapper**  | 支持AppCompat的着色功能    | 包装并重写getResources()返回TintResources |
| **Application** 的基类  | 实际是ContextWrapper的子类 | 通过包装获得系统功能                      |

## 八、使用原则与注意事项

### ✅ **应该使用ContextWrapper的情况**
1. 需要**在不修改原Context情况下增加功能**
2. 进行**单元测试Mock**
3. 实现**横切关注点**（日志、监控、权限检查）
4. **逐步重构**时作为中间层

### ❌ **不应使用ContextWrapper的情况**
1. 需要**完全不同的Context行为**
2. **性能敏感**的场景（多一层委托开销）
3. 简单的Context传递（直接使用原Context即可）

### ⚠️ **注意事项**
```java
// 注意1：确保正确传递被包装的Context
public class MyWrapper extends ContextWrapper {
    public MyWrapper(Context base) {
        super(base);  // 必须调用super(base)
    }
}

// 注意2：包装链不宜过长
Context c1 = new OriginalContext();
Context c2 = new Wrapper1(c1);
Context c3 = new Wrapper2(c2);  // 链式包装影响性能

// 注意3：避免循环引用
class BadWrapper extends ContextWrapper {
    private Context mBadReference;
    
    public BadWrapper(Context base) {
        super(base);
        mBadReference = this;  // 错误！循环引用
    }
}
```

## 九、一句话总结

**ContextWrapper是Context的装饰器，通过委托机制让子类可以零成本地增强或修改特定Context功能，是AOP编程和测试的利器，但本身不实现具体功能。**

> 提示：除非需要Mock或增强特定Context方法，否则开发者很少需要直接使用ContextWrapper，系统组件如Activity/Service已基于它构建。