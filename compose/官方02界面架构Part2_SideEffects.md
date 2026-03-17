# Jetpack Compose 副作用（Side-effects）核心解析

## 核心关键词说明

|         关键词         |                           核心描述                           |
| :--------------------: | :----------------------------------------------------------: |
| 副作用（Side-effects） | Compose 可组合函数内执行的、超出 UI 渲染逻辑的操作，需通过专属 API 管理生命周期 |
|      组合生命周期      | 可组合函数实例在 Composition 树中的 “进入 - 重组 - 离开” 全过程，副作用需与其绑定 |
|          协程          | Compose 处理异步副作用的核心方式，所有协程需绑定组合生命周期 |
|      Effect Keys       |   控制副作用重启的核心参数，值变化会触发副作用先清理再重启   |
|     LaunchedEffect     |     Compose 内启动协程的核心 API，协程生命周期与组合绑定     |
|    DisposableEffect    |     处理需显式清理的副作用，必须实现 onDispose 清理逻辑      |
|       SideEffect       | 每次重组成功后执行，同步 Compose 状态到非 Compose 管理的对象 |
|      produceState      |     将非 Compose 状态转换为 Compose 可感知的 State 对象      |
|     derivedStateOf     |    派生新 State，仅当派生结果变化时触发重组，减少无效重组    |
|      snapshotFlow      | 将 Compose 的 State 对象转换为 Kotlin Flow，仅发射有效变化的值 |
|        资源清理        |   副作用中需手动释放的操作（如监听器解注册），避免内存泄漏   |
|        状态转换        | 实现 Compose State 与非 Compose 状态（Flow/LiveData）的双向转换 |
| rememberCoroutineScope |   获取与组合绑定的协程作用域，支持在用户事件中手动启动协程   |
|  rememberUpdatedState  |           包装变量以获取最新值，且不触发副作用重启           |

## 核心介绍

虽然，理想情况下，Compose的可组合函数都应该是没有副作用的。但是，有时候，执行超出 UI 渲染逻辑的操作（如网络请求、注册监听器、页面导航等）。由于可组合函数的重组具有不可预测性，直接执行副作用易导致逻辑异常或内存泄漏，因此需通过 Compose 框架提供的专属 API 管理，确保副作用的执行时机、生命周期与组合绑定，同时通过 Effect Keys 控制重启逻辑，兼顾功能正确性与性能。



## 一、副作用的核心设计原则

1. 可组合函数应优先保持纯函数特性，仅根据输入参数渲染 UI，所有副作用需通过 Effect 系列 API 执行；
2. 所有 Effect API 均感知组合生命周期，进入组合时启动副作用，离开组合时自动执行清理或取消逻辑；
3. Effect Keys 是控制副作用重启的核心，Key 值变化时，框架会先清理原有副作用，再启动新的副作用逻辑。


Effect（副作用处理器）是一种**不会绘制 UI**的可组合函数，它的作用是**让你能够安全地执行那些会导致副作用的操作**，并且这些操作**会在组合过程完成之后才被实际运行**。





## 二、核心副作用 API 详解 - Effect

### 1. LaunchedEffect

- 作用：在可组合函数内部启动协程，执行挂起函数；默认是在主线程，如果需要子线程，使用`withContext(Dispatchers.IO)`。
- 核心特性：协程生命周期与组合绑定，进入组合时启动协程，离开组合时自动取消；Effect Keys 变化时，取消原有协程并重启新协程。
- 代码示例：

```kotlin
@Composable
fun LaunchedEffectUsage() {
    val count = remember { mutableStateOf(0) }
    
    LaunchedEffect(key1 = count.value) {
        delay(1000)
        println("LaunchedEffect 执行：count = ${count.value}")
    }

    Column(Modifier.padding(16.dp)) {
        Text("当前计数：${count.value}")
        Button(onClick = { count.value++ }) {
            Text("计数+1（触发 LaunchedEffect 重启）")
        }
    }
}
```

- 执行逻辑：点击按钮修改 count 值，Effect Keys 变化触发 LaunchedEffect 重启，1 秒后打印最新 count 值；离开该组合时，未执行完的协程会被自动取消。



### 2. DisposableEffect

- 作用：处理需要显式清理的副作用操作；
- 核心特性：必须实现 onDispose 代码块，Effect Keys 变化或离开组合时，先执行 onDispose 中的清理逻辑，再重启或终止副作用；
- 代码示例：

```kotlin
@Composable
fun DisposableEffectUsage() {
    val lifecycleOwner = LocalLifecycleOwner.current
    
    DisposableEffect(key1 = lifecycleOwner) {
        val observer = LifecycleEventObserver { _, event ->
            println("生命周期变化：$event")
        }
        lifecycleOwner.lifecycle.addObserver(observer)
        
        onDispose {
            lifecycleOwner.lifecycle.removeObserver(observer)
            println("移除生命周期监听器")
        }
    }

    Text("监听生命周期变化（离开组合自动清理）", Modifier.padding(16.dp))
}
```

- 执行逻辑：进入组合时注册生命周期监听器，打印生命周期事件；当 lifecycleOwner 变化或离开组合时，执行 onDispose 中的逻辑移除监听器，避免内存泄漏。



### 3. SideEffect

- 作用：将 Compose 状态同步到非 Compose 管理的对象（如原生 SDK、第三方库）；
- 核心特性：每次**可组合函数重组成功后**都会执行，确保非 Compose 代码能获取最新的 Compose 状态；
- 代码示例：

```kotlin
object ThirdPartyAnalytics {
    var userName: String = "未设置"
    fun updateUserInfo(name: String) {
        userName = name
        println("第三方统计更新用户名：$name")
    }
}

@Composable
fun SideEffectUsage() {
    var userName by remember { mutableStateOf("张三") }

    SideEffect { //每次重组后都会执行
        ThirdPartyAnalytics.updateUserInfo(userName)
    }

    Column(Modifier.padding(16.dp)) {
        Text("当前用户名：$userName")
        Button(onClick = { userName = "李四" }) {
            Text("修改用户名")
        }
    }
}
```

- 执行逻辑：点击按钮修改 userName 触发重组，重组成功后 SideEffect 执行，将最新的 userName 同步到第三方统计工具中。

  



## 三、核心副作用 API 详解 - 对象

### 1. rememberCoroutineScope

- 作用：获取与组合生命周期绑定的协程作用域；
- 核心特性：作用域在组合离开时自动取消，支持在可组合函数外（如用户点击事件）启动协程，手动控制协程生命周期；
- 代码示例：

```kotlin
@Composable
fun RememberCoroutineScopeUsage() {
    val scope = rememberCoroutineScope()
    var snackbarVisible by remember { mutableStateOf(false) }

    Column(Modifier.padding(16.dp)) {
        Button(onClick = {
            scope.launch {
                snackbarVisible = true
                delay(2000)
                snackbarVisible = false
            }
        }) {
            Text("显示提示（2秒后消失）")
        }
        
        if (snackbarVisible) {
            Text("操作成功！", Modifier.padding(top = 8.dp))
        }
    }
}
```

- 执行逻辑：点击按钮后启动协程，修改 snackbarVisible 状态显示提示，2 秒后隐藏；若此时离开组合，未执行完的协程会被作用域自动取消。



### 2. rememberUpdatedState

- 作用：创建可更新的状态引用，让副作用内始终获取最新的参数值；
- 核心特性：包装后的变量值更新不会改变 Effect Keys，避免长耗时副作用因参数变化被频繁重启；
- 代码示例：

```kotlin
@Composable
fun RememberUpdatedStateUsage(onTimeout: () -> Unit) {
    val latestOnTimeout by rememberUpdatedState(newValue = onTimeout)

    LaunchedEffect(key1 = true) {
        delay(3000)
        latestOnTimeout()
    }

    Text("3秒后执行回调（回调更新不重启协程）", Modifier.padding(16.dp))
}

@Composable
fun RememberUpdatedStateCaller() {
    var message by remember { mutableStateOf("初始回调") }
    
    RememberUpdatedStateUsage(
        onTimeout = { message = "最新回调执行！" }
    )
    
    Text(message, Modifier.padding(16.dp))
}
```

- 执行逻辑：LaunchedEffect 的 Key 为常量 true，仅进入组合时执行一次；即使 onTimeout 回调更新，通过 rememberUpdatedState 包装后，3 秒后仍能执行最新的回调逻辑，且不会重启协程。

>  当你需要在 长期运行的协程（如 LaunchedEffect 中的延迟、网络请求）中调用 可能变化的回调 / 参数 时，直接使用原参数会导致 “调用旧值”，而将参数设为 LaunchedEffect 的键又会导致协程频繁重启。
>
>  rememberUpdatedState 的价值就在于：它能让你在不中断协程执行的前提下，始终持有最新的参数引用，完美解决 “旧回调” 问题。
>
>  记住这个场景：长期协程 + 可变回调 = 用 rememberUpdatedState 保鲜引用。
>
>  https://blog.csdn.net/wangpengfei_p/article/details/157300422



### 3. produceState

- 作用：将非 Compose 状态（如 Flow、LiveData、网络请求结果）转换为 Compose 可感知的 State 对象；
- 核心特性：基于协程执行生产者逻辑，进入组合时启动，离开组合时取消；返回的 State 值重复时不会触发重组；
- 代码示例：

```kotlin
fun fetchUserInfo(): Flow<String> {
    return flow {
        delay(1000)
        emit("用户ID：1001，姓名：张三")
    }
}

@Composable
fun ProduceStateUsage() {
    val userInfo by produceState(initialValue = "加载中...") {
        fetchUserInfo().collect { value = it }
    }

    Text(userInfo, Modifier.padding(16.dp))
}
```

- 执行逻辑：初始时 userInfo 为 “加载中”，produceState 启动协程收集 Flow 数据，1 秒后获取到用户信息并更新 State，UI 自动刷新显示最新内容；离开组合时，协程自动取消。

> 仅在**单个 Compose 函数内**生效的临时状态（无需跨配置变更保活）；
>
> 简单的 “非业务级” 异步逻辑（如读取本地 SP、监听 UI 状态转换）；
>
> 快速原型开发（无需搭建 ViewModel，简化代码）。



更规范的写法ViewModel+StateFlow的方案(todo 形成skills开发框架)：

```kotlin
// 1. 定义ViewModel，承载业务逻辑和状态
class UserInfoViewModel : ViewModel() {
    // 私有可变StateFlow（仅ViewModel内部修改）
    private val _userInfo = MutableStateFlow("加载中...")
    // 公开不可变StateFlow（UI层仅能观察，不能修改）
    val userInfo: StateFlow<String> = _userInfo.asStateFlow()

    // 2. 封装异步获取数据的业务逻辑
    fun fetchUserInfo() {
        // 依托viewModelScope执行协程，自动管控生命周期
        viewModelScope.launch {
            // 模拟网络请求/异步操作
            delay(1000)
            // 更新StateFlow的值（UI层自动感知）
            _userInfo.value = "用户ID：1001，姓名：张三"
        }
    }

    // 可选：页面初始化时自动触发数据请求（也可由UI层手动触发）
    init {
        fetchUserInfo()
    }
}

@Composable
fun UserInfoScreen(
    // 注入ViewModel（Compose内置viewModel()函数，自动关联生命周期）
    viewModel: UserInfoViewModel = viewModel()
) {
    // 核心：收集ViewModel的StateFlow数据，转换为Compose可感知的State
    // collectAsStateWithLifecycle：符合生命周期管控的最佳实践
    val userInfo by viewModel.userInfo.collectAsStateWithLifecycle()

		Column(Modifier.padding(16.dp)) {
        Text(text = userInfo)
        Button(onClick = { viewModel.fetchUserInfo() }) {
            Text("重新获取用户信息")
        }
    }
}
```

- `collectAsState`：无论页面是否在前台，都会持续收集数据（比如页面退到后台，仍会接收 StateFlow 数据，浪费资源）；

  `collectAsStateWithLifecycle`：绑定页面生命周期（默认在 `STARTED` 状态收集，`STOPPED` 状态暂停），是官方推荐的 Compose 收集 Flow/StateFlow 的方式。



### 4. snapshotFlow

- 作用：将 Compose 的 State 对象转换为 Kotlin Flow；
- 核心特性：仅当 State 值发生有效变化（非重复值）时发射数据，等效于 Flow 的 distinctUntilChanged () 操作；
- 代码示例：

```kotlin
@Composable
fun SnapshotFlowUsage() {
    val scrollPosition by remember { mutableStateOf(0) }
    val scope = rememberCoroutineScope()

    val scrollFlow = snapshotFlow { scrollPosition }
        .filter { it > 50 }
        .debounce(300)

    LaunchedEffect(key1 = scrollFlow) {
        scrollFlow.collect {
            println("滚动位置超过50：$it")
        }
    }

    Column(Modifier.padding(16.dp)) {
        Text("当前滚动位置：$scrollPosition")
        Button(onClick = { scrollPosition += 30 }) {
            Text("滚动+30")
        }
    }
}
```

- 执行逻辑：点击按钮增加滚动位置，当位置超过 50 且防抖 300 毫秒后，snapshotFlow 发射数据，LaunchedEffect 收集并打印日志；若滚动位置重复变化（如多次点击但未超过 50），则不会发射数据。



### 5. derivedStateOf

- 作用：将一个或多个 Compose State 派生为新的 State 对象；
- 核心特性：仅当派生结果发生变化时触发重组，适用于高频变化的 State 需低频率更新 UI 的场景；
- 代码示例：

```kotlin
@Composable
fun DerivedStateOfUsage() {
    val scrollY by remember { mutableStateOf(0) }
    LaunchedEffect(key1 = true) {
        repeat(10) {
            delay(1000)
            scrollY += 20
        }
    }

    val showTopButton by remember {
        derivedStateOf { scrollY > 100 }
    }

    Column(Modifier.padding(16.dp)) {
        Text("滚动位置：$scrollY")
        if (showTopButton) {
            Button(onClick = { /* 回到顶部逻辑 */ }) {
                Text("回到顶部")
            }
        }
    }
}
```

- 执行逻辑：scrollY 每秒增加 20（高频变化），derivedStateOf 仅在 scrollY>100 时将 showTopButton 设为 true，此时 UI 才会重组并显示 “回到顶部” 按钮，避免因 scrollY 高频变化导致的无效重组。





## 四、Effect Keys 核心规则

Effect Keys 是控制副作用重启的核心参数，需遵循以下规则：

1. 副作用代码块中使用的所有可变 / 不可变变量，均需作为 Effect Keys 传入，确保变量变化时副作用能及时重启；

   代码示例（正确 / 错误对比）：

   ```kotlin
   // 正确：count作为Key，变化时副作用重启
   LaunchedEffect(key1 = count.value) {
       println("count 变化：${count.value}")
   }
   
   // 错误：count未作为Key，变化时副作用不重启，打印旧值
   LaunchedEffect(key1 = true) {
       println("count 变化：${count.value}")
   }
   ```

   

2. 使用常量（如 true、Unit）作为 Effect Keys 时，副作用仅在进入组合时执行一次，后续变量变化不会触发重启；

   代码示例：

   ```kotlin
   LaunchedEffect(key1 = Unit) {
       delay(1000)
       println("仅执行一次：${count.value}")
   }
   ```

3. 若变量变化无需重启副作用，需通过 rememberUpdatedState 包装变量，不将其作为 Effect Keys 传入。

   

## 四、使用误区

1. 滥用 derivedStateOf：简单的状态拼接（如姓名 + 年龄）无需使用，该 API 仅适用于高频 State 低频率更新 UI 的场景，滥用会增加性能开销；
2. DisposableEffect 空 onDispose 实现：DisposableEffect 的核心是显式清理资源，空 onDispose 说明选错了 API，此类场景应优先使用 LaunchedEffect；
3. 使用全局协程作用域：Compose 内的协程必须与组合生命周期绑定，避免使用 GlobalScope 启动协程，否则易导致内存泄漏；
4. 忽略 Effect Keys 配置：未将副作用使用的变量作为 Key，会导致副作用无法感知变量变化，执行逻辑与预期不符。



```kotlin
// ❌ 错误：Key为true（常量），count变化时Effect不重启，始终打印初始值
LaunchedEffect(true) {
    delay(1000)
    println("【错误】count = ${count.value}") // 永远打印0
}

LaunchedEffect(true) { // ❌ 协程无限循环，仅组合离开时才会取消
    while (true) {
        delay(1000)
        println("无限打印...") // 页面退到后台仍会打印，浪费资源
    }
}


// ✅ 有效：Key=true仅执行一次，通过rememberUpdatedState获取最新回调
val latestOnTimeout by rememberUpdatedState(onTimeout)
LaunchedEffect(true) {
    delay(3000)
    latestOnTimeout() // 能执行最新回调，且不重启协程
}

// ✅ 有效：仅首次进入组合时上报埋点，无需重启
LaunchedEffect(true) {
    println("【有效】页面首次进入，上报埋点")
}


// ✅ 有效：Key=true但通过repeatOnLifecycle管控，后台自动暂停
LaunchedEffect(true) {
    lifecycleOwner.lifecycle.repeatOnLifecycle(Lifecycle.State.RESUMED) {
        testFlow.collect { println("【有效】前台监听：$it") }
    }
}
```





## 总结

1. Compose 副作用需通过专属 API 管理，核心目标是绑定组合生命周期、精准控制重启时机、显式清理资源，避免逻辑异常和内存泄漏；
2. 协程类副作用优先使用 LaunchedEffect（组合内）或 rememberCoroutineScope（用户事件），资源清理类副作用使用 DisposableEffect，状态转换类使用 produceState/snapshotFlow；
3. Effect Keys 配置需遵循 “使用即传入” 原则，无需重启的变量通过 rememberUpdatedState 包装，平衡功能正确性与性能。



TODOList:

LocalLifecycleOwner