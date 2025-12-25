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







