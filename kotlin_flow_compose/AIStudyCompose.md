# Jetpack Compose 学习路线（适配 View/XML 开发者）

### 阶段 1：核心思想与环境准备（打基础）

1. Compose 核心理念：声明式 UI vs 命令式 UI，理解 “状态驱动 UI”
2. 环境配置：Compose 依赖、Preview 预览功能
3. 基础规范：@Composable 注解、可组合函数编写规则

### 阶段 2：基础 UI 组件（对应 XML 的基础控件）

1. 核心基础组件：Text、Button、Image
2. 组件基础参数：核心配置项
3. 简单状态交互：Button 点击修改 Text 内容

### 阶段 3：布局与 Modifier（核心！对应 XML 的布局 + 属性）

1. Modifier 核心：链式调用逻辑
2. 基础布局：Column、Row、Box
3. 滚动列表：LazyColumn、LazyRow（对比 RecyclerView，先掌握基础懒加载逻辑）

### 阶段 4：主题与样式（对应 XML 的 styles/themes）

1. MaterialTheme：主题总控
2. Typography：文字样式体系
3. 样式复用：自定义可复用样式
4. 颜色体系：Color、ColorScheme

### 阶段 5：页面骨架（对应 XML 的页面整体布局）

1. Scaffold：页面整体骨架（对应 XML 中 Activity/Fragment 的根布局）

2. TopAppBar：顶部导航栏（对应 XML 的 Toolbar）

3. BottomBar/BottomNavigation：底部导航栏（对应 XML 的 BottomNavigationView）

4. 折叠式标题栏（对应 XML 的 AppBarLayout + CollapsingToolbarLayout）：

   - 核心方案：`CollapsingToolbarScaffold`，结合 `TopAppBar` + `rememberCollapsingToolbarScaffoldState()` 实现折叠、缩放、滚动联动效果
   - 核心逻辑：通过 `ScrollBehavior` 监听滚动状态，驱动标题栏的折叠 / 展开动画，替代 XML 中 AppBarLayout 的滚动协调逻辑

   

5. 协调布局替代（对应 XML 的 CoordinatorLayout）：

   - Compose 无直接一对一替代组件（CoordinatorLayout 核心是 “布局间协调交互”）
   - 替代方案：通过 `Modifier` 交互修饰符（如 `nestedScroll`） + 状态驱动 + 布局组合实现协调效果（比如滚动联动、嵌套滑动、滑动隐藏 / 显示导航栏）
   - 关键：掌握 `nestedScroll` 修饰符、`ScrollState` 状态管理，替代 CoordinatorLayout 的 “父子布局交互” 核心能力

   

### 阶段 6：交互与弹窗（对应 XML 的事件 + Dialog/BottomSheet）

1. 基础交互：点击、长按、拖拽等事件处理

2. 弹窗体系（对应 XML 的各类 Dialog）：

   - 居中 Dialog 替代（对应 XML 的自定义居中 Dialog/AlertDialog）：

     - 基础版：`AlertDialog`，默认居中展示，可自定义标题、内容、按钮，替代 XML 的 AlertDialog
     - 自定义居中 Dialog：基于 `Dialog` 组件 + `Modifier.align(Alignment.Center)` 实现完全自定义的居中弹窗，替代 XML 中自定义布局的居中 Dialog
     - 关键：通过 `DialogProperties` 控制弹窗属性（如是否可取消、背景透明度），结合 Modifier 调整大小、位置、样式

     

   - BottomSheetDialog 替代（对应 XML 的 BottomSheetDialog）：

     - 核心方案：`ModalBottomSheet`（弹窗式底部 sheet）
     - 扩展方案：`BottomSheetScaffold`（底部 sheet 作为页面骨架一部分，而非弹窗，对应 XML 中 BottomSheetBehavior + CoordinatorLayout）
     - 核心：通过 `rememberModalBottomSheetState()` 管理底部 sheet 的展开 / 收起状态，替代 XML 的 BottomSheetBehavior

     

   

3. 扩展弹窗：ModalBottomSheet（标准化底部弹窗）

### 阶段 7：状态管理（Compose 的核心灵魂）

1. 基础状态：State、MutableState、remember
2. 状态提升：Compose 最佳实践
3. 状态保存：rememberSaveable

### 阶段 8：列表高级交互（核心：PullToRefreshBox）

这是 LazyColumn/LazyRow 的核心进阶能力，对应你熟悉的 SwipeRefreshLayout+RecyclerView 触底：

1. 下拉刷新体系：

   

   - 核心方案：

     ```
     PullToRefreshBox
     ```

     - 核心：支持灵活的样式定制（刷新指示器、下拉距离、动画等）
     - 关键：掌握其参数（refreshing、onRefresh、indicator、content 等）

     

   

2. 触底加载：LazyColumn/LazyRow 的 onScrollEvent 监听

   

   - 核心：检测列表滚动位置，判断是否滑到最后一项触发加载
   - 关键：处理加载中状态、避免重复请求

   

3. 组合使用：PullToRefreshBox + LazyColumn + 触底加载（推荐的列表交互组合）

   

### 阶段 9：进阶整合与实战（落地应用）

1. Compose+ViewModel：结合 PullToRefreshBox 处理异步刷新数据
2. LazyColumn 高级用法：多类型 item、分页加载 + PullToRefreshBox 刷新
3. 自定义 PullToRefreshBox：定制刷新指示器样式（比如替换默认动画）
4. 折叠标题栏实战：CollapsingToolbarScaffold + LazyColumn 实现滚动折叠效果（替代 AppBarLayout+CollapsingToolbarLayout+RecyclerView）
5. 弹窗 + 底部 sheet 实战：“居中 Dialog 提示 + ModalBottomSheet 选择器” 组合使用
6. 小型实战：“PullToRefreshBox 下拉刷新 + LazyColumn 触底加载 + 居中弹窗提示” 的完整列表页
7. 综合实战：CollapsingToolbarScaffold + LazyColumn + ModalBottomSheet + 居中 Dialog 的完整页面（覆盖 XML 中 CoordinatorLayout 组合场景）

------

### 总结

1. XML 中的 AppBarLayout/CollapsingToolbarLayout 对应 Compose 的 `CollapsingToolbarScaffold`，CoordinatorLayout 则通过 `Modifier.nestedScroll` + 状态驱动实现协调交互；
2. XML 中的 BottomSheetDialog 用 Compose 的 `ModalBottomSheet` 替代，居中 Dialog 用 `AlertDialog`（基础）或 `Dialog`（自定义）实现；
3. Compose 无 “一对一复制” XML 组件的设计，核心是通过 “组合函数 + Modifier + 状态” 替代 XML 布局的 “嵌套 + 行为（Behavior）” 逻辑。