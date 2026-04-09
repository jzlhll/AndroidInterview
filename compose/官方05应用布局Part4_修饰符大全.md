修饰符大全
用到的时候，查官方文档。

https://developer.android.google.cn/develop/ui/compose/modifiers-list?hl=zh-cn#Animation


| 分类                       | 修饰符                             | 核心作用                           | 适用作用域              |
| -------------------------- | ---------------------------------- | ---------------------------------- | ----------------------- |
| **尺寸与约束**             | `size`                             | 设置首选宽高，遵守父约束           | 全局                    |
|                            | `sizeIn`                           | 自定义最小 / 最大宽高范围          | 全局                    |
|                            | `requiredSize`                     | 强制固定尺寸，无视父约束           | 全局                    |
|                            | `width` / `height`                 | 固定宽 / 高，另一方向自适应        | 全局                    |
|                            | `requiredWidth` / `requiredHeight` | 强制固定宽 / 高                    | 全局                    |
|                            | `fillMaxSize`                      | 撑满父级全部空间                   | 全局                    |
|                            | `fillMaxWidth` / `fillMaxHeight`   | 撑满父级宽度 / 高度                | 全局                    |
|                            | `wrapContentSize`                  | 按内容自适应大小，可指定对齐       | 全局                    |
|                            | `aspectRatio`                      | 强制固定宽高比                     | 全局                    |
|                            | `defaultMinSize`                   | 设置组件默认最小尺寸               | 全局                    |
| **对齐与布局权重**         | `align`                            | Box 内任意方位对齐                 | BoxScope                |
|                            | `align`                            | Row 内垂直对齐 / Column 内水平对齐 | RowScope / ColumnScope  |
|                            | `alignByBaseline`                  | 按文本基线对齐                     | RowScope / ColumnScope  |
|                            | `weight`                           | 按比例分配父布局剩余空间           | RowScope / ColumnScope  |
|                            | `matchParentSize`                  | 子项铺满父 Box，但不撑大父布局     | BoxScope                |
| **内边距与偏移**           | `padding`                          | 通用内边距                         | 全局                    |
|                            | `absolutePadding`                  | 绝对方向内边距（无视 RTL）         | 全局                    |
|                            | `paddingFromBaseline`              | 基于文本基线的内边距               | 全局                    |
|                            | `offset`                           | 相对偏移（自动适配 RTL）           | 全局                    |
|                            | `absoluteOffset`                   | 绝对坐标偏移（无视 RTL）           | 全局                    |
| **绘制与外观**             | `background`                       | 设置背景（颜色、渐变、形状）       | 全局                    |
|                            | `border`                           | 添加边框（宽度、颜色、形状）       | 全局                    |
|                            | `clip`                             | 按指定形状裁剪                     | 全局                    |
|                            | `clipToBounds`                     | 裁剪到组件边界                     | 全局                    |
|                            | `alpha`                            | 设置透明度                         | 全局                    |
|                            | `shadow`                           | 绘制阴影                           | 全局                    |
|                            | `dropShadow`                       | 自定义外阴影                       | 全局                    |
|                            | `innerShadow`                      | 自定义内阴影                       | 全局                    |
|                            | `zIndex`                           | 控制叠放层级                       | 全局                    |
|                            | `drawBehind`                       | 在内容后方自定义绘制               | 全局                    |
|                            | `drawWithContent`                  | 控制内容前后绘制顺序               | 全局                    |
|                            | `indication`                       | 自定义点击反馈效果（如水波纹）     | 全局                    |
| **触摸与交互**             | `clickable`                        | 点击事件（带默认水波纹）           | 全局                    |
|                            | `combinedClickable`                | 点击 + 长按 + 双击统一处理         | 全局                    |
|                            | `draggable`                        | 单方向拖动                         | 全局                    |
|                            | `draggable2D`                      | 二维自由拖动                       | 全局                    |
|                            | `anchoredDraggable`                | 带锚点的拖动（替代 swipeable）     | 全局                    |
|                            | `selectable`                       | 单选选中交互                       | 全局                    |
|                            | `selectableGroup`                  | 单选组语义标记                     | 全局                    |
|                            | `toggleable`                       | 布尔态开关切换                     | 全局                    |
|                            | `triStateToggleable`               | 三态切换（开 / 关 / 半选）         | 全局                    |
|                            | `hoverable`                        | 鼠标悬停状态监听                   | 全局                    |
|                            | `pointerInput`                     | 底层自定义手势 / 触摸事件          | 全局                    |
| **滚动相关**               | `verticalScroll`                   | 垂直滚动                           | 全局                    |
|                            | `horizontalScroll`                 | 水平滚动                           | 全局                    |
|                            | `scrollable`                       | 基础单方向滚动逻辑                 | 全局                    |
|                            | `scrollable2D`                     | 二维滚动                           | 全局                    |
|                            | `nestedScroll`                     | 嵌套滚动联动                       | 全局                    |
|                            | `rotaryScrollable`                 | 表盘 / 旋转控制器滚动              | 全局                    |
| **焦点控制**               | `focusable`                        | 使组件可获取焦点                   | 全局                    |
|                            | `focusTarget`                      | 基础焦点目标                       | 全局                    |
|                            | `focusRequester`                   | 主动请求焦点                       | 全局                    |
|                            | `focusProperties`                  | 配置焦点导航规则                   | 全局                    |
|                            | `onFocusChanged`                   | 焦点变化监听                       | 全局                    |
|                            | `focusGroup`                       | 焦点分组                           | 全局                    |
| **布局基础**               | `layout`                           | 完全自定义测量与布局逻辑           | 全局                    |
|                            | `onSizeChanged`                    | 监听组件尺寸变化                   | 全局                    |
|                            | `onGloballyPositioned`             | 获取全局位置与尺寸                 | 全局                    |
|                            | `layoutId`                         | 给子项标记 ID，供父布局识别        | 全局                    |
| **动画相关**               | `animateContentSize`               | 内容尺寸改变时自动过渡动画         | 全局                    |
|                            | `animateBounds`                    | 布局边界变化动画                   | 全局                    |
|                            | `animateEnterExit`                 | 进入 / 退出动画                    | AnimatedVisibilityScope |
|                            | `animateItem`                      | 列表项增删 / 排序动画              | LazyItemScope           |
| **语义与测试**             | `semantics`                        | 无障碍语义配置                     | 全局                    |
|                            | `clearAndSetSemantics`             | 清空并重设语义                     | 全局                    |
|                            | `progressSemantics`                | 进度类组件语义                     | 全局                    |
|                            | `testTag`                          | UI 测试标记                        | 全局                    |
| **窗口 Insets / 系统边距** | `systemBarsPadding`                | 适配系统栏内边距                   | 全局                    |
|                            | `statusBarsPadding`                | 适配状态栏内边距                   | 全局                    |
|                            | `navigationBarsPadding`            | 适配导航栏内边距                   | 全局                    |
|                            | `imePadding`                       | 适配软键盘弹出内边距               | 全局                    |
|                            | `safeContentPadding`               | 安全区域内边距                     | 全局                    |
|                            | `windowInsetsPadding`              | 自定义 WindowInsets 边距           | 全局                    |
| **图形变换**               | `rotate`                           | 旋转                               | 全局                    |
|                            | `scale`                            | 缩放                               | 全局                    |
|                            | `transformable`                    | 支持缩放手势 + 平移 + 旋转         | 全局                    |
|                            | `graphicsLayer`                    | 高性能图层变换（合成效率更高）     | 全局                    |
|                            | `blur`                             | 模糊效果                           | 全局                    |
| **其他实用**               | `minimumInteractiveComponentSize`  | 保证最小可触摸尺寸（48dp）         | 全局                    |
|                            | `keepScreenOn`                     | 保持屏幕常亮                       | 全局                    |
|                            | `systemGestureExclusion`           | 排除系统手势区域                   | 全局                    |
|                            | `sensitiveContent`                 | 敏感内容遮挡（防录屏）             | 全局                    |
|                            | `bringIntoViewRequester`           | 请求滚动至可见区域                 | 全局                    |
|                            | `pullToRefresh`                    | 下拉刷新                           | 全局                    |