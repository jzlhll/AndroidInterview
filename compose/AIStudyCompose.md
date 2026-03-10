# Jetpack Compose 学习路线（适配 View/XML 开发者）

## 阶段 1：核心思想与环境准备（打基础）

1. Compose 核心理念：声明式 UI vs 命令式 UI，理解 “状态驱动 UI” 核心逻辑
2. 环境配置：Compose 依赖引入、Preview 预览功能使用、Compose Compiler 版本适配
3. 基础规范：@Composable 注解特性、可组合函数编写规则（无返回值、单一职责、纯函数优先）

## 阶段 2：基础 UI 组件（对应 XML 的基础控件）

1. 核心基础组件：Text（对应 TextView）、Button（对应 Button/TextView 点击）、Image（对应 ImageView）
2. 组件基础参数：Modifier 基础属性（大小、内边距、背景）、内容配置（文字颜色、图片缩放模式）
3. 简单状态交互：Button 点击修改 Text 内容（remember + MutableState 基础使用）

## 阶段 3：布局与 Modifier（核心！对应 XML 的布局 + 属性）

1. Modifier 核心：链式调用逻辑、修饰符执行顺序、自定义 Modifier
2. 基础布局：Column（对应 LinearLayout 垂直）、Row（对应 LinearLayout 水平）、Box（对应 FrameLayout）
3. 滚动列表：LazyColumn（对应 RecyclerView 垂直）、LazyRow（对应 RecyclerView 水平），掌握基础懒加载逻辑

## 阶段 4：主题与样式（对应 XML 的 styles/themes）

1. MaterialTheme：主题总控（颜色、文字、形状体系）
2. Typography：文字样式体系（字体、大小、行高，对应 XML 的 textStyle）
3. 样式复用：自定义可复用样式函数、Modifier 样式封装
4. 颜色体系：Color、ColorScheme（对应 XML 的 color 资源 + theme 色值）

## 阶段 5：页面骨架（对应 XML 的页面整体布局）

1. Scaffold：页面整体骨架（对应 XML 中 Activity/Fragment 的根布局）

2. TopAppBar：顶部导航栏（对应 XML 的 Toolbar）

3. BottomBar/BottomNavigation：底部导航栏（对应 XML 的 BottomNavigationView）

4. 折叠式标题栏（对应 XML 的 AppBarLayout + CollapsingToolbarLayout）：

   - 核心方案：`CollapsingToolbarScaffold` + `rememberCollapsingToolbarScaffoldState()` 实现折叠 / 缩放 / 滚动联动
   - 核心逻辑：`ScrollBehavior` 监听滚动状态，驱动标题栏动画

   

5. 协调布局替代（对应 XML 的 CoordinatorLayout）：

   - Compose 无直接替代组件，核心通过 `Modifier.nestedScroll` + 状态驱动 + 布局组合实现协调交互
   - 关键：掌握 `nestedScroll` 修饰符、`ScrollState` 状态管理，实现滚动联动、嵌套滑动等效果

   

## 阶段 6：交互与弹窗（对应 XML 的事件 + Dialog/BottomSheet）

1. 基础交互：点击、长按、拖拽等事件处理（Modifier.clickable、pointerInput 等）

2. 弹窗体系：

   - 居中 Dialog 替代：`AlertDialog`（基础版）、`Dialog` + Modifier（自定义版），通过 `DialogProperties` 控制弹窗属性
   - BottomSheetDialog 替代：`ModalBottomSheet`（弹窗式）、`BottomSheetScaffold`（页面骨架式），通过 `rememberModalBottomSheetState()` 管理状态

   

## 阶段 7：状态管理（Compose 的核心灵魂）

1. 基础状态：State、MutableState、remember（内存级状态保存）
2. 状态提升：Compose 最佳实践（单向数据流）
3. 状态保存：rememberSaveable（进程销毁后状态恢复，对应 XML 的 onSaveInstanceState）
4. 进阶状态：ViewModel + StateFlow/SharedFlow（跨组件状态共享，对应 XML 的 ViewModel + LiveData）

## 阶段 8：列表高级交互（对应 SwipeRefreshLayout+RecyclerView）

1. 下拉刷新：`PullToRefreshBox` 核心使用（refreshing、onRefresh、自定义指示器）
2. 触底加载：LazyColumn/LazyRow 的 onScrollEvent 监听，处理加载状态避免重复请求
3. 组合使用：PullToRefreshBox + LazyColumn + 触底加载（完整列表交互方案）
4. 列表进阶：多类型 Item、列表项动画、列表状态恢复

## 阶段 9：副作用（Side Effect）（Compose 生命周期与异步管理）

> 补充：原有文档缺失的核心模块，解决 “Compose 中执行异步操作、生命周期感知逻辑” 问题（对应 XML 的 Activity/Fragment 生命周期回调 + Handler/RxJava）

1. 核心副作用 API：

   - `LaunchedEffect`：组合函数内启动协程，随组合项进入 / 退出组合执行 / 取消
   - `rememberCoroutineScope`：获取组合作用域的协程作用域，用于非组合函数中启动协程
   - `DisposableEffect`：随组合项销毁执行清理逻辑（如注销监听、关闭流）
   - `SideEffect`：每次重组都会执行（慎用，用于同步 Compose 状态到外部）
   - `produceState`：将非 Compose 状态（如 Flow、Callback）转换为 Compose State

   

2. 副作用最佳实践：

   - 避免在可组合函数顶层执行副作用
   - 合理使用 key 参数控制副作用执行时机
   - 结合 ViewModel 处理异步数据请求（副作用仅负责触发，不存储数据）

   

## 阶段 10：导航（Navigation）（对应 XML 的 Navigation Component）

1. 基础导航：NavHost + NavController + composable 路由配置（对应 XML 的 navigation 标签）
2. 路由参数：参数传递（基础类型、Parcelable 类型）、参数解析、深层链接（DeepLink）
3. 嵌套导航：子 NavHost 实现页面嵌套导航（对应 XML 的 nested graph）
4. 导航状态：返回栈管理、导航动画自定义、底部导航与导航结合

## 阶段 11：动画（Animation）（对应 XML 的属性动画 / 补间动画）

1. 基础动画：`animate*AsState`（单个属性动画，如尺寸、颜色、透明度）

2. 高级动画：

   - `Animatable`：手动控制动画进度（对应 ValueAnimator）
   - `Transition`：多状态切换动画（如页面切换、组件显隐）
   - `AnimatedVisibility`：组件显隐动画（对应 XML 的 View 显隐动画）

   

3. 手势联动动画：拖拽 + 动画（如侧滑删除、下拉刷新动画）

## 阶段 12：自定义 Composable 组件（对应 XML 的自定义 View/ViewGroup）

1. 自定义布局：

   - 测量逻辑：Layout 修饰符实现自定义测量（对应 onMeasure）
   - 布局逻辑：Layout 函数实现子组件排列（对应 onLayout）

   

2. 自定义绘制：

   - Canvas 组件（对应 onDraw）：绘制图形、文字、路径
   - DrawModifier：封装绘制逻辑，实现复用

   

3. 自定义交互：触摸事件处理（pointerInput）、手势识别（如双击、缩放）

## 阶段 13：性能优化（Compose 渲染优化）

1. 重组优化：

   - 减少不必要重组：remember 正确使用、稳定类型（Stable）、不可变数据
   - 重组范围控制：`remember` 缓存计算结果、`derivedStateOf` 减少状态更新频率

   

2. 列表优化：LazyColumn 项复用、key 正确设置、避免项内频繁重组

3. 渲染优化：`skipToDestination` 跳过中间重组、减少 Modifier 频繁创建

4. 内存优化：及时取消协程 / 监听（DisposableEffect）、避免内存泄漏

## 阶段 14：测试（Compose UI 测试）

1. 基础测试环境：ComposeTestRule 配置、依赖引入
2. UI 交互测试：模拟点击、输入、滑动，断言组件状态 / 显示内容
3. 状态测试：验证状态更新是否正确驱动 UI 变化
4. 副作用测试：验证 LaunchedEffect、produceState 等副作用逻辑

## 阶段 15：进阶整合与实战（落地应用）

1. 基础组合实战：ViewModel + PullToRefreshBox + LazyColumn 实现列表刷新 / 加载

2. 页面完整实战：CollapsingToolbarScaffold + LazyColumn + ModalBottomSheet + Dialog 实现复杂页面

3. 项目级实战：

   - 导航 + 状态管理 + 副作用 实现多页面应用
   - 自定义 Composable + 动画 实现业务专属组件
   - 性能优化 + 测试 保障应用稳定性

   

## 总结

1. XML 布局体系 vs Compose 核心逻辑：

   - AppBarLayout/CollapsingToolbarLayout → `CollapsingToolbarScaffold`
   - CoordinatorLayout → `Modifier.nestedScroll` + 状态驱动
   - BottomSheetDialog → `ModalBottomSheet` / `BottomSheetScaffold`
   - Dialog/AlertDialog → `AlertDialog` / 自定义 `Dialog`
   - SwipeRefreshLayout+RecyclerView → `PullToRefreshBox` + `LazyColumn`
   - 自定义 View/ViewGroup → 自定义 Composable + Layout/Canvas 修饰符
   - 生命周期 / 异步操作 → Compose 副作用 API（LaunchedEffect/DisposableEffect 等）
   - Navigation Component → Compose Navigation（NavHost/NavController）

   

2. Compose 核心思维：无 “一对一复制” XML 组件，核心是 “组合函数 + Modifier + 状态 + 副作用” 替代 XML 布局的 “嵌套 + 行为（Behavior）” 逻辑。

3. 学习优先级：先掌握状态管理 + 布局 Modifier，再补充副作用 + 导航，最后深入动画 + 自定义组件 + 性能优化。