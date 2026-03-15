# Android 启动模式

## 一、启动模式分类概览

![image-20260113165819860](C:\Users\TSCQ\AppData\Roaming\Typora\typora-user-images\image-2026011316581986023556.png)

## 二、冷启动（Cold Start）

### 定义

**冷启动**是指应用**完全不在内存中**，系统需要**从零开始**创建应用进程并初始化所有组件的过程。

### 触发场景

```java
// 冷启动的常见场景：
1. 应用首次安装后启动
2. 应用被完全杀死后（手动或系统清理）
3. 设备重启后首次启动
4. 应用被设置为"不保留活动"后启动
```

### 冷启动的执行流程

```markdown
用户点击应用图标
    ↓
Zygote 进程 fork 出应用进程
    ↓
创建 Application 对象
    ↓
调用 Application.onCreate()
    ↓
创建主 Activity
    ↓
调用 Activity.onCreate() → onStart() → onResume()
    ↓
View 系统：测量 → 布局 → 绘制
    ↓
显示启动窗口（Splash/白屏）
    ↓
首帧渲染完成
    ↓
启动窗口消失，显示应用内容
```

### 代码层面的冷启动过程

```java
// Application 初始化
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        // ✅ 这里初始化全局资源
        // ❌ 避免在这里执行耗时操作
        initThirdPartySDKs();  // 耗时操作应该异步
    }
}

// 主 Activity
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 冷启动时 savedInstanceState 为 null
        if (savedInstanceState == null) {
            // 首次创建 Activity
            initData();
        }
    }
    
    @Override
    protected void onStart() {
        super.onStart();
        // 开始处理业务逻辑
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        // 应用完全可见
    }
}
```

### 冷启动的视觉体验

```markdown
时间线：
0ms      用户点击图标
         ↓
        显示系统默认启动窗口（白屏/黑屏）
         ↓
200ms    Application.onCreate() 执行
         ↓
500ms    Activity.onCreate() 执行
         ↓
800ms    首帧渲染完成
         ↓
1000ms   启动窗口消失，显示应用内容
```



## 三、热启动（Hot Start）

### 定义

**热启动**是指应用**完全在内存中**（在后台运行），用户将其切换到前台的过程。

### 触发场景

```java
// 热启动的常见场景：
1. 按 Home 键回到桌面，再点击应用图标
2. 从最近任务列表切换回应用
3. 从其他应用返回当前应用
4. 从通知栏点击返回应用
```

### 热启动的执行流程

```markdown
用户切换回应用
    ↓
应用进程已存在
    ↓
恢复 Activity 栈顶的 Activity
    ↓
调用 Activity.onRestart() → onStart() → onResume()
    ↓
立即显示应用界面
    ↓
无需重新初始化 Application
```

### 代码层面的热启动过程

```java
public class MainActivity extends AppCompatActivity {
    @Override
    protected void onRestart() {
        super.onRestart();
        // 热启动时调用
        // 恢复数据、刷新UI等
    }
    
    @Override
    protected void onStart() {
        super.onStart();
        // 从后台回到前台
    }
    
    @Override
    protected void onResume() {
        super.onResume();
        // 重新获取焦点
    }
    
    @Override
    protected void onPause() {
        super.onPause();
        // 应用退到后台
    }
    
    @Override
    protected void onStop() {
        super.onStop();
        // Activity 不可见
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // Activity 被销毁
    }
}
```

## 四、温启动（Warm Start）

### 定义

**温启动**是介于冷启动和热启动之间的情况，应用进程存在但 Activity 已被销毁，需要重建 Activity。

### 触发场景

```java
// 温启动的常见场景：
1. 系统内存不足，杀死了后台 Activity
2. 开发者选项中开启"不保留活动"
3. 应用长时间在后台，系统回收资源
4. 配置变更（如旋转屏幕）
```

### 温启动的执行流程

```markdown
用户切换回应用
    ↓
应用进程存在
    ↓
但 Activity 被销毁
    ↓
需要重建 Activity
    ↓
调用 Activity.onCreate()（有 savedInstanceState）
    ↓
恢复之前的状态
    ↓
调用 onStart() → onResume()
```

### 代码层面的温启动

```java
public class MainActivity extends AppCompatActivity {
    private static final String KEY_SCROLL_POSITION = "scroll_position";
    private int scrollPosition = 0;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // 温启动时 savedInstanceState 不为 null
        if (savedInstanceState != null) {
            // 恢复之前的状态
            scrollPosition = savedInstanceState.getInt(KEY_SCROLL_POSITION, 0);
            restoreUI();
        } else {
            // 冷启动，初始化新数据
            initData();
        }
    }
    
    @Override
    protected void onSaveInstanceState(@NonNull Bundle outState) {
        super.onSaveInstanceState(outState);
        // 保存状态，供温启动时恢复
        outState.putInt(KEY_SCROLL_POSITION, scrollPosition);
    }
}
```

## 五、三种启动模式的对比

| 维度                       | **冷启动**                    | **热启动**                     | **温启动**                      |
| -------------------------- | ----------------------------- | ------------------------------ | ------------------------------- |
| **进程状态**               | 不存在，需要创建              | 存在，在前台/后台              | 存在，但 Activity 被销毁        |
| **Application.onCreate()** | ✓ 调用                        | ✗ 不调用                       | ✗ 不调用                        |
| **Activity 创建**          | 全新创建                      | 恢复现有                       | 重新创建                        |
| **savedInstanceState**     | null                          | null                           | 有值（可恢复状态）              |
| **生命周期调用**           | onCreate → onStart → onResume | onRestart → onStart → onResume | onCreate → onStart → onResume   |
| **启动时间**               | 慢（1-3秒）                   | 快（< 500ms）                  | 中等（500ms-1.5秒）             |
| **内存使用**               | 高（创建新进程）              | 低（复用进程）                 | 中等（复用进程，重建 Activity） |
| **用户体验**               | 有启动窗口（白屏/黑屏）       | 无缝切换                       | 可能有短暂重建过程              |
| **优化重点**               | 减少初始化时间                | 保持响应性                     | 快速恢复状态                    |

## 六、启动时间测量与分析

### 1. **使用 ADB 测量启动时间**

```bash
# 测量冷启动时间
adb shell am start-activity -W -n com.example.app/.MainActivity

# 输出示例：
Starting: Intent { act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] cmp=com.example.app/.MainActivity }
Status: ok
Activity: com.example.app/.MainActivity
ThisTime: 1654
TotalTime: 1654
WaitTime: 1665

# 参数说明：
# - ThisTime: 最后一个Activity的启动耗时
# - TotalTime: 所有Activity的启动耗时
# - WaitTime: AMS启动Activity的总耗时
```

### 2. **在代码中测量启动时间**

```kotlin
// Application 中记录启动时间
class MyApplication : Application() {
    
    override fun onCreate() {
        super.onCreate()
        
        // 记录应用启动开始时间
        if (isColdStart()) {
            val startTime = System.currentTimeMillis()
            
            // 异步初始化
            initAsync {
                val endTime = System.currentTimeMillis()
                val coldStartTime = endTime - startTime
                Log.d("ColdStart", "冷启动耗时: ${coldStartTime}ms")
                
                // 上报到监控系统
                reportColdStartTime(coldStartTime)
            }
        }
    }
    
    private fun isColdStart(): Boolean {
        // 判断是否是冷启动
        val activityManager = getSystemService(ACTIVITY_SERVICE) as ActivityManager
        val runningAppProcesses = activityManager.runningAppProcesses
        val packageName = packageName
        
        return runningAppProcesses?.none { it.processName == packageName } ?: true
    }
}
```