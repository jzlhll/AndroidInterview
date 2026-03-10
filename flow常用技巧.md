# Flow常用技巧


## 1️⃣ 常用操作符

**最常用**

- `map`/`filter` 用于数据转换
- `debounce`/`throttle` 用于UI防抖
- `catch`/`retry` 用于错误处理
- `combine`/`zip` 用于多流组合
- `stateIn`/`shareIn` 用于作用域共享

```kotlin
//基础操作
flow.map { it * 2 }            // 值转换
flow.filter { it > 0 }         // 条件过滤
flow.take(5)                   // 取前n个值
flow.drop(3)                   // 丢弃前n个值

//去重防抖
flow.debounce(300)             // 防抖（300ms内只取最新），类似输入文本搜索，用户停止输入后才执行搜索
flow.distinctUntilChanged()    // 跳过连续重复值
flow.throttleLatest(1000)      // 节流（每秒最多一个最新值）

//流合并
flow.merge(flow2)            // 合并多个流（按时间顺序）
flow.zip(flow2) { a, b -> }   // 一对一组合
flow.combine(flow2) { a, b -> } // 实时组合（任意变化触发）

//异常处理
flow.catch { e -> }           // 捕获异常
flow.onCompletion { }         // 流完成回调
flow.retry(3)                 // 失败重试
flow.retryWhen { cause, attempt -> } // 条件重试

//生命周期
flow.onStart { }              // 收集开始前
flow.onEach { }               // 每个值发射时
flow.onEmpty { }              // 空流时触发

//缓冲控制
flow.buffer()                 // 添加缓冲区
flow.conflate()               // 合并过快数据
flow.collectLatest { }        // 取消前一个收集

//单次操作
flow.first()                  // 第一个值
flow.single()                 // 确保只有一个值
flow.count()                  // 计数
flow.reduce { acc, value -> } // 累积计算
flow.fold(initial) { acc, v -> } // 带初始值的累积


//转换
flow.stateIn(scope)           // 转为StateFlow
flow.shareIn(scope)           // 转为SharedFlow
flow.flowOn(Dispatchers.IO)   //修改前序动作的执行线程模型

//开启收集
flow.launchIn(scope)          // 在作用域启动收集
```



## 2️⃣ 比较flow类型

### 1. `flow{}` 基础冷流

**适用场景：**

- 单次数据获取（网络请求、数据库查询）
- 转换或处理数据
- 需要 `suspend` 函数支持的操作

```kotlin
class UserRepository(
    private val apiService: ApiService,
    private val userDao: UserDao
) {
    // 示例1：网络请求
    fun fetchUserData(userId: String): Flow<User> = flow {
        try {
            val user = apiService.getUser(userId)
            emit(user)
        } catch (e: Exception) {
            // 可以在这里处理错误，或者向上传递
            throw e
        }
    }
    
    // 示例2：结合多个数据源
    fun getUserWithPreferences(userId: String): Flow<UserWithPreferences> = flow {
        val user = userDao.getUserStream(userId).first()
        val prefs = userDao.getPreferencesStream(userId).first()
        emit(UserWithPreferences(user, prefs))
    }
}
```

### 2. ROOM数据库

Room支持返回Flow，当数据库表发生变化时，Flow会自动发射新的数据。这实际上是一种冷流，但Room会管理其更新。

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getUsers(): Flow<List<User>>
}
```



### 3. **`StateFlow` - 状态（数据）持有**

**适用场景：**

- 需要始终有值的状态（如 UI 状态、缓存数据）
- 新订阅者需要立即获取当前值
- 数据层内部的状态管理

```kotlin
class SettingsRepository {
    // 私有可变，公开不可变
    private val _themeState = MutableStateFlow(Theme.DEFAULT)
    val themeState: StateFlow<Theme> = _themeState.asStateFlow()
    
    private val _appSettings = MutableStateFlow<AppSettings?>(null)
    val appSettings: StateFlow<AppSettings?> = _appSettings.asStateFlow()
    
    suspend fun loadSettings() {
        val settings = settingsDao.getSettings()
        _appSettings.value = settings
    }
    
    fun updateTheme(theme: Theme) {
        _themeState.value = theme
        // 可异步保存到数据库
        viewModelScope.launch {
            settingsDao.updateTheme(theme)
        }
    }
}
```

### 4. **`SharedFlow` - 事件广播**

**适用场景：**

- 一次性事件（Toast、Snackbar、导航事件）
- 需要多个订阅者的数据流
- 无需初始值，不保留历史记录的事件

```kotlin
class NotificationRepository {
    // 使用 replay = 0 表示不重放给新订阅者
    private val _notifications = MutableSharedFlow<NotificationEvent>(
        replay = 0,
        extraBufferCapacity = 64
    )
    val notifications: SharedFlow<NotificationEvent> = _notifications.asSharedFlow()
    
    suspend fun sendNotification(event: NotificationEvent) {
        _notifications.emit(event)
    }
    
    // 更复杂的示例：带背压处理
    private val _dataUpdates = MutableSharedFlow<DataUpdate>(
        replay = 1,  // 新订阅者获取最近一次更新
        onBufferOverflow = BufferOverflow.DROP_OLDEST
    )
    val dataUpdates: SharedFlow<DataUpdate> = _dataUpdates.asSharedFlow()
}
```

### 5. 最佳实践

#### repository示例

```kotlin
class UserRepository(
    private val userRemoteDataSource: UserRemoteDataSource,
    private val userLocalDataSource: UserLocalDataSource,
    private val dispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    // 1. 公开 StateFlow 用于 UI 状态观察
    private val _userState = MutableStateFlow<UserState>(UserState.Loading)
    val userState: StateFlow<UserState> = _userState.asStateFlow()
    
    // 2. 公开 SharedFlow 用于一次性事件
    private val _userEvents = MutableSharedFlow<UserEvent>()
    val userEvents: SharedFlow<UserEvent> = _userEvents.asSharedFlow()
    
    // 3. 获取单个用户（冷流）
    fun getUserStream(userId: String): Flow<User> = flow {
        // 先尝试从本地获取
        val localUser = userLocalDataSource.getUser(userId)
        emit(localUser)
        
        // 然后从远程更新
        try {
            val remoteUser = userRemoteDataSource.fetchUser(userId)
            userLocalDataSource.saveUser(remoteUser)
            emit(remoteUser)
        } catch (e: Exception) {
            // 发送错误事件
            _userEvents.emit(UserEvent.Error(e))
        }
    }.flowOn(dispatcher)
    
    // 4. 刷新数据并更新 StateFlow
    suspend fun refreshUser(userId: String) {
        _userState.value = UserState.Loading
        try {
            val user = userRemoteDataSource.fetchUser(userId)
            userLocalDataSource.saveUser(user)
            _userState.value = UserState.Success(user)
            _userEvents.emit(UserEvent.RefreshSuccess)
        } catch (e: Exception) {
            _userState.value = UserState.Error(e)
            _userEvents.emit(UserEvent.RefreshError(e))
        }
    }
    
    // 5. 观察本地数据变化（数据库支持 Flow 时）
    fun observeUsers(): Flow<List<User>> {
        return userLocalDataSource.observeUsers()
            .catch { e ->
                _userEvents.emit(UserEvent.DatabaseError(e))
            }
            .flowOn(dispatcher)
    }
}
```

#### DataSource示例

```kotlin
// 网络数据源
class UserRemoteDataSource {
    // 使用 flow 包装网络请求
    fun fetchUser(userId: String): Flow<User> = flow {
        val response = apiService.getUser(userId)
        if (response.isSuccessful) {
            emit(response.body()!!)
        } else {
            throw ApiException(response.code(), response.message())
        }
    }
}

// 本地数据源（Room 数据库）
class UserLocalDataSource(private val userDao: UserDao) {
    // Room 直接返回 Flow
    fun observeUsers(): Flow<List<User>> = userDao.observeAllUsers()
    
    // 单次查询也使用 Flow
    fun getUser(userId: String): Flow<User?> = flow {
        val user = userDao.getUserById(userId)
        emit(user)
    }
}
```



------



## 3️⃣ 比较launchIn和collect

如下两种写法，是否相同？

```kotlin
//第一种写法
xxxFlow
.onEach {
    //do something...
}
.launchIn(scope)

//第二种写法
scope.launch {
    xxxFlow.collect {
        //do something...
    }
}
```

**`launchIn` 就是 `scope.launch { collect() }` 的语法糖**。

### 1. 操作便利性

#### 链式调用

```kotlin
// launchIn 可以方便地链式调用
flow
    .filter { ... }
    .map { ... }
    .onEach { ... }
    .launchIn(scope)  // 一行代码

// collect 需要更多代码
scope.launch {
    flow
        .filter { ... }
        .map { ... }
        .collect { value ->
            // 所有操作都要写在这里
        }
}
```

#### 异常处理

```kotlin
// ✅ 推荐：使用 launchIn + catch
discovery.discoveredDevices
    .onEach { device ->
        onCurrentFrameChangedAsync(device)
    }
    .catch { e ->
        // 处理异常，不会传播到父作用域
        Log.e("Flow", "Error in device discovery", e)
    }
    .launchIn(scope)

// ❌ 问题：异常会取消整个协程
scope.launch {
    try {
        discovery.discoveredDevices.collect {
            onCurrentFrameChangedAsync(it)
        }
    } catch (e: Exception) {
        // 能捕获异常，但协程已结束
    }
}
```

#### Job

```kotlin
// launchIn 返回 Job，可以单独管理
val job = flow.onEach { ... }.launchIn(scope)
job.cancel()  // 单独取消这个流收集

// collect 是整个协程的一部分
val job = scope.launch {
    flow.collect { ... }
    // 协程中其他代码也会一起取消
}
```

#### 并行多个流

```kotlin
// ✅ launchIn 更清晰
val job1 = flow1.onEach { ... }.launchIn(scope)
val job2 = flow2.onEach { ... }.launchIn(scope)
val job3 = flow3.onEach { ... }.launchIn(scope)

// ✅ collect 也可以，但代码冗长
scope.launch { flow1.collect { ... } }
scope.launch { flow2.collect { ... } }
scope.launch { flow3.collect { ... } }
```

### 2. 建议

```kotlin
////////使用函数式
// ✅ 有多个操作符时更清晰
flow
    .filter { ... }
    .map { ... }
    .onStart { showLoading() }
    .onEach { updateUI(it) }
    .onCompletion { hideLoading() }
    .catch { showError(it) }
    .launchIn(viewModelScope)

//////////使用launch命令式
// ✅ 需要复杂条件逻辑时
scope.launch {
    flow.collect { value ->
        when {
            value.type == Type.A -> handleA(value)
            value.count > 10 -> handleMany(value)
            else -> handleDefault(value)
        }
    }
}

// ✅ 需要与其他协程操作组合
scope.launch {
    // 1. 先获取配置
    val config = loadConfig()
    
    // 2. 根据配置处理流
    flow.collect { value ->
        if (config.enabled) {
            processValue(value)
        }
    }
    
    // 3. 流结束后清理
    cleanup()
}
```



## 4️⃣ 热流冷流

### 一、核心基础：冷流 vs 热流（最关键的底层逻辑）

#### 1. 本质定义（判断标准）

| 维度              | 冷流（Cold Flow）                                  | 热流（Hot Flow）                                     |
| ----------------- | -------------------------------------------------- | ---------------------------------------------------- |
| 核心特征          | **订阅驱动**：有订阅者才启动上游，无订阅者上游停止 | **独立运行**：上游不依赖订阅者，有无订阅者都可能运行 |
| 订阅者 - 上游关系 | 一对一：每个订阅者触发**独立的上游实例**           | 一对多：所有订阅者共享**同一个上游实例**             |
| 数据接收规则      | 订阅后才能收到后续发射的数据（错过的收不到）       | 无论何时订阅，都能收到订阅后的新数据（可配置缓存）   |
| 典型代表          | 普通 Flow（flow {...}）、flowOf、asFlow 等         | StateFlow、SharedFlow、Channel.asFlow ()（特殊）     |

#### 2. 代码验证（一眼看懂区别）

```kotlin
// 冷流示例：每个订阅者触发一次上游（打印一次"冷流上游启动"）
val coldFlow = flow {
    println("冷流上游启动") // 订阅时才执行
    emit(1)
    emit(2)
}

// 订阅1次 → 打印"冷流上游启动"
coldFlow.collect { println("订阅者1：$it") }
// 订阅2次 → 再次打印"冷流上游启动"（上游重复执行）
coldFlow.collect { println("订阅者2：$it") }

// 热流示例：所有订阅者共享上游（只打印一次"热流上游启动"）
val hotFlow = coldFlow.shareIn(
    scope = CoroutineScope(Dispatchers.Default),
    started = SharingStarted.Eagerly,
    replay = 1
)

// 订阅1次 → 打印"热流上游启动"
hotFlow.collect { println("热流订阅者1：$it") }
// 订阅2次 → 不打印"热流上游启动"（共享已有上游）
hotFlow.collect { println("热流订阅者2：$it") }
```

#### 3. 关键结论

- 冷流的核心问题：**多订阅者会重复执行上游逻辑**（比如多次发起网络请求、多次监听 WebSocket 状态），这是 “性能浪费” 的根源；
- 热流的核心价值：**共享上游**，无论多少订阅者，上游只执行一次。

### 二、热流的具体实现：SharedFlow vs StateFlow

StateFlow 是**特殊的 SharedFlow**，两者都是热流，核心区别如下：

| 特性           | SharedFlow                               | StateFlow                                                    |
| -------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 初始值         | 可选（无默认值）                         | 必须有初始值（创建时指定）                                   |
| 缓存（replay） | 可自定义（比如 replay=3 缓存 3 个值）    | 固定 replay=1（只缓存最新值）                                |
| 发射规则       | 可以发射相同值（多次 emit (1) 都会触发） | 只发射**与当前值不同**的新值（emit (1) 后再 emit (1) 不触发） |
| 典型场景       | 事件通知（比如点击事件、网络请求结果）   | 状态存储（比如 UI 状态、WebSocket 连接状态、网络请求 ID）    |

**代码示例**

```kotlin
// 1. StateFlow（存储网络请求ID状态，有初始值）
val _currentRequestId = MutableStateFlow<String?>(null) // 必须有初始值
val currentRequestId: StateFlow<String?> = _currentRequestId

// 只有值变化时才触发订阅者（重复emit相同值不触发）
_currentRequestId.value = "req_123" // 触发订阅者
_currentRequestId.value = "req_123" // 不触发订阅者

// 2. SharedFlow（发送WebSocket连接事件，可重复发射）
val _webSocketEvents = MutableSharedFlow<WebSocketEvent>(
    replay = 0, // 不缓存事件，错过就没了
    extraBufferCapacity = 0
)
val webSocketEvents: SharedFlow<WebSocketEvent> = _webSocketEvents

// 重复发射相同事件也会触发订阅者
_webSocketEvents.emit(WebSocketEvent.Connected)
_webSocketEvents.emit(WebSocketEvent.Connected)
```

### 三、关键操作符：是否改变 Flow 的冷热属性？

**核心结论**：`combine`/`map`/`filter`/`merge`/`zip`/`debounce` 等**所有基础操作符，都不会改变原 Flow 的冷热属性**，也不会自动共享上游。

#### 1. 重点验证：combine 对冷热属性的影响

```kotlin
// 上游1：StateFlow（热流）
val apiInfoFlow: StateFlow<ApiResponse?> = apiService.infoFlow
// 上游2：普通Flow（冷流）
val socketFlow: Flow<WebSocketState> = webSocketManager.socketFlow

// combine后的流：冷流！（只要有一个上游是冷流，combine后就是冷流）
val combinedFlow = combine(apiInfoFlow, socketFlow) { requestData, state ->
    // 这个lambda，每个订阅者都会触发独立执行
    println("combine逻辑执行") 
    Pair(requestData, state)
}

// 订阅1次 → 打印"combine逻辑执行"
combinedFlow.collect { ... }
// 订阅2次 → 再次打印"combine逻辑执行"（上游重复执行）
combinedFlow.collect { ... }
```

#### 2. 各操作符的统一规则

| 操作符                        | 是否改变冷热属性 | 是否重复执行上游 | 核心影响                    |
| ----------------------------- | ---------------- | ---------------- | --------------------------- |
| map/filter                    | 否               | 是（冷流场景）   | 只是转换 / 过滤数据，不共享 |
| combine/merge/zip             | 否               | 是（冷流场景）   | 包装多个流，冷流特性保留    |
| debounce/distinctUntilChanged | 否               | 是（冷流场景）   | 只是限流 / 去重，不共享     |
| shareIn/stateIn               | 是（冷→热）      | 否（热流场景）   | 转为热流，共享上游          |

> 使用了 combine 操作符，它要求 所有 输入的 Flow 必须至少发射一次数据，才会合并并发射第一个结果。

### 四、性能优化：避免上游 “多开” 

核心诉求是 “不浪费性能”，本质就是**避免冷流的多订阅者重复执行上游**，唯一解决方案：`shareIn`/`stateIn` 将冷流转热流。

#### 1. 最优写法（通用场景）

```kotlin
// 步骤1：定义冷流（combine后的流）
val coldCombinedFlow = combine(
    apiService.infoFlow, // 可能是StateFlow（热）
    webSocketManager.socketFlow // 可能是冷流
) { requestInfo, _ ->
    // 业务逻辑处理
    Pair(requestInfo, _)
}.filterNotNull()

// 步骤2：转为热流（核心：避免多订阅者重复执行combine逻辑）
val hotCombinedFlow = coldCombinedFlow.shareIn(
    scope = appCoroutineScope, // 建议用生命周期绑定的Scope（如viewModelScope）
    started = SharingStarted.WhileSubscribed(5000), // 5秒超时释放
    replay = 1 // 缓存最新值，新订阅者立即拿到
)

// 多次订阅 → 上游combine逻辑只执行一次
hotCombinedFlow.collect { ... } // 第一次订阅启动上游
hotCombinedFlow.collect { ... } // 第二次订阅共享上游，不重复执行
```

#### 2. `WhileSubscribed(5000)` 的深层价值

- 避免 “页面旋转 / 短时间切后台” 导致上游重启（5 秒内重新订阅，上游不停止）；
- 避免 “长时间无订阅” 导致资源浪费（5 秒后停止上游，释放网络请求 / 网络监听）。

#### 3. 实战避坑点

1. **误区**：“上游是 StateFlow（热流），combine 后就是热流” → 错！combine 后的流依然是冷流，多订阅者会重复执行 combine 内的逻辑；
2. **误区**：“加了 filterNotNull 就是热流” → 错！filter 只是过滤，不改变冷热属性；
3. **关键**：只要流需要被**多个地方订阅**（比如 Fragment+ViewModel），就必须用 shareIn/stateIn 转为热流。

### 总结

**冷热流核心本质**：冷流是 “数据产生逻辑的封装”，本身不主动产生数据，仅在调用`collect`订阅时执行内部逻辑、创建独立的 “数据生产者”，多订阅者会触发多个独立生产者（如多次发起独立网络请求），适合需要 “多次独立执行” 的场景；热流的核心是 “全局唯一的数据流”，上游不依赖订阅者独立运行，所有订阅者共享同一个生产者，能从根源避免重复执行上游逻辑造成的性能浪费。

**操作符核心规则**：map、combine、filter 等基础操作符不会改变 Flow 的冷热属性，冷流经其处理后仍为冷流；只有`shareIn`/`stateIn`能将冷流转为热流，是实现上游共享的唯一方式。

**性能优化核心方案**：多订阅场景（如 Fragment+ViewModel 共用数据流）下，需通过`shareIn`/`stateIn`将冷流（尤其是 combine 后的冷流）转为热流；配合`WhileSubscribed(5000)`使用可平衡响应速度与资源释放，既避免短时间页面切换导致的上游重启，又能在长时间无订阅时释放网络、WebSocket 监听等资源。

**热流选型原则**：基于数据类型选择适配的热流实现 —— 状态型数据（如网络请求 ID、WebSocket 连接状态）优先用`StateFlow`（有初始值、固定 replay=1、自动去重重复值，适配 “存储单一最新状态” 需求）；事件型数据（如网络请求结果、WebSocket 连接事件）优先用`SharedFlow`（无强制初始值、可自定义缓存、支持重复发射相同事件）；已有冷流需转为全局唯一数据流时，用`shareIn`配合生命周期 Scope、`WhileSubscribed(5000)`和合理的 replay 值改造。



## 5️⃣ Flow/Channel/协程-原理

### 协程

**编译器重写 + 状态机（核心是「标签式状态分发」）**

协程的核心是 **Kotlin 编译器对挂起函数的代码重写** + 「状态机模式」，而非 OS 级线程：

1. **本质**：协程是用户态的轻量级执行单元，底层依托线程池（Dispatcher）调度，挂起 / 恢复完全由 Kotlin 控制，不涉及 OS 内核态操作；

2. 状态机的具体实现

   （反编译可见）：

   - 编译器会将包含`suspend`的函数，拆分为**多个执行块**，并为每个 “挂起点”（如`delay`、`withContext`）分配唯一的「状态标签」（整数 / 枚举）；
   - 同时生成一个**状态持有者类**（或用局部变量存状态），记录当前执行到哪个状态、以及所有局部变量的值；
   - 挂起时：保存当前状态标签 + 变量值，释放线程（线程去处理其他任务），协程从线程中 “脱离”；
   - 恢复时：根据保存的状态标签，直接跳转到对应的执行块继续，而非从头执行。

| 类型       | `join()` 方法                                                                 | `await()` 方法                                                                 | 异常对 `forEach` 遍历的影响       | 核心用途                     |
|------------|-------------------------------------------------------------------------------|--------------------------------------------------------------------------------|-----------------------------------|------------------------------|
| `Job`      | ✅ 有（挂起函数）<br>→ 等待任务完成<br>→ 无返回值<br>→ 任务异常时**不主动抛出** | ❌ 无（调用会直接编译报错）                                                     | -（无 `await()` 方法，仅用 `join()` 遍历时，单个任务异常不会中断遍历） | 仅需等待任务完成，不需要结果 |
| `Deferred` | ✅ 有（继承自 `Job`）<br>→ 行为和 `Job` 的 `join()` 完全一致                   | ✅ 有（`Deferred` 独有）<br>→ 等待任务完成<br>→ **返回任务执行结果**<br>→ 任务异常时**主动抛出** | 用 `await()` 遍历时，单个任务异常会立即抛错，中断后续遍历；用 `join()` 遍历时，无此问题 | 等待完成 + 获取任务结果      |


3. 极简反编译视角举例

   ```kotlin
   suspend fun test() {
       println("第一步") // 状态0
       delay(1000)      // 挂起点 → 状态1
       println("第二步") // 状态2
   }
   ```

   编译器重写后（伪代码）：

   ```kotlin
   // 状态标签：0=初始，1=等待delay恢复，2=执行第二步
   fun test(state: Int, vars: TestVars) {
       when (state) {
           0 -> {
               println("第一步")
               // 调用delay，保存状态为1，挂起当前执行
               suspendCoroutine { cont ->
                   delayDispatch(1000) { cont.resumeWith(1, vars) }
               }
           }
           1 -> {
               // delay结束，恢复执行，跳转到状态2
               println("第二步")
           }
       }
   }
   ```

### Flow/Channel

Flow 的底层**并非基于 Channel**，两者设计核心完全不同：

1. Flow 核心逻辑

   - 冷流本质是「拉取式的挂起序列」：下游`collect()`（挂起函数）主动向上游请求数据，上游`emit()`（挂起函数）按需发射，全程通过协程的挂起 / 恢复协作，无队列、无独立线程；
   - 只有`channelFlow`（扩展函数）会借助 Channel 实现（解决多线程并发`emit`的问题），但这是 “适配层”，不是 Flow 的原生底层；

   

2. Channel vs Flow 的核心差异

   - Channel：「推送模型」，核心是线程安全的队列，生产者主动推送数据到队列，消费者被动取，是 “热” 的；
   - Flow：「拉取模型」，无队列，下游不拉取，上游不发射，是 “冷” 的（除非转热流）。

