https://developer.android.google.cn/develop/ui/compose/lifecycle?hl=zh-cn

## Composable函数的生命周期

Compose 是「声明式 UI」框架，声明**状态与 UI 的映射关系**，框架会自动维护内部的 UI 结构树，并处理 UI 的更新逻辑。本章节，目的是，了解composable函数的生命周期和如何确定重组。

可组合对象的生命周期比Activity、View， Fragment的生命周期更简单。当Composable需要与更复杂生命周期的外部资源交互时，您应该使用`Effect`。

以下围绕代码执行、节点的增删改，拆解 Compose 的核心概念、生命周期，以及关键的性能优化逻辑。

### 一、概念

#### 1. Composition（组合）

Composition 是可组合函数首次执行（初始组合）时，框架在内部生成的**可组合函数实例树状结构**，可理解为框架存储 UI 结构的 “UI 骨架节点树”。

- 示例：首次执行下方代码时，框架会在 Composition 中创建`Column`节点，以及其下嵌套的`Text`, `Button`子节点，形成层级树：

  ```kotlin
  @Composable fun CounterPage() {
      Column { // 生成Column节点
          Text("计数：0") // 生成Text子节点
          Button(onClick = {}) { Text("+1") } // 生成Button子节点
      }
  }
  ```

- 关键规则：Composition 仅由初始组合生成，后续只能通过**重组**更新，无法手动修改。

#### 2. Recomposition（重组）

重组是当 `State<T>` 状态变化时，框架**仅重新执行读取该状态的可组合函数**，并更新` Composition` 树中对应节点的过程（最终同步到屏幕）。

- 示例：仅读取状态的函数会重组，未读取的函数不受影响：

  ```kotlin
  val count = remember { mutableStateOf(0) }
  
  @Composable fun CountText() {
      Text("计数：${count.value}") // 读取count，状态变化时重组
  }
  
  @Composable fun CounterPage() {
      Column {
          CountText() // 仅该函数因count变化重组
          Text("固定文本") // 不读count，永不重组
          Button(onClick = { count.value++ }) { Text("+1") }
      }
  }
  ```

#### 3. Composable Instance（可组合函数实例）

可组合函数实例是可组合函数被调用后，在 Composition 树中创建的**独立节点**—— 可组合函数本身是 “生成节点的模板”，实例才是树中的具体节点。

- 示例：同一个可组合函数多次调用会生成多个独立节点，彼此生命周期无关：

  ```kotlin
  ////////////// 示例01 //////////////
  @Composable fun MyText(text: String) { Text(text) }
  
  @Composable fun Test() {
      MyText("A") // 生成实例1（节点1）
      MyText("B") // 生成实例2（节点2）
  }
  
  ////////////// 示例02 //////////////
  @Composable fun CounterPage() {
      Column { // 生成Column节点
          Text("计数：0") // 生成Text子节点
          Button(onClick = {}) { Text("+1") } // 生成Button子节点
      }
  }
  ```

#### 4. Call Site（调用点）

调用点是可组合函数被调用的**源码位置**（嵌套层级、行号、调用者），是框架判断 “Composition 树中某个节点是否为原有节点” 的核心依据。

- 示例：调用点不变则节点保留，调用点消失 / 新增则节点移除 / 新增：

  ```kotlin
  @Composable fun LoginScreen(showError: Boolean) {
      if (showError) { LoginError() } // 调用点1：仅showError=true时生成节点
      LoginInput() // 调用点2：始终生成节点
  }
  ```
  
  当 `showError` 从 `false` 变 `true`，`LoginInput`的调用点未变，其节点会被保留；`LoginError` 调用点从 “不存在” 变为 “存在”，会新增节点。
  

### 二、生命周期

可组合函数的生命周期，本质是其对应节点在 Composition 树中 **“添加 - 更新 - 移除”** 的全过程，仅包含 3 个阶段：

#### 1. 进入组合

- 触发：可组合函数首次被调用，对应的节点被**添加**到 Composition 树。
- 示例：打开 `CounterPage` 时，`Column`/`CountText`/`Button` 节点被首次添加到 Composition 树，函数执行并渲染初始 UI。

#### 2. 重组

- 触发：节点对应的函数读取的 `State` 变化，且函数无法被跳过（依赖 “稳定类型” 判断）。
- 示例：点击按钮使 `count.value` 变化，`CountText` 节点对应的函数重新执行，节点本身不销毁重建，仅更新文本内容；可多次触发，也可永不触发（如 “固定文本” 节点）。

#### 3. 离开组合

- 触发：可组合函数的调用点消失，对应的节点被**移除**出 Composition 树。
- 示例：关闭 `CounterPage` 页面、`LoginScreen` 中 `showError` 从 `true` 变 `false`（`LoginError` 节点移除）、列表删除某一项（对应节点移除）；节点离开后，关联的 `LaunchedEffect` 等副作用会自动终止。

### 三、优化方向

#### 1. key：优化列表节点标识

循环生成节点时，仅靠执行顺序标识节点会导致增删列表项时大量节点销毁重建，`key` 可自定义节点的唯一标识：

```kotlin
@Composable fun MoviesScreen(movies: List<Movie>) {
    Column {
        for (movie in movies) {
            key(movie.id) { // 用唯一ID标识节点，仅id消失时节点离开组合
                MovieOverview(movie)
            }
        }
    }
}
```

- 核心作用：框架通过 `key` 值匹配节点，避免因列表顺序 / 增删导致的无意义节点销毁重建，减少副作用（如网络请求）的重复执行。

#### 2. 稳定类型：减少无效重组

稳定类型是 Compose 编译器判断 “是否可跳过函数重组” 的核心依据 —— 若可组合函数的所有输入参数为稳定类型，且参数值未变化，框架会直接跳过该函数的执行，无需重新生成 / 更新节点，大幅提升重组效率。

##### （1）稳定类型的判定规则（需同时满足）

① 类型实例的 `equals` 结果永久稳定（相等则永远相等，不等则永远不等）；

② 类型的公共属性变化时，Compose 能感知（如 `MutableState` 的 `value` 变化会触发通知）；

③ 类型的所有公共属性也都是稳定类型。

##### （2）常见的稳定 / 非稳定类型

|      类型分类      |             示例              |  是否稳定  |
| :----------------: | :---------------------------: | :--------: |
| 基础类型 / 字符串  |     Int、Boolean、String      |     是     |
|  Compose 状态类型  |   MutableState<T>、State<T>   |     是     |
|     不可变集合     | List<T>、Map<T>（非 Mutable） |     是     |
|   Lambda 表达式    |          () -> Unit           | 是（注 1） |
|      可变集合      | MutableList<T>、MutableMap<T> |     否     |
| 未标注的普通可变类 | class User(var name: String)  |     否     |

注 1：Lambda 本身是稳定类型，但如果 Lambda 捕获了非稳定变量（如普通可变类的属性），会间接导致函数无法跳过重组。

##### （3）自定义稳定类型

若需传递多字段的业务数据，无需拆分为多个基本类型，可通过 `@Stable` 注解标注自定义类型，使其成为稳定类型：

```kotlin
// 自定义稳定类型：封装业务数据，简洁且支持重组优化
@Stable
data class Movie(
    val id: String, // 基础类型（稳定）
    val title: String, // 基础类型（稳定）
    val isFavorited: MutableState<Boolean> // Compose状态类型（稳定）
)

// 入参使用自定义稳定类型，参数值不变则跳过重组
@Composable fun MovieItem(movie: Movie) {
    Text("${movie.title} - 收藏状态：${movie.isFavorited.value}")
}
```

##### （4）实操建议

- 优先使用稳定类型作为可组合函数入参，避免直接使用 `MutableList`、未标注的普通可变类；
- 若需使用可变数据，建议用 `MutableState<T>` 托管（如 `MutableState<Int>` 替代直接用 `var Int`），让 Compose 感知变化并优化重组；
- 自定义业务模型时，用 `@Stable` 标注，兼顾代码简洁性和重组效率。

### 总结

1. Composition 是框架维护的节点树，由可组合函数首次执行生成，仅能通过重组更新；
2. 可组合函数的生命周期是其对应节点在 Composition 树中 “添加 - 更新 - 移除” 的过程，调用点是节点标识的核心依据；
3. `key` 优化列表节点的标识逻辑，稳定类型通过 “参数无变化则跳过重组” 减少无效执行，是 Compose 性能优化的两大核心手段。
