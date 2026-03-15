# ANR

## 一、什么是 ANR？

**ANR** 是 **Application Not Responding**（应用程序无响应）的缩写。这是 Android 系统的一种保护机制，当应用**长时间阻塞主线程**导致用户无法操作时，系统会弹出一个对话框，让用户选择是等待还是强制关闭应用。

**简单理解**：用户操作应用时，如果应用超过一定时间没有响应，Android 系统就会弹出 "XXX应用无响应" 的提示框，这就是 ANR。

**ANR 对话框示例**：

```markdown
应用未响应
"XXX应用" 无响应。要将其关闭吗？

[等待] [确定]
```

------

## 二、ANR 的触发条件

系统为不同类型的操作设置了不同的超时阈值：

| 触发场景              | 超时时间                               | 说明                                               |
| --------------------- | -------------------------------------- | -------------------------------------------------- |
| **按键或触摸事件**    | **5秒**                                | Activity 的主线程在5秒内没有处理完输入事件         |
| **BroadcastReceiver** | **前台广播：10秒** **后台广播：60秒**  | BroadcastReceiver 的 `onReceive()`方法执行时间过长 |
| **Service**           | **前台服务：20秒** **后台服务：200秒** | Service 的各个生命周期方法执行超时                 |
| **ContentProvider**   | **10秒**                               | ContentProvider 的响应超时                         |
| **应用启动**          | **冷启动：10-15秒** **温启动：5秒**    | 应用启动时间过长                                   |

**注意**：这些时间阈值可能因 Android 版本和设备制造商而略有不同。

------

## 三、为什么会发生 ANR？

核心原因只有一个：**主线程被阻塞**。但具体表现多样：

### 1. **主线程执行耗时操作**

```java
// ❌ 错误示例：在主线程执行耗时操作
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
    // 这些操作会阻塞主线程
    heavyCalculation();      // 复杂计算
    readLargeFile();         // 大文件读写
    networkRequest();        // 同步网络请求
    databaseOperation();     // 大量数据库操作
}
```

### 2. **主线程等待锁**

```java
// 死锁或锁竞争
private final Object lock = new Object();

// 线程A（主线程）
public void onMainThread() {
    synchronized(lock) {
        // 持有锁，等待子线程完成
        threadB.join();  // 这里会阻塞
    }
}

// 线程B（子线程）
public void onWorkerThread() {
    synchronized(lock) {  // 尝试获取锁，但被主线程持有
        // 永远无法执行到这里
    }
}
```

### 3. **Binder 调用超时**

```java
// 跨进程调用耗时过长
public void callRemoteService() {
    // 调用其他进程的服务，如果对方处理时间过长
    IBinder binder = ...;
    IRemoteService service = IRemoteService.Stub.asInterface(binder);
    String result = service.timeConsumingOperation();  // 耗时操作
    // 主线程在这里等待，可能触发 ANR
}
```

### 4. **广播接收器阻塞**

```java
// ❌ BroadcastReceiver 中执行耗时操作
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 这些操作不应该在 onReceive 中执行
        try {
            Thread.sleep(15000);  // 休眠15秒 → ANR!
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // 或者执行其他耗时操作
        processLargeData();
    }
}
```

------

## 四、如何检测和分析 ANR？

### 1. **Android Studio 监控**

```markdown
View → Tool Windows → Logcat
过滤关键字：ANR、Input dispatching、Slow operation
```

### 2. **ANR 日志文件位置**

```markdown
// 系统会生成 ANR 日志文件
/data/anr/traces.txt           # 所有进程的堆栈信息
/data/anr/anr_*.txt           # 具体的 ANR 报告
```

### 3. **获取 ANR 日志**

```bash
# 通过 adb 获取
adb pull /data/anr/traces.txt

# 如果有 root 权限
adb shell cat /data/anr/traces.txt > traces.txt
```

### 4. **ANR 日志结构分析**

一个典型的 ANR 日志包含：

```markdown
----- pid 12345 at 2023-10-01 10:30:00 -----
Cmd line: com.example.myapp  # 发生 ANR 的应用

DALVIK THREADS (主线程状态):
"main" prio=5 tid=1 Native
  | group="main" sCount=1 dsCount=0 flags=1 obj=0x72e3f2a0 self=0x7f8c0a4000
  | sysTid=12345 nice=-10 cgrp=default sched=0/0 handle=0x7fa1a4a4a0
  | state=S schedstat=( 123456789 987654321 1234 ) utm=12 stm=34 core=1 HZ=100
  | stack=0x7fc34a2000-0x7fc34a4000 stackSize=8192KB
  | held mutexes=
  
  # 关键信息：主线程在等待什么？
  native: #00 pc 000000000006aabc  /system/lib64/libc.so (__epoll_pwait+8)
  native: #01 pc 0000000000014d88  /system/lib64/libutils.so (android::Looper::pollInner(int)+148)
  native: #02 pc 0000000000014c40  /system/lib64/libutils.so (android::Looper::pollOnce(int, int*, int*, void**)+56)
  native: #03 pc 000000000012b2dc  /system/lib64/libandroid_runtime.so (android::NativeMessageQueue::pollOnce(_JNIEnv*, _jobject*, int)+24)
  at android.os.MessageQueue.nativePollOnce(Native method)
  at android.os.MessageQueue.next(MessageQueue.java:336)
  at android.os.Looper.loop(Looper.java:174)
  at android.app.ActivityThread.main(ActivityThread.java:7356)
  at java.lang.reflect.Method.invoke(Native method)
  at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:492)
  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:930)
```

**关键线索**：

- 

  查找 **"main"** 线程（主线程）的状态

- 

  查看主线程的 **堆栈调用**（stack trace）

- 

  关注 **线程状态**：`Native`、`Blocked`、`Waiting`、`Timed_Waiting`

------

## 五、ANR 的完整解决方案

### 1. **避免主线程耗时操作**

#### 使用 AsyncTask（已废弃，但了解原理）

```java
private class DownloadTask extends AsyncTask<String, Integer, Bitmap> {
    @Override
    protected Bitmap doInBackground(String... urls) {
        // 在后台线程执行
        return downloadImage(urls[0]);
    }
    
    @Override
    protected void onPostExecute(Bitmap result) {
        // 在主线程更新UI
        imageView.setImageBitmap(result);
    }
}
```

#### 使用 HandlerThread

```java
// 创建后台线程
HandlerThread handlerThread = new HandlerThread("WorkerThread");
handlerThread.start();

Handler workerHandler = new Handler(handlerThread.getLooper());

// 提交任务到后台线程
workerHandler.post(() -> {
    // 在后台线程执行耗时操作
    Bitmap bitmap = processImage();
    
    // 切回主线程更新UI
    mainHandler.post(() -> {
        imageView.setImageBitmap(bitmap);
    });
});
```

#### 使用 Kotlin 协程（推荐）

```kotlin
// 在 ViewModel 中
viewModelScope.launch {
    // 在主线程开始
    showLoading()
    
    // 切换到 IO 线程执行耗时操作
    val result = withContext(Dispatchers.IO) {
        repository.fetchData()  // 网络请求、数据库操作等
    }
    
    // 自动切回主线程
    updateUI(result)
    hideLoading()
}

// 或者使用 Flow
fun getData(): Flow<List<Data>> = flow {
    val data = withContext(Dispatchers.IO) {
        dao.loadData()
    }
    emit(data)
}.flowOn(Dispatchers.IO)  // 指定上游执行线程

// 在 UI 层收集
lifecycleScope.launch {
    viewModel.getData().collect { data ->
        // 在主线程更新
        adapter.submitList(data)
    }
}
```

#### 使用 RxJava

```java
Observable.fromCallable(() -> {
    // 在后台线程执行
    return heavyComputation();
})
.subscribeOn(Schedulers.io())  // 指定执行线程
.observeOn(AndroidSchedulers.mainThread())  // 指定观察线程
.subscribe(result -> {
    // 在主线程处理结果
    updateUI(result);
}, error -> {
    handleError(error);
});
```

#### 使用 WorkManager（长期后台任务）

```kotlin
val uploadWorkRequest = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(uploadWorkRequest)
```

### 2. **优化 BroadcastReceiver**

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        // ❌ 不要在 onReceive 中执行耗时操作
        // heavyOperation();  
        
        // ✅ 正确做法：启动服务或工作线程
        if (Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            // 启动 IntentService 处理
            Intent serviceIntent = new Intent(context, MyIntentService.class);
            context.startService(serviceIntent);
            
            // 或者使用 WorkManager
            val workRequest = OneTimeWorkRequestBuilder<BootWorker>().build()
            WorkManager.getInstance(context).enqueue(workRequest)
            
            // 或者使用 goAsync()（API 11+）
            final PendingResult result = goAsync();
            new Thread(() -> {
                // 在后台线程执行
                processData();
                
                // 完成广播处理
                result.finish();
            }).start();
        }
    }
}
```

### 3. **优化 Service**

```java
public class MyService extends Service {
    // ❌ 避免在 Service 生命周期方法中执行耗时操作
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        // 不要在这里执行耗时操作
        
        // ✅ 使用 IntentService（已废弃但仍有参考价值）
        // 或者使用 JobIntentService
        
        // ✅ 现代方式：使用 WorkManager 或前台服务
        startForeground(NOTIFICATION_ID, createNotification());
        
        // 在后台线程执行任务
        Executors.newSingleThreadExecutor().execute(() -> {
            performLongRunningTask();
            
            // 任务完成后停止服务
            stopSelf();
        });
        
        return START_STICKY;
    }
}

// 使用 JobIntentService（兼容性更好）
public class MyJobService extends JobIntentService {
    @Override
    protected void onHandleWork(@NonNull Intent intent) {
        // 在后台线程执行，不会阻塞主线程
        processData();
    }
}
```

### 4. **优化 Activity 和 Fragment**

```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // ❌ 避免在 onCreate 中执行耗时操作
        // loadHeavyData()
        
        // ✅ 延迟加载或使用 ViewModel
        viewModel.loadData()
        
        // ✅ 使用 ViewStub 延迟加载复杂布局
        val viewStub = findViewById<ViewStub>(R.id.stub_complex_view)
        viewStub?.setOnInflateListener { stub, inflated ->
            // 布局被需要时才加载
            initComplexView(inflated)
        }
        
        // ✅ 使用 IdleHandler 在空闲时执行
        Looper.myQueue().addIdleHandler {
            // 主线程空闲时执行
            performLowPriorityTask()
            false // 返回 false 表示只执行一次
        }
    }
    
    // 处理配置变化
    override fun onConfigurationChanged(newConfig: Configuration) {
        super.onConfigurationChanged(newConfig)
        // 避免在此进行耗时操作
    }
}
```

### 5. **优化 ContentProvider**

```java
public class MyContentProvider extends ContentProvider {
    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
                       String[] selectionArgs, String sortOrder) {
        // ❌ 避免复杂的查询
        
        // ✅ 使用异步查询
        final CountDownLatch latch = new CountDownLatch(1);
        final Cursor[] result = new Cursor[1];
        
        new Thread(() -> {
            result[0] = database.query(...);
            latch.countDown();
        }).start();
        
        try {
            latch.await(5, TimeUnit.SECONDS);  // 设置超时
        } catch (InterruptedException e) {
            return null;
        }
        
        return result[0];
        
        // ✅ 或者使用 Room 的异步查询
        // return LiveData 或 Flow
    }
}
```

### 6. **优化 Binder 通信**

```java
// 如果频繁进行跨进程调用，考虑：
// 1. 批量处理请求，减少调用次数
// 2. 使用异步回调，避免同步等待
// 3. 使用缓存，减少重复调用

public class RemoteService extends Service {
    private final IBinder binder = new IRemoteService.Stub() {
        @Override
        public String getData() {
            // 确保这里不执行耗时操作
            return cachedData;  // 返回缓存数据
        }
        
        @Override
        public void fetchDataAsync(ICallback callback) {
            // 异步获取数据
            Executors.newSingleThreadExecutor().execute(() -> {
                String data = fetchFromNetwork();
                try {
                    callback.onResult(data);
                } catch (RemoteException e) {
                    e.printStackTrace();
                }
            });
        }
    };
}
```

### 7. **使用 StrictMode 检测潜在问题**

```java
// 在 Application 或 Activity 的 onCreate 中启用
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
        .detectAll()  // 检测所有问题
        .penaltyLog()  // 记录日志
        .penaltyDeath()  // 在 DEBUG 版本中崩溃，便于发现问题
        .build());
    
    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
        .detectLeakedSqlLiteObjects()
        .detectLeakedClosableObjects()
        .penaltyLog()
        .build());
}
```

### 8. **优化应用启动**

```kotlin
// Application 类优化
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        
        // ❌ 避免在 Application.onCreate 中执行耗时初始化
        // initHeavyLibrary();
        
        // ✅ 延迟初始化或使用后台线程
        Executors.newSingleThreadExecutor().execute {
            initThirdPartyLibrary()
        }
        
        // ✅ 使用 Startup 库进行组件初始化优化
        AppInitializer.getInstance(this)
            .initializeComponent(MyInitializer::class.java)
    }
}

// 使用 Jetpack Startup 库
class MyInitializer : Initializer<MyLibrary> {
    override fun create(context: Context): MyLibrary {
        // 初始化工作
        return MyLibrary.init(context)
    }
    
    override fun dependencies(): List<Class<out Initializer<*>>> {
        // 定义依赖关系
        return emptyList()
    }
}
```

### 9. **使用性能分析工具**

#### **Android Profiler**

```markdown
1. 打开 Android Studio → View → Tool Windows → Profiler
2. 监控 CPU、内存、网络使用情况
3. 查看主线程状态，找出耗时操作
```

#### **Systrace**

```bash
# 生成 trace 文件
python systrace.py -t 10 sched gfx view wm am app -o trace.html

# 分析主线程阻塞
1. 查找主线程的"长帧"
2. 查看锁等待情况
3. 分析 Binder 调用耗时
```

#### **Perfetto**

```bash
# 更强大的性能分析工具
adb shell perfetto --config :test --out /data/misc/perfetto-traces/trace
```

------

## 六、ANR 监控和线上统计

### 1. **自定义 ANR 监控**

```java
public class ANRWatchDog extends Thread {
    private static final int TIMEOUT = 5000; // 5秒
    private Handler mainHandler = new Handler(Looper.getMainLooper());
    private volatile boolean isRunning = true;
    private volatile long lastTick = 0;
    
    @Override
    public void run() {
        while (isRunning) {
            lastTick = System.currentTimeMillis();
            mainHandler.post(() -> {
                // 主线程收到tick，更新lastTick
                lastTick = System.currentTimeMillis();
            });
            
            try {
                Thread.sleep(TIMEOUT);
            } catch (InterruptedException e) {
                break;
            }
            
            // 如果超时，说明主线程卡住了
            if (System.currentTimeMillis() - lastTick > TIMEOUT) {
                // 收集堆栈信息
                StringBuilder stackTrace = new StringBuilder();
                for (StackTraceElement element : Looper.getMainLooper()
                        .getThread().getStackTrace()) {
                    stackTrace.append(element.toString()).append("\n");
                }
                
                // 上报到监控平台
                reportANR(stackTrace.toString());
            }
        }
    }
    
    public void stopWatching() {
        isRunning = false;
        interrupt();
    }
}
```

### 2. **使用第三方监控 SDK**

```gradle
dependencies {
    // Bugly ANR 监控
    implementation 'com.tencent.bugly:crashreport:latest'
    
    // Firebase Performance Monitoring
    implementation 'com.google.firebase:firebase-perf:latest'
    
    // 或使用其他 APM 工具
}
```

### 3. **ANR 日志上报策略**

```kotlin
class ANRReporter {
    fun reportANR(trace: String, context: Context) {
        // 1. 保存到本地
        saveToLocal(trace)
        
        // 2. 在下次启动时上报
        if (shouldUpload()) {
            uploadToServer(trace)
        }
        
        // 3. 触发自定义处理
        EventBus.getDefault().post(AnrEvent(trace))
    }
    
    private fun saveToLocal(trace: String) {
        val file = File(context.filesDir, "anr_traces.txt")
        file.appendText("${Date()}\n$trace\n\n")
    }
}
```

------

## 七、最佳实践总结

### 1. **主线程规则**

- 

  **只做 UI 操作**：布局、绘制、事件处理

- 

  **快速完成**：任何操作都不应超过 16ms（60 FPS）

- 

  **异步处理**：网络、数据库、文件、计算等全部放到后台线程

### 2. **代码编写规范**

```kotlin
// ✅ 好的实践
lifecycleScope.launch {
    // 显示加载状态（主线程）
    showLoading()
    
    // 耗时操作（后台线程）
    val result = withContext(Dispatchers.IO) {
        performNetworkRequest()
    }
    
    // 更新 UI（主线程）
    updateUI(result)
    hideLoading()
}

// ❌ 避免的写法
button.setOnClickListener {
    // 直接在点击事件中执行耗时操作
    val result = performNetworkRequestBlocking()  // 同步阻塞
    updateUI(result)
}
```

### 3. **架构建议**

- 

  **使用 MVVM/MVI 架构**：分离业务逻辑和 UI

- 

  **使用 Repository 模式**：统一数据访问层

- 

  **使用响应式编程**：RxJava、Flow、LiveData

- 

  **合理使用缓存**：减少重复计算和网络请求

### 4. **监控和告警**

- 

  **开发阶段**：启用 StrictMode，使用 Profiler

- 

  **测试阶段**：Monkey 测试，ANR 专项测试

- 

  **线上监控**：集成 APM 工具，设置 ANR 告警阈值（如 >1%）

### 5. **常见性能陷阱**

```java
// 1. 过度绘制
// 解决方案：使用 ViewStub、Merge、减少布局层级

// 2. 内存泄漏导致 GC 频繁
// 解决方案：使用 LeakCanary，注意生命周期

// 3. 同步锁竞争
// 解决方案：使用并发集合、减小锁粒度

// 4. 频繁创建对象
// 解决方案：对象池、复用、避免在循环中创建对象

// 5. 大图片处理
// 解决方案：图片压缩、采样、使用 Glide/Picasso
```

### 6. **测试 ANR**

```kotlin
// 编写 ANR 测试用例
@Test
fun testANRScenario() {
    val scenario = ActivityScenario.launch(MainActivity::class.java)
    
    // 模拟主线程耗时操作
    scenario.onActivity { activity ->
        // 这里应该触发 ANR（仅在测试中）
        Thread.sleep(10000)  // 休眠10秒
    }
    
    // 验证 ANR 处理逻辑
    // ...
}
```

------

## 八、高级调试技巧

### 1. **分析 traces.txt**

```bash
# 1. 查找主线程
grep -A 50 '"main"' traces.txt

# 2. 查找 BLOCKED 状态
grep -B5 -A5 "Blocked" traces.txt

# 3. 查找等待的锁
grep -B10 -A10 "waiting to lock" traces.txt

# 4. 查找 Binder 调用
grep -B5 -A5 "Binder" traces.txt
```

### 2. **使用 Debug 工具**

```java
// 在代码中插入调试点
Debug.startMethodTracing("anr_trace");
// ... 可疑代码段 ...
Debug.stopMethodTracing();

// 生成 trace 文件，在 Android Studio 中分析
```

### 3. **ANR 优化检查清单**

- 

  [ ] 所有网络请求都在后台线程

- 

  [ ] 数据库操作都在后台线程

- 

  [ ] 文件读写都在后台线程

- 

  [ ] 复杂计算都在后台线程

- 

  [ ] BroadcastReceiver 快速返回

- 

  [ ] Service 生命周期方法快速返回

- 

  [ ] 避免主线程同步锁

- 

  [ ] 使用异步 Binder 调用

- 

  [ ] 优化布局层次和绘制

- 

  [ ] 监控并优化启动时间

------

## 总结

ANR 是 Android 开发中最常见的性能问题之一，根本原因总是**主线程被阻塞**。解决 ANR 需要：

1. 

   **预防为主**：严格遵守主线程只做 UI 操作的原则

2. 

   **工具辅助**：使用 Profiler、StrictMode 等工具提前发现问题

3. 

   **异步处理**：合理使用线程、协程、RxJava 等异步方案

4. 

   **架构优化**：采用合理的架构模式，分离关注点

5. 

   **监控上报**：建立完善的线上监控体系