## Compose 修饰符

借助修饰符，您可以修饰或扩充可组合项。您可以使用修饰符来执行以下操作：

- 更改可组合项的大小、布局、行为和外观
- 添加信息，如无障碍标签
- 处理用户输入
- 添加高级互动，如使元素可点击、可滚动、可拖动或可缩放。



### 基本示例

修复符就是kotlin对象，提供了很多类函数来创建修饰符。

```kotlin
@Composable
private fun Greeting(...) {
    Column(
        modifier = Modifier
            .padding(24.dp)
            .fillMaxWidth()
    ) {
        Text(text = "Hello,")
        Text(text = name)
    }
}
```

<img src="../pictures/composeModified1.png" alt="modified01" width="420" align="left"/>

在上面的代码中，结合使用了不同的修饰符函数:

- `padding` 在元素周围留出空间。
- `fillMaxWidth` 使可组合项填充其父项为它提供的最大宽度。

最佳实践是让所有可组合项接受 `modifier` 参数，并将该修饰符传递给其发出界面元素的第一个子项。

> 最佳实践：自定义 Compose 组件 → 必须像 View 体系一样，有「唯一的根父控件」
>
> 1. 不能直接暴露**多个并列、无包裹的元素**（比如直接写两个并列 Row，外面不包布局）；
> 2. 外部传入的 `modifier`，**只绑定这个唯一的根父控件**；
> 3. 这个 `modifier` 负责控制**整个组件的基础约束**（大小、边距、位置、点击、外层布局行为）。
>
> - **View**：自定义控件 → 必须继承 `LinearLayout/RelativeLayout`（唯一根父控件），外部设置 `layout_width/margin/padding` → 作用于**整个根父控件**；
> - **Compose**：自定义可组合项 → 必须用 `Column/Row/Box` 包成**唯一根布局**，外部传 `modifier` → 作用于**整个根布局**。



### 顺序很重要

```kotlin
Modifier.padding(20dp).clickable {}
```

执行逻辑：

1. 先给组件**加 20dp 内边距**，把组件整体撑大；
2. 再给，撑大后的整个区域加点击。

✅ 结果：内边距的空白区域也能点击。

```kotlin
Modifier.clickable {}.padding(20dp)
```

1. 先给**组件原始大小**加点击；
2. 再在点击区域外面加 20dp 空白。

✅ 结果：只有原始组件能点击，空白内边距点了没反应。

**一句话总结**：**修饰符越靠前，作用范围越大**。

#### 没有margin只有padding

View 体系：**分两个属性**

- `padding`：内边距（元素内部空白，点击有效）
- `margin`：外边距（元素外部空白，点击无效）

Compose 体系：**只有一个 `padding`**

没有专门的`margin`修饰符！**用 `padding` + 修饰符顺序，直接模拟出 margin 和 padding 两种效果**：

1. 想做 **View 的 margin（外边距）** → `padding` 放最前面
2. 想做 **View 的 padding（内边距）** → `padding` 放后面



### 常用内置修饰符

Compose 自带的布局（`Column`/`Row`/`Box`），**默认行为是：父布局包裹子元素的大小**。子元素多大，父布局就多大；

完全等价于 ViewGroup 的 `wrap_content`！

- View：`layout_width="wrap_content"` → 包裹子元素
- Compose：布局不加修饰符 → 默认包裹子元素



#### 尺寸

- `size`

  使用`size()`来设置尺寸：

  ```kotlin
  @Composable
  fun ArtistCard(/*...*/) {
      Row(
          modifier = Modifier.size(width = 400.dp, height = 100.dp)
      ) {
          Image(/*...*/)
          Column { /*...*/ }
      }
  }
  ```

  如果指定的尺寸不符合来自布局父项的约束条件，则可能不会采用该尺寸。

- `requiredSize`

  如果您希望可组合项的尺寸固定不变，而不考虑传入的约束条件，请使用 `requiredSize` 修饰符：

  在此示例中，即使父项的 `height` 设置为 `100.dp`，`Image` 的高度还是 `150.dp`，因为 `requiredSize` 修饰符优先级较高。将 `requiredSize` 等修饰符直接传递给子项，替换子项从父项接收的约束条件。父项会将子项的 `width` 和 `height` 值视为已根据父项提供的约束条件强制转换过。然后，布局系统会假定子项遵守这些约束条件，将子项居中放置在父项分配的空间中。

  ```kotlin
  @Composable
  fun ArtistCard(/*...*/) {
      Row(
          modifier = Modifier.size(width = 400.dp, height = 100.dp)
      ) {
          Image(
              /*...*/
              modifier = Modifier.requiredSize(150.dp) //高优先级
          )
          Column { /*...*/ }
      }
  }
  ```

- `wrapContentSize`

  通过对子项应用 `wrapContentSize` 修饰符来取消默认的居中行为。

  ```kotlin
  Modifier.wrapContentSize(
      align: Alignment = Alignment.TopStart, // 默认：左上，改这个参数，就能实现任意对齐方式。
      unbounded: Boolean = false 
  )
  ```

  Alignment.TopStart左上（默认）

  Alignment.TopEnd右上

  Alignment.BottomStart左下

  Alignment.BottomEnd右下

  Alignment.BottomCenter底部居中

  Alignment.CenterStart左居中



#### 间距

- `padding`

​	如需在整个元素周围全添加内边距使用`padding`。

- `paddingFromBaseLine`

  如需在文本基线上方添加内边距，以实现从布局顶部到基线保持特定距离，请使用 `paddingFromBaseline` 修饰符。

  ```kotlin
  @Composable
  fun ArtistCard(artist: Artist) {
      Row(/*...*/) {
          Column {
              Text(
                  text = artist.name,
                  modifier = Modifier.paddingFromBaseline(top = 50.dp)
              )
              Text(artist.lastSeenOnline)
          }
      }
  }
  ```

  <img src="../pictures/composeModified2.png" alt="composeLayoutBasic1" width="400" align="left"/>



- `offset`

  如需相对于原始位置放置布局，请添加 `offset`修饰符，并设置在 **x** 轴和 **y** 轴的偏移量。偏移量可以是正数，也可以是非正数：

  ```kotlin
  @Composable
  fun ArtistCard(artist: Artist) {
      Row(/*...*/) {
          Column {
              Text(artist.name)
              Text(
                  text = artist.lastSeenOnline,
                  modifier = Modifier.offset(x = 4.dp)
              )
          }
      }
  }
  ```

  `padding` 和 `offset` 之间的区别在于，向可组合项添加 `offset` 不会改变其测量结果：

  

   `padding内边距`：给元素加空白 → 元素的**测量宽高变大** → 周围的布局会给它留足空间 → 不会重叠。**会改变元素占用的空间大小**（影响布局）。类似View 的 `setPadding()`（改变 View 大小）

   `offset偏移`：把元素**平移**→ 元素的**测量宽高完全不变** → 周围布局不知道它挪了位置 → 可能和其他元素重叠。**只挪位置，不改变空间大小**（不影响布局），类似View 的 `translationX/translationY`（只移动，不改变 View 大小）

  **和 View 体系对比**：

  

  有两种重载函数:

​	偏移量参数和采用lambda的offset。https://developer.android.google.cn/develop/ui/compose/performance?hl=zh-cn#defer-reads性能章节会讲述。

 

- `absoluteOffset`

  如果您需要设置偏移量，而不考虑布局方向，该修饰符中的正偏移值一律会将元素向右移。



#### 填充满

- `fillMaxSize` & `fillMaxWidth` & `fillMaxHeight`

  子布局填充父项允许的所有可用高度、宽度，高宽。
