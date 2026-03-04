## 细化版：阶段 1 + 阶段 2（附 Demo 代码 & 核心指导）

### 阶段 1：核心思想与环境准备（打基础）

#### 1. Compose 核心理念：声明式 UI vs 命令式 UI，理解 “状态驱动 UI”

**核心解释（适配 XML 开发者）**：

- **命令式 UI（XML/View 方式）**：你需要 “手动” 操控 UI 元素的状态和显示。比如想修改 TextView 的文字，要先通过 `findViewById` 找到控件，再调用 `setText()` 方法；想隐藏按钮，要调用 `setVisibility()`。每一步 UI 变化都需要你主动下发指令。
- **声明式 UI（Compose 方式）**：你只需要 “描述” UI 应该是什么样子（基于当前状态），Compose 会自动处理 UI 的更新。比如文字显示由一个变量（状态）决定，只要这个变量变了，UI 会自动刷新，无需你手动调用任何方法。
- **状态驱动 UI**：UI 是状态的 “投影”，状态是因，UI 是果。修改状态 = 修改 UI。

**对比示例（修改文本内容）**：

```kotlin
// 【命令式（XML/View）】伪代码
val textView = findViewById<TextView>(R.id.tv_demo)
// 手动修改 UI
textView.text = "新的文字内容" 

// 【声明式（Compose）】核心逻辑
var text by remember { mutableStateOf("初始文字") }
// 只需修改状态，UI 自动更新
text = "新的文字内容" 
```

#### 2. 环境配置：Compose 依赖、Preview 预览功能

**前置条件**：

- Android Studio 版本 ≥ Arctic Fox (2020.3.1)
- 项目 Gradle 插件版本 ≥ 7.0.0

**核心配置步骤**：

1. 项目根目录 `build.gradle`（或 `build.gradle.kts`）确保依赖仓库：

```gradle
repositories {
    google()
    mavenCentral()
}
```

1. 模块级 `build.gradle`（或 `build.gradle.kts`）配置 Compose 核心依赖：

```gradle
android {
    compileSdk 34 // 建议使用 33+ 版本

    defaultConfig {
        minSdk 21 // Compose 最低支持 minSdk 21
    }

    buildFeatures {
        compose true // 开启 Compose 功能
    }

    composeOptions {
        kotlinCompilerExtensionVersion "1.5.3" // 需与 Kotlin 版本匹配
        // 版本匹配规则：https://developer.android.com/jetpack/compose/releases#kotlin-compatibility
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }
}

dependencies {
    // Compose 核心依赖
    implementation "androidx.compose.ui:ui:1.6.0"
    implementation "androidx.compose.material:material:1.6.0"
    implementation "androidx.compose.ui:ui-tooling-preview:1.6.0"
    debugImplementation "androidx.compose.ui:ui-tooling:1.6.0"
    debugImplementation "androidx.compose.ui:ui-test-manifest:1.6.0"
}
```

**Preview 预览功能使用 Demo**：



```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview

// 可组合函数
@Composable
fun SimpleText() {
    Column {
        Text(text = "Hello Compose")
    }
}

// Preview 注解：无需运行 App，直接在 IDE 中预览 UI
@Preview(showBackground = true, name = "SimpleText 预览", fontSize = 16)
@Composable
fun SimpleTextPreview() {
    SimpleText() // 调用要预览的可组合函数
}
```

**关键说明**：

- `@Preview` 仅在 debug 模式生效，可通过 `showBackground` 设置预览背景、`name` 命名预览、`device` 指定预览设备等；
- 预览界面支持 “实时刷新”，修改代码后无需重新编译即可看到效果。

#### 3. 基础规范：@Composable 注解、可组合函数编写规则

**核心规则（适配 XML 开发者）**：

可组合函数是 Compose 构建 UI 的基本单元，替代 XML 中的布局文件 / 控件标签，核心规则：

1. 必须添加 `@Composable` 注解；
2. 函数名**首字母大写**（遵循组件化命名习惯，如 `SimpleText` 而非 `simpleText`）；
3. 无返回值（返回类型为 `Unit`，无需写）；
4. 可接收参数（替代 XML 中的控件属性）；
5. 不能有副作用（如不能直接在函数内修改外部变量，状态修改需通过 `State`/`MutableState`）。

**规范示例**：

```kotlin
import androidx.compose.foundation.layout.padding
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

// 符合规范的可组合函数：带注解、首字母大写、接收参数、无返回值
@Composable
fun CustomButton(
    text: String, // 按钮文字（替代 XML 中 Button 的 android:text 属性）
    modifier: Modifier = Modifier, // 布局修饰符（必传默认值，提升复用性）
    onClick: () -> Unit // 点击事件（替代 XML 中 Button 的 android:onClick 属性）
) {
    Button(
        onClick = onClick,
        modifier = modifier.padding(16.dp) // 链式调用修饰符（替代 XML 中 android:padding 属性）
    ) {
        Text(text = text)
    }
}

// 预览函数
@Preview(showBackground = true)
@Composable
fun CustomButtonPreview() {
    CustomButton(
        text = "点击我",
        onClick = {} // 预览时无需实现具体逻辑
    )
}
```

------

### 阶段 2：基础 UI 组件（对应 XML 的基础控件）

#### 1. 核心基础组件：Text、Button、Image

**核心说明**：这三个组件对应 XML 中的 `TextView`、`Button`、`ImageView`，是 Compose 最基础的 UI 单元，以下是每个组件的核心 Demo 及与 XML 的对比：

##### （1）Text 组件（对应 TextView）



```Kotlin
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.sp

@Composable
fun CustomText() {
    Text(
        text = "Hello Compose 基础文本", // 对应 XML：android:text
        fontSize = 18.sp, // 对应 XML：android:textSize
        color = Color.Blue, // 对应 XML：android:textColor
        fontWeight = FontWeight.Bold, // 对应 XML：android:textStyle="bold"
        textAlign = TextAlign.Center, // 对应 XML：android:gravity="center"
        maxLines = 1 // 对应 XML：android:maxLines
    )
}

@Preview
@Composable
fun CustomTextPreview() {
    CustomText()
}
```

##### （2）Button 组件（对应 Button）



```kotlin
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.tooling.preview.Preview

@Composable
fun CustomInteractiveButton() {
    // 定义状态：记录点击次数
    val clickCount = remember { mutableStateOf(0) }

    Button(
        onClick = { 
            // 点击时修改状态
            clickCount.value++ 
        }
    ) {
        Text(text = "已点击 ${clickCount.value} 次")
    }
}

@Preview
@Composable
fun CustomInteractiveButtonPreview() {
    CustomInteractiveButton()
}
```

##### （3）Image 组件（对应 ImageView）



```kotlin
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.size
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.painter.Painter
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
// 替换为你的项目资源 ID
import com.example.mycompose.R

@Composable
fun CustomImage() {
    // 加载本地资源图片（对应 XML：android:src="@drawable/ic_launcher"）
    val icon: Painter = painterResource(id = R.drawable.ic_launcher)
    
    Image(
        painter = icon,
        contentDescription = "应用图标", // 无障碍描述（必传，提升兼容性）
        modifier = Modifier.size(80.dp) // 对应 XML：android:width/android:height
    )
}

@Preview
@Composable
fun CustomImagePreview() {
    CustomImage()
}
```

#### 2. 组件基础参数：核心配置项

**核心说明**：Compose 组件的参数替代了 XML 中的控件属性，以下是基础组件的通用核心参数（以表格对比更清晰）：

|     Compose 参数     |       作用        |                对应 XML 属性                 |
| :------------------: | :---------------: | :------------------------------------------: |
|      `modifier`      | 布局 / 样式修饰符 | 所有布局相关属性（padding、size、margin 等） |
|   `color`（Text）    |     文字颜色      |             `android:textColor`              |
|      `fontSize`      |     文字大小      |              `android:textSize`              |
| `onClick`（Button）  |     点击事件      |              `android:onClick`               |
|  `painter`（Image）  |     图片资源      |                `android:src`                 |
| `contentDescription` |    无障碍描述     |      -（XML 无对应，Compose 强制要求）       |

**参数组合使用 Demo**：



```kotlin
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.*
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.mycompose.R

@Composable
fun CombinedComponentDemo() {
    // 垂直布局（对应 XML 的 LinearLayout(vertical)）
    Column(
        modifier = Modifier
            .padding(20.dp) // 外层内边距
            .fillMaxWidth(), // 占满宽度（对应 XML：android:layout_width="match_parent"）
        horizontalAlignment = Alignment.CenterHorizontally, // 子组件水平居中
        verticalArrangement = Arrangement.spacedBy(16.dp) // 子组件垂直间距
    ) {
        // 文本
        Text(
            text = "组合参数示例",
            fontSize = 20.sp,
            color = Color.Black,
            modifier = Modifier.padding(bottom = 8.dp)
        )
        
        // 图片
        Image(
            painter = painterResource(id = R.drawable.ic_launcher),
            contentDescription = "示例图片",
            modifier = Modifier.size(60.dp)
        )
        
        // 按钮
        Button(
            onClick = {},
            modifier = Modifier
                .width(200.dp) // 固定宽度
                .height(48.dp) // 固定高度
        ) {
            Text(text = "确认", fontSize = 16.sp)
        }
    }
}

@Preview(showBackground = true)
@Composable
fun CombinedComponentDemoPreview() {
    CombinedComponentDemo()
}
```

#### 3. 简单状态交互：Button 点击修改 Text 内容

**核心说明**：这是 Compose 最基础的状态交互，替代 XML 中 “点击按钮 → 找到 TextView → 修改文字” 的逻辑，核心是 `remember` + `mutableStateOf` 管理状态。

**完整 Demo（可直接运行）**：



```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.padding
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp

// 核心交互组件
@Composable
fun ButtonTextInteractionDemo() {
    // 1. 定义状态：remember 保证重组时状态不丢失，mutableStateOf 让状态可观察
    var message by remember { mutableStateOf("初始文字") }
    var clickCount by remember { mutableStateOf(0) }

    // 垂直布局，子组件居中
    Column(
        modifier = Modifier.padding(20.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        // 2. Text 依赖状态显示内容
        Text(
            text = message,
            modifier = Modifier.padding(bottom = 16.dp)
        )
        
        // 3. Button 点击修改状态，UI 自动更新
        Button(
            onClick = {
                clickCount++
                message = "已点击 $clickCount 次"
            }
        ) {
            Text(text = "点击修改文字")
        }
        
        // 重置按钮
        Button(
            onClick = {
                clickCount = 0
                message = "初始文字"
            },
            modifier = Modifier.padding(top = 8.dp)
        ) {
            Text(text = "重置")
        }
    }
}

// 预览
@Preview(showBackground = true, name = "按钮修改文字示例")
@Composable
fun ButtonTextInteractionDemoPreview() {
    ButtonTextInteractionDemo()
}
```

**关键解释（适配 XML 开发者）**：

- `var message by remember { mutableStateOf("初始文字") }`：等价于 XML 中 “定义 TextView + 保存文字的变量”，`remember` 避免组件重组时状态重置，`mutableStateOf` 让变量变成 “可观察状态”；
- 点击按钮时只需修改 `message` 和 `clickCount`，无需手动调用 `setText()`，Compose 会自动检测状态变化并刷新 Text 组件；
- `by` 是 Kotlin 的委托属性，简化 `message.value` 的写法（等价于 `val messageState = remember { mutableStateOf("初始文字") }`，使用时 `messageState.value`）。

------

### 总结

1. **核心思想**：Compose 是声明式 UI，通过 “状态驱动” 替代 XML 的 “命令式操控”，只需关注状态，UI 自动更新；
2. **环境配置**：核心是开启 `compose true` 并引入正确的依赖，`@Preview` 可快速预览 UI 无需运行 App；
3. **基础组件**：Text/Button/Image 分别对应 XML 的 TextView/Button/ImageView，参数替代 XML 属性，`modifier` 是布局 / 样式的核心；
4. **状态交互**：最基础的交互需通过 `remember + mutableStateOf` 管理状态，修改状态即可自动更新 UI，无需手动操作控件。