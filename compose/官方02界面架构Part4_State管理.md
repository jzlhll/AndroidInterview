

Compose 是声明式 UI 框架，其核心规则是：**UI = f(State)**（UI 是状态的函数）。

1. **Composition（组合）**：Jetpack Compose 执行可组合项（`@Composable`函数）时，构建出的 UI 描述树，包含所有 UI 元素的属性、布局和交互逻辑。
2. **Initial composition（初始组合）**：首次调用可组合项，生成初始 Composition 的过程，是 UI 首次渲染的基础。
3. **Recomposition（重组）**：当可观察状态发生变化时，Compose 会**增量式**重新执行受该状态影响的可组合项，更新 Composition 的过程（仅更新状态变化相关的 UI 部分，而非全部重新渲染）。
4. 应用中的**状态（State）**:  指任何可以随时间变化的值，典型示例包括：网络异常时弹出的 Snackbar、博客文章及评论内容、用户点击按钮时播放的涟漪动画、用户在图片上绘制的贴纸等。



## 一、State & Composition（状态与组合）

### 

### 1. 最简化认知

|         用法组合          |       适用场景        |         核心作用         |
| :-----------------------: | :-------------------: | :----------------------: |
| remember + mutableStateOf |   数据变化需更新 UI   |  跨重组保留 + 触发重组   |
|     单独使用 remember     | 仅保存对象，不更新 UI | 仅跨重组保留，无观察需求 |

#### 单独使用 remember（无需搭配 mutableStateOf）

- 场景 1：存储不可变对象

  （静态资源、画笔，初始化后不修改）

  ```kotlin
  // 仅缓存画笔实例，无需监听变化
  val brush = remember(key1 = avatarRes) { ShaderBrush(...) }
  ```

- 场景 2：存储**高成本初始化对象**（自定义状态类、复杂工具类，依赖 key 变化重建）

- 场景 3：存储无需 UI 响应的对象（临时工具、局部变量，变化不刷新 UI）

  ```kotlin
  // 缓存随机工具类，仅保留实例，不触发UI更新
  val random = remember { Random() }
  ```

#### 必须组合：remember + mutableStateOf（需要监听变化）

- 场景：UI 交互状态（输入框、计数、开关，数据变化必须刷新 UI）

  ```kotlin
  // 输入框状态：变化触发UI重组，必须组合使用
  var name by remember { mutableStateOf("") }
  ```

### 2. mutableStateOf

**定义：创建一个持有普通数据的`MutableState`容器，值发生变化时会被 Compose 感知，并自动触发依赖该值的可组合项重组；**

* **三种等价的状态声明方式**

Compose 提供了三种语法糖，用于声明`MutableState`对象，功能完全等价，可根据代码可读性选择：

```kotlin
//方式1：基础赋值方式
val mutableState = remember { mutableStateOf(default) }

//方式2：属性委托方式（推荐，最简洁）
import androidx.compose.runtime.getValue
import androidx.compose.runtime.setValue
var value by remember { mutableStateOf(default) }

//方式3：解构声明方式
val (value, setValue) = remember { mutableStateOf(default) }
```


* **注意事项**

**禁止使用不可观察的可变对象作为状态**，例如`ArrayList<T>`、`mutableListOf()`、自定义可变数据类。这类对象的内部变化无法被 Compose 观察到，不会触发重组，会导致 UI 显示过期 / 错误数据。

**推荐用法**：使用`State<List<T>>` + 不可变的`listOf()`，每次更新都传入新的 List 实例，通过修改`MutableState`的`value`触发重组。

```kotlin
// 正确写法
val listState by remember { mutableStateOf(listOf<String>()) }
// 更新方式：创建新的List实例，修改MutableState的value，触发重组
listState = listState + "新元素"
```

另外， `stateOf()` 无观察能力（即使强行修改也无重组）。



> `MutableState<T>` 的可观察能力，由**Compose 编译器插件 + 运行时快照系统**共同实现：
>
> 1. 编译期：编译器插件会对`value`的`get/set`方法进行插桩处理
> 2. 读操作：当可组合项读取`value`时，会自动将该可组合项注册为这个状态的观察者
> 3. 写操作：当`value`被修改时，会通知所有注册的观察者，调度对应可组合项触发重组
>
> 接口仅定义了状态的读写契约，核心的观察逻辑在运行时的实现类（如`SnapshotMutableState`）中，因此接口源码中看不到相关逻辑。



### 3. remember

**定义**：让数据 / 对象跨 Compose 重组保留，组件移除时自动回收。

- `remember` 可以将对象存储在内存中，初始组合时计算的值会被存入组合中，重组时会直接返回已存储的值，支持存储可变与不可变对象。

- 状态应该声明在**使用它的最低节点**，一方面可以避免不必要的重组（只有使用该状态的子节点会触发重组），另一方面遵循状态就近原则，提升代码可读性与可维护性。

- `remember` 存储的对象与调用它的可组合项生命周期强绑定：当调用`remember`的可组合项从 Composition 中移除时，存储的对象会被回收（被遗忘）。

- `remember { mutableStateOf("") }`：让字符串既具备 “变化监听 + 触发重组” 能力，又能跨重组保留状态，组件移除时自动回收该状态容器。

**带 key 的 remember：控制缓存失效时机**

​	默认情况下，`remember` 仅在初始组合时执行一次计算，后续重组直接返回缓存值。当传入`key`（一个或多个）参数时，**任何一个 key 发生变化，`remember`都会失效缓存，重新执行计算 lambda**。

这个特性适用于：

- 初始化成本高的对象（如`ShaderBrush`），仅当依赖参数变化时才重新创建
- 状态持有者实例，仅当核心配置（如窗口尺寸）变化时才重新初始化

**示例 1：依赖资源的高成本对象初始化**

```kotlin
@Composable
private fun BackgroundBanner(
    @DrawableRes avatarRes: Int,
    modifier: Modifier = Modifier,
    res: Resources = LocalContext.current.resources
) {
    // 仅当avatarRes变化时，才重新创建ShaderBrush
    val brush = remember(key1 = avatarRes) {
        ShaderBrush(
            BitmapShader(
                ImageBitmap.imageResource(res, avatarRes).asAndroidBitmap(),
                Shader.TileMode.REPEAT,
                Shader.TileMode.REPEAT
            )
        )
    }
    Box(modifier = modifier.background(brush)) {
        /* 内容 */
    }
}
```

**示例 2：状态持有者的动态更新**

```kotlin
@Composable
private fun rememberMyAppState(windowSizeClass: WindowSizeClass): MyAppState {
    // 仅当windowSizeClass变化时，才重新创建MyAppState实例
    return remember(windowSizeClass) {
        MyAppState(windowSizeClass)
    }
}

@Stable
class MyAppState(private val windowSizeClass: WindowSizeClass) {
    /* 业务逻辑与状态管理 */
}
```

只有当状态发生变化且被 Compose 观察到，才会触发重组（Recomposition）来更新 UI；任何脱离可观察状态的 UI 变更，都无法在 Compose 中生效。

比如TextField，不会自己更新的示例:

```kotlin
///////错误做法
@Composable
private fun HelloContent() {
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello!",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.bodyMedium
        )
        // ❌ 错误：value固定为空字符串，onValueChange未更新任何状态
        OutlinedTextField(
            value = "",
            onValueChange = { },
            label = { Text("Name") }
        )
    }
}

////////////正确做法
@Composable
private fun HelloContent() {
    // ✅ 正确：用remember+mutableStateOf创建可观察状态
    var name by remember { mutableStateOf("") }
    
    Column(modifier = Modifier.padding(16.dp)) {
        Text(
            text = "Hello, ${name.ifEmpty { "Stranger" }}!",
            modifier = Modifier.padding(bottom = 8.dp),
            style = MaterialTheme.typography.bodyMedium
        )
        // ✅ 正确：value绑定可观察状态，onValueChange更新状态
        OutlinedTextField(
            value = name,
            onValueChange = { name = it }, // 更新状态，触发重组
            label = { Text("Name") }
        )
    }
}
```

**错误原因**：

- `value` 参数是固定的空字符串，未绑定任何可观察状态；
- `onValueChange` 仅接收输入内容，但未对其做任何处理；
- 无状态变化 → Compose 不会触发重组 → UI 始终显示空字符串。

**正确原理**：

1. `remember { mutableStateOf("") }` 创建了与 Compose 生命周期绑定的可观察状态；
2. `value = name` 将 TextField 的显示值与可观察状态绑定；
3. 用户输入时，`onValueChange` 回调更新`name`的值 → 触发 Compose 重组；
4. 重组时，`OutlinedTextField` 读取更新后的`name`值 → UI 同步显示输入内容。



### 4. rememberSaveable

### 状态存储方式

`rememberSaveable` 原生支持所有可存入 Bundle 的数据类型，对于自定义复杂类型，有三种实现状态持久化的方案：

**方案 1：@Parcelize 注解（最简单）**

给自定义数据类添加`@Parcelize`注解，实现`Parcelable`接口，即可被自动序列化存入 Bundle。**要求类中的所有字段都必须是 Bundle 支持的类型（基本类型、String、Parcelable 等）**。

示例：

```kotlin
@Parcelize
data class City(
    val name: String,
    val country: String
) : Parcelable

@Composable
fun CityScreen() {
    // 自动支持跨配置变更持久化
    var selectedCity by rememberSaveable {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

**方案 2：自定义 MapSaver**

当`@Parcelize`不适用时，可通过`mapSaver`自定义规则，将对象转为 Bundle 支持的 Map 集合，恢复时再转回原对象。

示例：

```kotlin
data class City(val name: String, val country: String)

// 自定义MapSaver规则
val CitySaver = run {
    val nameKey = "Name"
    val countryKey = "Country"
    mapSaver(
        save = { city -> mapOf(nameKey to city.name, countryKey to city.country) },
        restore = { map -> City(map[nameKey] as String, map[countryKey] as String) }
    )
}

@Composable
fun CityScreen() {
    var selectedCity by rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

**方案 3：自定义 ListSaver**

无需定义 Map 的 key，直接用列表索引作为 key，简化序列化逻辑，适用字段顺序固定的简单数据类。

示例：

```kotlin
data class City(val name: String, val country: String)

// 自定义ListSaver规则
val CitySaver = listSaver<City, Any>(
    save = { city -> listOf(city.name, city.country) },
    restore = { list -> City(list[0] as String, list[1] as String) }
)

@Composable
fun CityScreen() {
    var selectedCity by rememberSaveable(stateSaver = CitySaver) {
        mutableStateOf(City("Madrid", "Spain"))
    }
}
```

#### 带 inputs 的 rememberSaveable：控制缓存失效

`rememberSaveable` 用`inputs`参数实现和`remember`的`key`参数完全一致的能力：当任何一个`inputs`参数发生变化时，会失效缓存，重新执行计算 lambda。

示例：

```kotlin
var userTypedQuery by rememberSaveable(
    typedQuery,  //`rememberSaveable`的`inputs`参数，当外部传入的`typedQuery`变化时，会重新执行 lambda，用新值初始化输入状态。
    stateSaver = TextFieldValue.Saver //传入 TextFieldValue 内置的 Saver 实现，告诉系统如何将这个复杂类型序列化 / 反序列化到 Bundle 中，实现跨配置变更的状态持久化。
) {
    mutableStateOf(
        TextFieldValue(text = typedQuery, selection = TextRange(typedQuery.length)) //官方库的类，封装了 TextField 的完整状态，包括输入文本`text`、光标 / 选中范围`selection`、IME 编辑状态等，比单纯的 String 更适合复杂输入场景。
    )
}
```

创建一个可跨配置变更保留的输入状态，当外部`typedQuery`变化时自动重置输入框的文本与光标位置，保证屏幕旋转等场景下状态不丢失。



#### 比较remember & rememberSaveable & ViewModel

| 特性                                          | remember               | rememberSaveable                   | ViewModel                                      |
| --------------------------------------------- | ---------------------- | ---------------------------------- | ---------------------------------------------- |
| 跨重组保留状态                                | ✅                      | ✅                                  | ✅                                              |
| 跨配置变更（屏幕旋转、语言切换等）保留状态    | ❌                      | ✅                                  | ✅                                              |
| 跨系统发起的 Activity 重建 / 进程死亡保留状态 | ❌                      | ✅                                  | ❌（需配合 SavedStateHandle 才 ✅）              |
| 底层实现                                      | Composition 内存缓存   | Android Saved Instance State 机制  | ViewModelStore + ViewModelProvider             |
| 状态存储位置                                  | Compose 组合树         | Activity 所属 Bundle               | 应用进程内存（堆内存）                         |
| 适用状态范围                                  | 单个可组合项 / 局部 UI | 单个可组合项 / 局部 UI（需持久化） | 跨 UI 组件（如多个 Composable / 页面）共享状态 |

##### 跨系统发起的 Activity 重建 / 进程死亡保留状态

- **默认行为（❌）**：ViewModel 的核心生命周期是 “从首次创建到 Activity/Fragment 彻底销毁（用户主动关闭）”，当系统因内存不足杀死进程、或强制重建 Activity 时，ViewModel 实例会被销毁，其内部状态也会丢失。

- 补救方案（✅）需配合 `SavedStateHandle` 使用（ViewModel 构造函数注入），`SavedStateHandle` 底层复用 Android Saved Instance State 机制，可将关键状态存入 Bundle，实现进程死亡后的状态恢复：

  ```kotlin
  class MyViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
      // 用SavedStateHandle存储状态，支持进程死亡恢复
      val count = savedStateHandle.getStateFlow("count", 0)
      
      fun increment() {
          savedStateHandle["count"] = count.value + 1
      }
  }
  ```

##### 底层实现与生命周期

- ViewModel 由 `ViewModelProvider` 创建，存储在 `ViewModelStore` 中（Activity/Fragment 作为 `ViewModelStoreOwner` 持有）；
- 配置变更时，Activity/Fragment 重建，但 `ViewModelStore` 会被系统保留，因此 ViewModel 实例不变；
- 与 Compose 无关：ViewModel 的生命周期不依赖 Compose 组合树，即使 Composable 被移除 / 重组，ViewModel 仍会保留（直到宿主销毁）。

##### 适用场景差异

- `remember`：仅用于 Composable 内部、无需跨配置变更的临时状态（如单个按钮的点击状态）, 当前节点的临时缓存，仅跨重组保留；
- `rememberSaveable`：仅用于 Composable 内部、底层复用 Bundle，支持配置变更 / 进程死亡恢复（如输入框文本、单选按钮选中状态）；
- `ViewModel`：用于跨 UI 组件共享的业务状态，或需要脱离 UI 生命周期的状态（如网络请求、数据解析）。默认仅支持配置变更保留，需 `SavedStateHandle` 才能实现进程死亡恢复，适合存储业务级状态。



## 二、其他支持的状态类型

Compose 不强制要求必须使用`MutableState<T>`持有状态，它支持所有主流的可观察类型，但**核心规则是：在 Compose 中读取其他可观察类型前，必须先转为`State<T>`，才能让 Compose 在状态变化时自动触发重组**。

重点掌握Flow和LiveData的使用。

### 1. Flow 转 State

Compose 提供了两个核心 API，用于将 Flow 转为 State：

##### collectAsStateWithLifecycle ()（Android 推荐用法）

- 特性：生命周期感知，仅当宿主组件处于`STARTED`及以上生命周期时收集 Flow，能有效节省应用资源、避免内存泄漏，是 Android 平台收集 Flow 的首选方案。

```kotlin
// ViewModel 中定义Flow
class UserViewModel : ViewModel() {
    val userFlow: Flow<User> = userRepository.getUser()
}

// Composable 中使用
@Composable
fun UserScreen(viewModel: UserViewModel) {
    val user by viewModel.userFlow.collectAsStateWithLifecycle(
        initialValue = User.Loading
    )
    when (user) {
        is User.Loading -> LoadingView()
        is User.Success -> UserInfoView(user.data)
        is User.Error -> ErrorView(user.message)
    }
}
```


##### collectAsState ()（平台无关用法）

- 特性：无生命周期感知，只要可组合项处于 Composition 中就会持续收集 Flow，适合跨平台的 Compose 项目，无需额外依赖。

### 2. LiveData 转 State

通过`observeAsState()` API，可将 LiveData 转为 Compose State，自动观察 LiveData 的数值变化。

```kotlin
// ViewModel 中定义LiveData
class CounterViewModel : ViewModel() {
    private val _count = MutableLiveData(0)
    val count: LiveData<Int> = _count

    fun increment() {
        _count.value = (_count.value ?: 0) + 1
    }
}

// Composable 中使用
@Composable
fun CounterScreen(viewModel: CounterViewModel) {
    val count by viewModel.count.observeAsState(initial = 0)
    Column {
        Text("当前计数: $count")
        Button(onClick = { viewModel.increment() }) {
            Text("+1")
        }
    }
}
```

### 3. 其他支持的类型

Compose 还支持 RxJava2/RxJava3 的响应式流，通过`subscribeAsState()` API 转为 State，需添加对应的`runtime-rxjava2`/`runtime-rxjava3`依赖，用法与上述 API 一致。



### 4. 自定义可观察类型转 State

对于自定义的可观察类型，可通过`produceState` API 转为`State<T>`。`produceState` 会创建一个 State，可将任意异步 / 可观察数据源的变化同步到 State 中，同时支持 key 控制重启、自动资源清理。

示例（自定义回调式 API 转 State）：

```kotlin
// 自定义可观察数据源
class LocationManager {
    fun requestLocationUpdates(callback: (Location) -> Unit) {
        // 模拟位置更新回调
    }
    fun removeUpdates(callback: (Location) -> Unit) {
        // 移除监听，清理资源
    }
}

// Composable 中使用produceState转为State
@Composable
fun LocationScreen(locationManager: LocationManager) {
    val currentLocation by produceState<Location?>(
        initialValue = null,
        key1 = locationManager
    ) {
        // 生产者逻辑，运行在协程中
        val callback = { location: Location ->
            value = location // 给State赋值，触发重组
        }
        locationManager.requestLocationUpdates(callback)
        
        // 可组合项移除/key变化时，自动执行清理逻辑
        awaitDispose {
            locationManager.removeUpdates(callback)
        }
    }

    Text("当前位置: ${currentLocation?.toString() ?: "获取中"}")
}
```



## 三、有状态和无状态

**核心定义**

- **有状态可组合项**：在内部使用`remember`/`rememberSaveable`持有状态的可组合项，状态由自身管理，调用方无需控制状态。
- **无状态可组合项**：不持有任何内部状态，所有状态都通过参数传入，所有事件都通过回调向上传递，完全由调用方控制。

**有状态可组合项示例：**

```kotlin
@Composable
fun StatefulCounter() {
    // 内部持有状态，外部无法访问、控制
    var count by remember { mutableStateOf(0) }
    Column {
        Text("计数: $count")
        Button(onClick = { count++ }) {
            Text("增加")
        }
    }
}
```

- 优点：使用简单，调用方无需管理状态，直接调用`StatefulCounter()`即可
- 缺点：复用性差、可测试性差，外部无法控制状态（如无法从外部重置计数）

**无状态可组合项示例：**

```kotlin
// 无状态组件：所有状态/事件都交给外部控制
@Composable
fun StatelessInput(
    // 外部传入的状态（供渲染）
    inputValue: String,
    // 外部传入的回调（事件上抛）
    onInputChange: (String) -> Unit,
    // 额外的UI配置（也由外部控制）
    hint: String,
    modifier: Modifier = Modifier
) {
    // 组件仅负责渲染，无任何状态管理
    OutlinedTextField(
        value = inputValue,
        onValueChange = onInputChange, // 直接转发事件给外部
        label = { Text(hint) },
        modifier = modifier
    )
}

// 调用者：管理状态 + 处理事件
@Composable
fun InputScreen() {
    // 调用者持有状态（可自行决定用remember/rememberSaveable）
    var username by rememberSaveable { mutableStateOf("") }
    var password by rememberSaveable { mutableStateOf("") }

    Column {
        // 复用无状态组件：控制“用户名输入框”的状态和行为
        StatelessInput(
            inputValue = username,
            onInputChange = { username = it }, // 外部处理状态变更
            hint = "请输入用户名"
        )

        // 复用同一个无状态组件：控制“密码输入框”的状态和行为
        StatelessInput(
            inputValue = password,
            onInputChange = { password = it }, // 不同的状态处理逻辑
            hint = "请输入密码"
        )

        // 外部可自由控制状态（如清空）
        Button(onClick = {
            username = ""
            password = ""
        }) {
            Text("清空输入")
        }
    }
}
```

- 优点：完全解耦、复用性极强、可测试性好，调用方可完全控制状态与行为，是 Compose **推荐**的写法

- 最佳实践：当组件的状态需要**外部控制 / 复用 / 定制**时，必须设计为无状态组件，通过 “状态入参 + lambda 回调” 把管理权交给调用者；

  无状态组件的核心是 “纯渲染”，不持有状态、不处理业务逻辑，只做 UI 映射；

  有状态组件仅适用于 “状态无需外部控制” 的极简场景（如独立的、不可复用的局部 UI）。



## 四、状态提升 & 状态容器

https://developer.android.google.cn/develop/ui/compose/state-hoisting

**根据状态的「作用域（哪些组件需要访问）」和「生命周期（和哪个组件绑定）」** 选择存储位置，避免状态混乱、生命周期不一致（如屏幕旋转丢失数据）、UI 与业务逻辑耦合等问题。

通过「分层存储」实现**状态的最小作用域、单向数据流、UI 与业务解耦**。

### 1. 状态存在单个 Composable 节点内部（局部状态）

状态**仅当前 Composable 使用**，无任何子 / 父组件依赖，生命周期和该 Composable 完全绑定（组件销毁，状态消失）。

私有化、外部不可访问，轻量易维护；用`remember`（内存缓存，重组保留、配置变更丢失）或`rememberSaveable`（持久化，屏幕旋转 / 配置变更可恢复）实现。

示例：

按钮的高亮状态、单个卡片的展开 / 折叠、输入框的临时本地文本（无需提交 / 共享），比如一个独立的「收藏按钮」：

```kotlin
@Composable
fun CollectButton() {
    // 局部状态：仅当前按钮使用，记录是否已收藏
    val isCollected by remember { mutableStateOf(false) }
    IconButton(onClick = { isCollected = !isCollected }) {
        Icon(
            imageVector = if (isCollected) Icons.Filled.Star else Icons.Outlined.Star,
            tint = if (isCollected) Color.Yellow else Color.Gray,
            contentDescription = "收藏"
        )
    }
}
```

**为什么不放在高层 / VM**：完全无共享需求，过度提升会增加代码冗余，违背「状态最小作用域」原则。

### 2. 状态存在更高层的 Composable 节点中（跨子组件的组合状态）

状态需要被**当前高层 Composable 的多个子 Composable 共享**，生命周期和高层组件绑定，**无跨页面 / 导航的需求**。

作用域覆盖所有子组件，遵循**单向数据流**：高层持有状态（唯一数据源），子组件通过**参数接收状态**、通过**回调函数更新状态**，子组件自身无状态（纯 UI 组件）；仍用`remember/rememberSaveable`（属于 Compose 组合树内管理）。

示例：

登录页面（高层`LoginScreen`）持有「用户名 / 密码」状态，共享给子组件「用户名输入框、密码输入框、登录按钮」（按钮需要判断用户名 / 密码是否为空）：

```kotlin
// 高层Composable：登录页面，持有共享状态
@Composable
fun LoginScreen() {
    val userName by remember { mutableStateOf("") }
    val pwd by remember { mutableStateOf("") }
    // 子组件1：用户名输入框（仅接收状态+更新回调）
    UserNameInput(userName = userName, onUserNameChange = { userName = it })
    // 子组件2：密码输入框（同理）
    PwdInput(pwd = pwd, onPwdChange = { pwd = it })
    // 子组件3：登录按钮（接收状态判断是否可点击）
    LoginButton(canLogin = userName.isNotBlank() && pwd.isNotBlank()) {
        // 登录点击逻辑
    }
}

// 子Composable：纯UI，无状态
@Composable
fun UserNameInput(userName: String, onUserNameChange: (String) -> Unit) {
    OutlinedTextField(
        value = userName,
        onValueChange = onUserNameChange,
        label = { Text("用户名") }
    )
}
```

避免多个子组件各自持有状态导致的**状态不一致**，高层统一管理，子组件可复用（比如其他页面也能调用`UserNameInput`，只需传不同的状态 / 回调）。

### 3. 状态存在ViewModel 中（页面级 / 跨生命周期的业务状态）

满足以下任一条件即可，也是官方最推荐的**业务状态管理方式**：

1. 状态需要**跨配置变更**（如屏幕旋转、分屏，Compose 重组但 ViewModel 不销毁）；
2. 状态和**业务逻辑绑定**（如网络请求、数据库操作、数据解析）；
3. 状态需要**跨页面导航保留**（如从列表页到详情页，返回后列表的筛选状态不变）；
4. 状态需要被**多个无关的 Composable 共享**（非父子关系）。

#### 核心特点

- ViewModel 的生命周期和**页面（Activity/Fragment/NavBackStackEntry）** 绑定，不受 Compose 重组影响；
- 持有**业务逻辑**，实现**UI 与业务解耦**（Compose 仅做 UI 展示，不处理逻辑）；
- 用`StateFlow/LiveData`（推荐 StateFlow）保存状态，Compose 通过`collectAsStateWithLifecycle`观察状态变化；
- 用`viewModelScope`处理异步逻辑（协程随 ViewModel 销毁自动取消，避免内存泄漏）。

#### 实际示例

用户信息页面，需要网络请求获取用户数据，且屏幕旋转时数据不丢失：

```kotlin
// 第一步：ViewModel层——持有状态+业务逻辑，和UI完全解耦
class UserInfoViewModel : ViewModel() {
    // 私有可变状态，外部仅可观察
    private val _userInfo = MutableStateFlow<UserInfo?>(null)
    // 公共只读状态，Compose观察
    val userInfo: StateFlow<UserInfo?> = _userInfo.asStateFlow()

    // 业务逻辑：网络请求获取用户信息
    fun fetchUserInfo(userId: String) {
        viewModelScope.launch {
            // 模拟网络请求（实际调用Repository层）
            val data = UserRepository.getUserInfo(userId)
            _userInfo.emit(data)
        }
    }
}

// 第二步：Compose UI层——仅观察状态+展示UI，无业务逻辑
@Composable
fun UserInfoScreen(userId: String) {
    // 获取ViewModel，自动关联页面生命周期
    val viewModel: UserInfoViewModel = viewModel()
    // 观察StateFlow状态，跟随生命周期感知
    val userInfo by viewModel.userInfo.collectAsStateWithLifecycle()

    // 页面首次加载时请求数据（LaunchedEffect：仅执行一次）
    LaunchedEffect(Unit) {
        viewModel.fetchUserInfo(userId)
    }

    // 根据状态展示不同UI
    when (userInfo) {
        null -> CircularProgressIndicator() // 加载中
        else -> Column(Modifier.padding(16.dp)) {
            Text("姓名：${userInfo.name}", fontSize = 18.sp)
            Text("手机号：${userInfo.phone}", fontSize = 16.sp, color = Color.Gray)
        }
    }
}
```

**为什么不放在 Compose 中**：如果把用户信息状态放在高层 Composable，屏幕旋转时 Compose 重组，状态会丢失，需要重新请求网络；且业务逻辑和 UI 耦合，代码难以测试和维护。



1. **单向数据流**：无论状态放哪，都遵循「UI 触发事件 → 事件修改状态 → 状态驱动 UI 更新」，禁止子组件直接修改高层 / VM 的状态，只能通过回调 / 方法触发，保证状态变更可追溯；
2. **状态最小化**：只把必要的状态提升到高层 / VM，不要过度提升（比如局部的按钮高亮状态），减少状态的作用域，降低维护成本；
3. **UI 无状态化**：尽可能让 Compose 组件成为「纯 UI 组件」（仅接收参数 + 回调），状态和业务逻辑抽离到高层 / VM，提升组件的复用性。
