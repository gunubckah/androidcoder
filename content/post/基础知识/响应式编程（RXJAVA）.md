# 响应式编程（RXJAVA）

## 1. **RxJava 是什么？**

**RxJava** 是一个基于 **观察者模式** 的异步事件处理库，专门用于 **响应式编程**。它是 **ReactiveX** 在 Java 平台上的实现。

### 核心定义：

```markdown
RxJava = 观察者模式 + 迭代器模式 + 函数式编程
```

## 2. **核心概念**

### 2.1 四大天王

| 组件           | 角色            | 功能             |
| -------------- | --------------- | ---------------- |
| **Observable** | 被观察者/事件源 | 发出数据流       |
| **Observer**   | 观察者          | 订阅并处理事件   |
| **Subscriber** | 订阅者          | Observer的实现类 |
| **Operator**   | 操作符          | 对数据流进行变换 |

### 2.2 三种事件

```java
Observable
    .onNext(value)    // 发送数据
    .onError(error)   // 发送错误
    .onComplete()     // 发送完成
```





# RxJava Observable 完全指南

## 一、Observable 核心概念

### 1. **什么是 Observable？**

| 概念             | 描述                | 类比           |
| ---------------- | ------------------- | -------------- |
| **Observable**   | 可观察对象/被观察者 | 生产者/数据源  |
| **Observer**     | 观察者              | 消费者/订阅者  |
| **Subscription** | 订阅关系            | 连接管道       |
| **Emitter**      | 发射器              | 数据发射器     |
| **Disposable**   | 可销毁对象          | 取消订阅的凭证 |

### 2. **Observable 生命周期**

```
Observable.create(emitter -> {
    // onSubscribe
    emitter.onNext("数据1");  // 发射数据
    emitter.onNext("数据2");
    emitter.onComplete();    // 完成
    // 或 emitter.onError(error); // 错误
})
.subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) { }  // 订阅时调用
    
    @Override
    public void onNext(String s) { }          // 接收数据
    
    @Override
    public void onError(Throwable e) { }      // 错误处理
    
    @Override
    public void onComplete() { }              // 完成处理
});
```

## 二、Observable 创建方式

### 1. **基本创建方法**

| 创建方法       | 描述            | 示例                                       |
| -------------- | --------------- | ------------------------------------------ |
| **create()**   | 自定义创建      | `Observable.create(emitter -> {...})`      |
| **just()**     | 直接发射数据    | `Observable.just(1, 2, 3)`                 |
| **fromXxx()**  | 从集合/数组创建 | `Observable.fromArray(array)`              |
| **range()**    | 发射数字范围    | `Observable.range(1, 10)`                  |
| **interval()** | 定时发射        | `Observable.interval(1, TimeUnit.SECONDS)` |
| **timer()**    | 延迟发射        | `Observable.timer(5, TimeUnit.SECONDS)`    |
| **empty()**    | 空Observable    | `Observable.empty()`                       |
| **error()**    | 错误Observable  | `Observable.error(new Exception())`        |
| **never()**    | 永不发射        | `Observable.never()`                       |

### 2. **延迟创建**

| 方法               | 描述           | 特点                         |
| ------------------ | -------------- | ---------------------------- |
| **defer()**        | 延迟创建       | 每次订阅时创建新的Observable |
| **fromCallable()** | 从Callable创建 | 延迟执行，可抛出异常         |

```
// defer 示例：每次订阅获取最新时间
Observable<Long> timeObservable = Observable.defer(() -> 
    Observable.just(System.currentTimeMillis())
);

// fromCallable 示例
Observable<String> callableObservable = Observable.fromCallable(() -> {
    if (Math.random() > 0.5) {
        return "Success";
    } else {
        throw new Exception("Random failure");
    }
});
```

## 三、操作符分类

### 1. **转换操作符（Transformation）**

| 操作符          | 描述              | 示例                                     |
| --------------- | ----------------- | ---------------------------------------- |
| **map()**       | 一对一转换        | `.map(x -> x * 2)`                       |
| **flatMap()**   | 一对多转换，合并  | `.flatMap(x -> Observable.just(x, x*2))` |
| **concatMap()** | 有序的flatMap     | 保持顺序                                 |
| **switchMap()** | 最新优先的flatMap | 取消之前的                               |
| **scan()**      | 累积计算          | `.scan((sum, item) -> sum + item)`       |
| **buffer()**    | 缓冲收集          | `.buffer(5)`                             |
| **window()**    | 窗口分组          | `.window(5)`                             |
| **groupBy()**   | 按条件分组        | `.groupBy(item -> item % 2)`             |

### 2. **过滤操作符（Filtering）**

| 操作符                     | 描述          | 示例                      |
| -------------------------- | ------------- | ------------------------- |
| **filter()**               | 过滤          | `.filter(x -> x > 10)`    |
| **take()**                 | 取前N个       | `.take(5)`                |
| **takeLast()**             | 取后N个       | `.takeLast(5)`            |
| **skip()**                 | 跳过前N个     | `.skip(5)`                |
| **skipLast()**             | 跳过后N个     | `.skipLast(5)`            |
| **distinct()**             | 去重          | `.distinct()`             |
| **distinctUntilChanged()** | 相邻去重      | `.distinctUntilChanged()` |
| **elementAt()**            | 获取指定位置  | `.elementAt(3)`           |
| **first() / last()**       | 获取首个/末个 | `.first(0)`               |

### 3. **组合操作符（Combining）**

| 操作符              | 描述               | 示例                                       |
| ------------------- | ------------------ | ------------------------------------------ |
| **merge()**         | 合并多个Observable | `Observable.merge(obs1, obs2)`             |
| **concat()**        | 顺序连接           | `Observable.concat(obs1, obs2)`            |
| **zip()**           | 按位合并           | `Observable.zip(obs1, obs2, (a,b) -> a+b)` |
| **combineLatest()** | 最新值合并         | 任意发射时合并最新值                       |
| **startWith()**     | 开头插入           | `.startWith(0)`                            |
| **join()**          | 时间窗口连接       | 复杂的时间窗口连接                         |

### 4. **错误处理操作符**

| 操作符                  | 描述                 | 示例                                            |
| ----------------------- | -------------------- | ----------------------------------------------- |
| **onErrorReturn()**     | 错误时返回默认值     | `.onErrorReturn(e -> "default")`                |
| **onErrorResumeNext()** | 错误时切换Observable | `.onErrorResumeNext(Observable.just("backup"))` |
| **retry()**             | 重试                 | `.retry(3)`                                     |
| **retryWhen()**         | 条件重试             | 根据错误类型重试                                |
| **catch()**             | 捕获错误             | `.onErrorReturnItem("error")`                   |

### 5. **工具操作符**

| 操作符              | 描述         | 示例                                         |
| ------------------- | ------------ | -------------------------------------------- |
| **delay()**         | 延迟发射     | `.delay(1, TimeUnit.SECONDS)`                |
| **doOnNext()**      | 副作用处理   | `.doOnNext(item -> log(item))`               |
| **doOnSubscribe()** | 订阅时调用   | `.doOnSubscribe(d -> log("subscribe"))`      |
| **doOnComplete()**  | 完成时调用   | `.doOnComplete(() -> log("complete"))`       |
| **timeout()**       | 超时处理     | `.timeout(5, TimeUnit.SECONDS)`              |
| **subscribeOn()**   | 指定订阅线程 | `.subscribeOn(Schedulers.io())`              |
| **observeOn()**     | 指定观察线程 | `.observeOn(AndroidSchedulers.mainThread())` |

## 四、线程调度（Schedulers）

### 1. **RxJava 内置调度器**

| 调度器                             | 描述          | 适用场景           |
| ---------------------------------- | ------------- | ------------------ |
| **Schedulers.io()**                | I/O操作线程池 | 网络请求、文件读写 |
| **Schedulers.computation()**       | 计算线程池    | CPU密集型计算      |
| **Schedulers.newThread()**         | 创建新线程    | 长时间运行任务     |
| **Schedulers.single()**            | 单一线程      | 顺序执行任务       |
| **Schedulers.trampoline()**        | 当前线程排队  | 避免递归导致栈溢出 |
| **Schedulers.from(executor)**      | 自定义线程池  | 使用现有Executor   |
| **AndroidSchedulers.mainThread()** | Android主线程 | UI更新             |

### 2. **线程调度示例**

```
Observable.create(emitter -> {
    // 在io线程执行耗时操作
    String data = downloadData();
    emitter.onNext(data);
    emitter.onComplete();
})
.subscribeOn(Schedulers.io())          // 指定上游执行线程
.observeOn(AndroidSchedulers.mainThread()) // 指定下游接收线程
.subscribe(data -> {
    // 在主线程更新UI
    updateUI(data);
});
```

## 五、背压（Backpressure）处理

### 1. **背压问题场景**

```
// 快速发射，慢速消费导致背压
Observable.interval(1, TimeUnit.MILLISECONDS)  // 每秒发射1000个
    .observeOn(Schedulers.computation())       // 计算线程处理
    .subscribe(item -> {
        Thread.sleep(100);  // 慢速处理，每秒只能处理10个
        process(item);
    });
```

### 2. **背压策略**

| 策略     | 操作符                   | 描述                |
| -------- | ------------------------ | ------------------- |
| **丢弃** | `onBackpressureDrop()`   | 丢弃无法处理的数据  |
| **缓存** | `onBackpressureBuffer()` | 缓存数据（可能OOM） |
| **最新** | `onBackpressureLatest()` | 只保留最新数据      |
| **错误** | `onBackpressureError()`  | 背压时抛出错误      |

### 3. **Flowable vs Observable**

| 特性         | Observable        | Flowable |
| ------------ | ----------------- | -------- |
| **背压支持** | 不支持            | 支持     |
| **适用场景** | 少量数据（<1000） | 大量数据 |
| **性能**     | 稍快              | 稍慢     |
| **操作符**   | 丰富              | 稍少     |

## 六、订阅管理

### 1. **订阅方式**

```
// 方式1：完整订阅
Disposable disposable = observable.subscribe(
    item -> { /* onNext */ },
    error -> { /* onError */ },
    () -> { /* onComplete */ }
);

// 方式2：简写订阅
Disposable disposable = observable.subscribe(item -> {
    // 只处理onNext，忽略错误
});

// 方式3：使用Observer接口
observable.subscribe(new Observer<String>() {
    private Disposable mDisposable;
    
    @Override
    public void onSubscribe(Disposable d) {
        mDisposable = d;
    }
    
    @Override
    public void onNext(String s) { }
    
    @Override
    public void onError(Throwable e) { }
    
    @Override
    public void onComplete() { }
});
```

### 2. **Disposable 管理**

```
// 1. 单个Disposable管理
private Disposable mDisposable;

private void startRequest() {
    if (mDisposable != null && !mDisposable.isDisposed()) {
        mDisposable.dispose();  // 取消之前的请求
    }
    
    mDisposable = observable.subscribe(...);
}

@Override
protected void onDestroy() {
    super.onDestroy();
    if (mDisposable != null && !mDisposable.isDisposed()) {
        mDisposable.dispose();  // 避免内存泄漏
    }
}

// 2. CompositeDisposable管理多个
private CompositeDisposable mCompositeDisposable = new CompositeDisposable();

private void addSubscription() {
    Disposable disposable1 = observable1.subscribe(...);
    Disposable disposable2 = observable2.subscribe(...);
    
    mCompositeDisposable.add(disposable1);
    mCompositeDisposable.add(disposable2);
}

private void clearAll() {
    mCompositeDisposable.clear();  // 取消所有订阅
}
```

## 七、高级特性

### 1. **Hot vs Cold Observable**

| 类型                | 特点                 | 示例                     |
| ------------------- | -------------------- | ------------------------ |
| **Cold Observable** | 每个订阅者独立数据流 | `Observable.just(1,2,3)` |
| **Hot Observable**  | 多个订阅者共享数据流 | `ConnectableObservable`  |

```
// Cold Observable示例
Observable<Integer> cold = Observable.create(emitter -> {
    System.out.println("Create new data");
    emitter.onNext(1);
    emitter.onNext(2);
    emitter.onComplete();
});

cold.subscribe(i -> System.out.println("Subscriber 1: " + i));
cold.subscribe(i -> System.out.println("Subscriber 2: " + i));
// 输出两次"Create new data"

// Hot Observable示例
ConnectableObservable<Integer> hot = Observable.interval(1, TimeUnit.SECONDS)
    .take(5)
    .publish();  // 转换为Hot Observable
    
hot.connect();  // 开始发射数据

hot.subscribe(i -> System.out.println("Subscriber A: " + i));
Thread.sleep(3000);
hot.subscribe(i -> System.out.println("Subscriber B: " + i));
// B从订阅时开始接收，可能错过之前的数据
```

### 2. **Subject（既是Observable又是Observer）**

| Subject类型         | 特点                   | 适用场景 |
| ------------------- | ---------------------- | -------- |
| **PublishSubject**  | 实时发射给所有订阅者   | 事件总线 |
| **BehaviorSubject** | 发射最近一个值和后续值 | 状态管理 |
| **ReplaySubject**   | 重放所有值给新订阅者   | 缓存历史 |
| **AsyncSubject**    | 只发射最后一个值       | 单次结果 |
| **UnicastSubject**  | 只允许一个订阅者       | 单消费者 |

```
// PublishSubject示例（事件总线模式）
public class RxBus {
    private static final RxBus INSTANCE = new RxBus();
    private final PublishSubject<Object> mBus = PublishSubject.create();
    
    public static RxBus getInstance() { return INSTANCE; }
    
    public void post(Object event) {
        mBus.onNext(event);
    }
    
    public <T> Observable<T> toObservable(Class<T> eventType) {
        return mBus.ofType(eventType);
    }
}

// 使用
RxBus.getInstance().post(new UserLoginEvent(user));
RxBus.getInstance().toObservable(UserLoginEvent.class)
    .subscribe(event -> updateUI(event.getUser()));
```

## 八、错误处理最佳实践

### 1. **错误处理策略**

```
// 策略1：全局错误处理（不推荐用于生产）
RxJavaPlugins.setErrorHandler(throwable -> {
    // 全局错误处理
    Log.e("RxJava", "全局错误: " + throwable.getMessage());
});

// 策略2：链式错误处理
observable
    .map(data -> {
        try {
            return parseData(data);
        } catch (ParseException e) {
            throw new DataParseException(e);
        }
    })
    .onErrorResumeNext(throwable -> {
        if (throwable instanceof NetworkException) {
            return Observable.just(getCachedData());
        } else if (throwable instanceof DataParseException) {
            return Observable.error(new UserFriendlyError("数据解析失败"));
        } else {
            return Observable.error(throwable);
        }
    })
    .retryWhen(errors -> errors
        .zipWith(Observable.range(1, 3), (throwable, retryCount) -> {
            if (retryCount > 3) {
                throw new RuntimeException("重试超过3次", throwable);
            }
            return retryCount;
        })
        .flatMap(retryCount -> Observable.timer(retryCount, TimeUnit.SECONDS))
    )
    .subscribe(...);
```

### 2. **资源清理模式**

```
// 使用using操作符自动清理资源
Observable.using(
    () -> {
        // 创建资源（如数据库连接）
        DatabaseConnection conn = new DatabaseConnection();
        conn.open();
        return conn;
    },
    conn -> {
        // 使用资源创建Observable
        return Observable.create(emitter -> {
            try {
                List<Data> data = conn.queryData();
                for (Data d : data) {
                    if (emitter.isDisposed()) {
                        return;  // 检查是否已取消
                    }
                    emitter.onNext(d);
                }
                emitter.onComplete();
            } catch (Exception e) {
                emitter.onError(e);
            }
        });
    },
    conn -> {
        // 清理资源
        if (conn != null) {
            conn.close();
        }
    }
)
.subscribe(...);
```

## 九、性能优化

### 1. **内存泄漏预防**

```
// Android中避免内存泄漏
public class MainActivity extends AppCompatActivity {
    
    private CompositeDisposable mDisposables = new CompositeDisposable();
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        // 使用AutoDispose（RxJava扩展库）
        observable
            .delay(5, TimeUnit.SECONDS)
            .as(AutoDispose.autoDisposable(AndroidLifecycleScopeProvider.from(this)))
            .subscribe(...);
            
        // 或手动管理
        Disposable disposable = observable
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(data -> {
                // 检查Activity是否已销毁
                if (isDestroyed() || isFinishing()) {
                    return;
                }
                updateUI(data);
            });
            
        mDisposables.add(disposable);
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        mDisposables.clear();  // 取消所有订阅
    }
}
```

### 2. **操作符选择优化**

```
// 避免不必要的操作符链
observable
    .map(x -> x * 2)      // ✓ 必要转换
    .filter(x -> x > 10)  // ✓ 必要过滤
    .doOnNext(x -> log(x)) // ⚠️ 谨慎使用副作用
    .map(x -> x.toString()) // ✓ 必要转换
    .subscribe(...);

// 选择更高效的操作符
observable
    .flatMap(item -> {    // ❌ 可能创建过多Observable
        return fetchItemDetails(item);
    })
    .subscribe(...);

observable
    .concatMapEager(item -> {  // ✓ 更高效
        return fetchItemDetails(item);
    })
    .subscribe(...);
```

## 十、常见使用模式

### 1. **网络请求组合**

```
// 并行请求 + 结果合并
Observable.zip(
    api.getUserProfile(userId),
    api.getUserOrders(userId),
    api.getUserPreferences(userId),
    (profile, orders, preferences) -> {
        return new UserData(profile, orders, preferences);
    }
)
.subscribeOn(Schedulers.io())
.observeOn(AndroidSchedulers.mainThread())
.subscribe(
    userData -> showUserData(userData),
    error -> showError(error)
);

// 顺序依赖请求
api.login(username, password)
    .flatMap(token -> api.getUserInfo(token))
    .flatMap(userInfo -> api.getUserProjects(userInfo.getId()))
    .subscribe(...);
```

### 2. **防抖和节流**

```
// 搜索框输入防抖
RxTextView.textChanges(searchEditText)
    .debounce(300, TimeUnit.MILLISECONDS)  // 防抖
    .filter(text -> text.length() > 2)     // 过滤短文本
    .distinctUntilChanged()                // 去重
    .switchMap(query -> api.search(query)) // 取消旧请求
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(results -> updateSearchResults(results));
```

## 总结表格

### Observable 核心特性速查

| 特性     | 描述          | 关键点                            |
| -------- | ------------- | --------------------------------- |
| **创建** | 12+种创建方式 | create, just, from, interval      |
| **转换** | 数据变换      | map, flatMap, concatMap           |
| **过滤** | 数据筛选      | filter, take, skip, distinct      |
| **组合** | 多流合并      | merge, concat, zip, combineLatest |
| **错误** | 错误处理      | onErrorReturn, retry, retryWhen   |
| **线程** | 线程调度      | subscribeOn, observeOn            |
| **背压** | 流量控制      | onBackpressureBuffer/drop/latest  |
| **订阅** | 订阅管理      | Disposable, CompositeDisposable   |

### 选择指南

| 场景         | 推荐方案        | 理由       |
| ------------ | --------------- | ---------- |
| **少量数据** | Observable      | 简单高效   |
| **大量数据** | Flowable        | 支持背压   |
| **事件总线** | PublishSubject  | 实时广播   |
| **状态管理** | BehaviorSubject | 记忆状态   |
| **缓存历史** | ReplaySubject   | 重放数据   |
| **单次结果** | AsyncSubject    | 只发最后值 |

### 最佳实践

1. 

   **始终管理Disposable**：避免内存泄漏

2. 

   **合理选择调度器**：IO密集型用Schedulers.io()

3. 

   **避免嵌套订阅**：使用flatMap替代嵌套subscribe

4. 

   **处理背压**：大数据量时考虑Flowable

5. 

   **统一错误处理**：定义全局错误策略

6. 

   **资源清理**：使用using操作符管理资源

掌握Observable的核心在于理解**数据流思想**：创建流 → 转换流 → 组合流 → 订阅流。通过合理的操作符组合和线程调度，可以构建出高效、清晰、易维护的异步代码。

# Observable、Observer、Subscriber 三者的关系详解

## 一、类比理解（最简单的方式）

## 1. **现实世界比喻**

| 概念           | 现实类比        | 角色                 |
| -------------- | --------------- | -------------------- |
| **Observable** | 电视台/广播电台 | 数据生产者/发布者    |
| **Observer**   | 电视机/收音机   | 数据消费者/接收者    |
| **Subscriber** | 有线电视用户    | 建立连接的具体消费者 |

## 2.三者关系

以现实生活中杂志订阅为例，解释这三者的关系：

- 

  **Observable（可观察对象）**：好比一家杂志社，它负责生产杂志（数据），并且可以有多个订阅者。

- 

  **Observer（观察者）**：好比订阅杂志的人，他订阅杂志后，每当有新杂志出版，杂志社就会送给他，他就可以阅读杂志。

- 

  **Subscription（订阅）**：好比订阅行为本身，它连接了杂志社和订阅者。在RxJava中，订阅动作一旦发生，Observable就会开始向Observer发送数据。

在RxJava中，Observable是数据的生产者，Observer是数据的消费者。而Subscriber是Observer的扩展，增加了对订阅生命周期的控制（例如取消订阅）。

具体到代码层面：

1. 

   **Observable**：通过`create`、`just`、`from`等静态方法创建，它定义了如何生成数据流，以及当有Observer订阅时，如何发送数据。

2. 

   **Observer**：一个接口，有四个方法：

   - 

     `onSubscribe(Disposable d)`：当订阅建立时调用，参数Disposable可用于取消订阅。

   - 

     `onNext(T t)`：接收到数据时调用。

   - 

     `onError(Throwable e)`：发生错误时调用。

   - 

     `onComplete()`：数据发送完成时调用。

3. 

   **Subscriber**：在RxJava 1.x中，Subscriber是一个实现了Observer的抽象类，它增加了`unsubscribe()`和`isUnsubscribed()`方法来管理订阅状态。但在RxJava 2.x中，Subscriber被重新设计为`Flowable`的订阅者，而`Observable`的订阅者就是`Observer`。在RxJava 2.x中，我们通常使用`Observer`来订阅`Observable`，而`Subscriber`用于`Flowable`。

在RxJava 2.x中，当我们调用`Observable.subscribe(Observer)`时，实际上会创建一个`Subscription`（在2.x中称为`Disposable`）来表示订阅关系，并返回给调用者，以便在需要时取消订阅。

下面我们通过一个简单的例子来理解：

```
// 创建一个Observable（杂志社）
Observable<String> magazinePublisher = Observable.create(emitter -> {
    emitter.onNext("杂志第1期");
    emitter.onNext("杂志第2期");
    emitter.onNext("杂志第3期");
    emitter.onComplete();
});

// 创建一个Observer（订阅者）
Observer<String> reader = new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {
        // 当订阅时，可以在这里保存Disposable，以便后续取消订阅
        System.out.println("开始订阅");
    }

    @Override
    public void onNext(String s) {
        // 接收到数据
        System.out.println("收到: " + s);
    }

    @Override
    public void onError(Throwable e) {
        // 错误处理
        e.printStackTrace();
    }

    @Override
    public void onComplete() {
        // 完成
        System.out.println("订阅完成");
    }
};

// 建立订阅关系（订阅杂志）
magazinePublisher.subscribe(reader);
```

输出：

```
开始订阅
收到: 杂志第1期
收到: 杂志第2期
收到: 杂志第3期
订阅完成
```

在这个例子中，`magazinePublisher`是Observable，它生产了三期杂志。`reader`是Observer，他订阅了杂志。当订阅发生时，Observable开始发送数据，Observer的`onNext`被调用三次，然后`onComplete`被调用。

如果我们想要在接收到第二期杂志后取消订阅，可以在`onNext`中判断并调用`Disposable.dispose()`：

```
Observer<String> reader = new Observer<String>() {
    private Disposable mDisposable;

    @Override
    public void onSubscribe(Disposable d) {
        mDisposable = d;
        System.out.println("开始订阅");
    }

    @Override
    public void onNext(String s) {
        System.out.println("收到: " + s);
        if (s.equals("杂志第2期")) {
            mDisposable.dispose(); // 取消订阅
            System.out.println("取消订阅");
        }
    }

    @Override
    public void onError(Throwable e) {
        e.printStackTrace();
    }

    @Override
    public void onComplete() {
        System.out.println("订阅完成");
    }
};
```

这样，第三期杂志就不会被接收了。

总结：

- 

  **Observable**：数据生产者，决定什么时候发送数据，发送什么数据。

- 

  **Observer**：数据消费者，接收数据并处理。

- 

  **Subscription/Disposable**：表示订阅关系，用于取消订阅。

在RxJava 2.x中，我们通常使用`Observable`和`Observer`，而`Subscriber`主要用于`Flowable`（支持背压的Observable）。在RxJava 1.x中，`Subscriber`是`Observer`的扩展，但在2.x中两者有了更明确的区分。

## 二、记忆口诀

```markdown
Observable：被观察者，主动发
Observer  ：观察者接口，定义规范
Subscriber：订阅者实现，具体干

Observable 定义"怎么发"
Observer   定义"怎么收"  
Subscriber 是"谁在收"+"怎么收"
```

## 三、总结对比表

| 维度         | Observable               | Observer                          | Subscriber                                       |
| ------------ | ------------------------ | --------------------------------- | ------------------------------------------------ |
| **角色**     | 数据生产者               | 数据消费者协议                    | 数据消费者实现                                   |
| **主动性**   | 主动发送                 | 被动接收                          | 被动接收                                         |
| **状态**     | 通常无状态               | 接口无状态                        | 可以有状态                                       |
| **数量关系** | 一个对应多个             | 一个对应一个Observable            | 一个对应一个Observable                           |
| **主要方法** | create(), just(), from() | onNext(), onError(), onComplete() | onNext(), onError(), onComplete(), unsubscribe() |
| **类比**     | 广播电台                 | 收音机设计图                      | 具体的收音机+收听者                              |
| **创建时机** | 定义数据流               | 定义处理逻辑                      | 订阅时创建实例                                   |
| **生命周期** | 从创建到所有订阅者取消   | 从订阅到取消/完成                 | 从订阅到取消/完成                                |