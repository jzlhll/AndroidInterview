# Compose CompositionLocal

CompositionLocal 是 Compose 实现**组合树中数据隐式向下传递**的核心工具，是开发必备机制，日常多使用内置的`LocalXXX`（如`LocalContext`），也是 MaterialTheme 底层实现，解决高频通用数据（主题、上下文等）层层显式传参的繁琐问题。

## 一、概念与特性

1. **核心作用**：在组合树某节点为其提供值，后代可组合函数无需将其声明为参数，通过`current`直接获取，实现隐式传参。

2. **关键特性**

   - 作用域：同一实例可在组合树不同层级提供不同值，后代取**最近祖先**的`current`值；

   - 重组：值更新时，仅重组读取该值的内容（自定义 API 决定重组范围）；

   - 底层支撑：MaterialTheme 的`colorScheme/typography/shapes`，底层对应`LocalColorScheme/LocalTypography/LocalShapes`。

3. **易混概念区分**

   - Composition：可组合函数的调用图记录；

   - UI 树：组合过程构建的`LayoutNode`树，二者不可混用。

## 二、常用CompositionLocal

日常开发无需自定义，直接使用 Compose 框架已提供值的`LocalXXX`，通过`LocalXXX.current`调用，核心常用项及用法如下：

```Kotlin
@Composable
fun CommonUsage() {
    // 1. 获取当前上下文（最常用的CompositionLocal之一）
    val context = LocalContext.current
    // 2. 获取生命周期持有者（你之前问的LocalLifecycleOwner）
    val lifecycleOwner = LocalLifecycleOwner.current
    // 3. 获取屏幕密度/尺寸相关配置
    val density = LocalDensity.current
    // 4. 获取屏幕方向、语言等设备配置
    val configuration = LocalConfiguration.current
    // 5. 获取Compose的主题配置（比如Material3的颜色/字体）
    val colors = LocalColors.current
    val typography = LocalTypography.current

    // 实际业务场景：比如用LocalContext弹Toast
    Button(onClick = {
        Toast.makeText(context, "使用LocalContext", Toast.LENGTH_SHORT).show()
    }) {
        Text("点击弹Toast")
    }
}
```

其他常用：`LocalContentColor`（文本/图标首选颜色）、`LocalShapes`（Material 形状配置）。



## 三、手动自定义 CompositionLocal（较麻烦后续学习）

内置无法满足业务树作用域传参时可自定义，需遵循**创建-提供-消费**流程，且实例命名**必须以Local为前缀**（IDE 友好）。

### 1. 两个创建 API（核心区别，性能关键）

| API                        | 重组特性                                    | 适用场景                                            |
| -------------------------- | ------------------------------------------- | --------------------------------------------------- |
| `compositionLocalOf`       | 跟踪读取，值变仅重组**读取该值**的内容      | 数据**可能动态变化**（如随系统主题变化的配置）      |
| `staticCompositionLocalOf` | 不跟踪读取，值变重组**整个提供值的content** | 数据**几乎/永远不变**（固定设计系统配置），性能更高 |

### 2. 自定义三步法（以LocalElevations为例）

#### 步骤1：定义数据载体+创建实例（必须带默认值）

```Kotlin
data class Elevations(val card: Dp = 0.dp, val default: Dp = 0.dp)
val LocalElevations = compositionLocalOf { Elevations() } // 带默认值
```

#### 步骤2：通过CompositionLocalProvider提供值

用`provides`中缀函数绑定值，作用于其后代组合树：

```Kotlin
setContent {
    val elevations = if (isSystemInDarkTheme()) Elevations(1.dp) else Elevations()
    CompositionLocalProvider(LocalElevations provides elevations) {
        // 后代可获取上述elevations值
    }
}
```

#### 步骤3：后代消费值

```Kotlin
@Composable
fun SomeComposable() {
    MyCard(elevation = LocalElevations.current.card) { /* 内容 */ }
}
```

## 四、使用判定条件（禁止过度使用）

仅满足以下**两个条件**才适合使用，否则优先选替代方案：

1. 有**良好的默认值**；若无，需保证开发者无法在未提供值时使用（避免预览/测试报错）；

2. 数据为**树/子层级作用域**，可被组合树**任意后代**使用，而非少数后代。

### 不建议使用的场景

为单个页面的`ViewModel`创建 CompositionLocal（并非所有后代都需要，违反树作用域原则）。

### 过度使用的弊端

1. 隐式依赖导致代码行为**难以推理**，无法从函数参数直接看出依赖；

2. 调试困难，数据可在任意节点修改，需向上遍历组合树查找值的提供位置（可通过 IDE「Find usages」/Compose 布局检查器缓解）。

**适用层级**：更适合基础架构层，Compose 自身大量使用，业务层需谨慎自定义。

## 五、非适用场景的替代方案

不满足上述条件时，优先选择以下方案，让依赖更清晰，代码更易复用/测试。

### 方案1：显式传参（最推荐）

遵循**最小传参原则**+**状态向下、事件向上**，仅传递后代实际需要的信息，而非整个对象：

```Kotlin
@Composable
fun MyComposable(vm: MyViewModel = viewModel()) {
    MyDescendant(vm.data) // 仅传需要的data，而非整个ViewModel
}
@Composable
fun MyDescendant(data: DataToDisplay) { Text(data.content) }
```

### 方案2：控制反转

子组件通过**回调函数**暴露操作入口，父组件处理具体业务逻辑，实现父子解耦：

```Kotlin
@Composable
fun MyComposable(vm: MyViewModel = viewModel()) {
    ReusableButton(onClick = { vm.loadData() }) // 父组件处理逻辑
}
@Composable
fun ReusableButton(onClick: () -> Unit) { Button(onClick) { Text("加载") } } // 通用子组件
```

### 方案3：组合式 Content Lambda

父组件将带业务逻辑的可组合内容，通过`@Composable ()->Unit`传递给子组件，子组件仅负责通用布局：

```Kotlin
@Composable
fun MyComposable(vm: MyViewModel = viewModel()) {
    ReusableLayout { Button(onClick = { vm.loadData() }) { Text("确认") } }
}
@Composable
fun ReusableLayout(content: @Composable () -> Unit) { Column { content() } }
```

## 六、总结

1. CompositionLocal 是隐式传参核心，**内置LocalXXX为开发高频使用项**，直接通过`current`调用；

2. 自定义需选合适 API，遵循**创建-提供-消费**流程，实例命名以Local为前缀，且必须带默认值；

3. 禁止过度使用，仅适用于**有默认值+树作用域**的场景，不可为单个ViewModel自定义；

4. 非适用场景优先选择**显式传参、控制反转、Content Lambda**，保证依赖清晰、代码可复用。

