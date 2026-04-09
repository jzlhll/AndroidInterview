## 创建自定义修饰符

本章节算是进阶章节。我们可以后面再来学习。https://developer.android.google.cn/develop/ui/compose/custom-modifiers

1. **官方定位**：文档将其归为 “扩展 Compose 功能” 的高级技巧，而非入门必备
2. **使用场景**：仅在**内置修饰符无法满足需求**时才需要（如复用复杂样式、自定义测量 / 绘制、特殊交互）
3. **技术复杂度**：涉及`Modifier.Node`底层 API、布局测量原理、状态管理等进阶知识

## 二、常规开发的最佳实践

**90% 以上的 UI 场景**，用这些内置修饰符组合就能搞定：

- 尺寸控制：`size`/`fillMaxSize`/`wrapContentSize`/`sizeIn`
- 布局定位：`padding`/`offset`/`align`/`weight`
- 外观样式：`background`/`clip`/`border`
- 交互能力：`clickable`/`scrollable`/`draggable`
- 约束修改：`fillMaxWidth`/`wrapContentSize`等

## 三、自定义修饰符的适用场景（什么时候需要学）

只有遇到以下情况，才需要考虑自定义修饰符：

1. **样式复用**：多个组件需要相同的修饰符组合（如统一的卡片样式）
2. **特殊行为**：需要自定义测量逻辑、绘制效果或交互反馈
3. **性能优化**：复杂场景下用`Modifier.Node`替代`composed`提升性能
4. **团队规范**：封装通用组件库，统一 UI 风格与交互行为

## 四、自定义修饰符的实现难度梯度（从易到难）

| 实现方式                        | 难度   | 适用场景                                                     |
| ------------------------------- | ------ | ------------------------------------------------------------ |
| 链式组合现有修饰符              | 入门级 | 简单样式复用<br />（如`cardModifier = Modifier.padding(16.dp).clip(RoundedCornerShape(8.dp)).background(Color.White)`） |
| `composed`工厂方法 （不再推荐） | 中级   | 需要访问 Compose 上下文（如读取主题、使用状态）              |
| `Modifier.Node`底层 API         | 高级   | 复杂布局 / 绘制、性能敏感场景（官方推荐）Android Developers  |

## 五、建议

**先熟练掌握内置修饰符，等遇到 “重复写相同修饰符” 或 “内置修饰符无法实现” 的问题时，再回头学习该章节**。