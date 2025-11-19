

### 第一课 简介

[Jetpack Compose UI App Development Toolkit - Android Developers](https://developer.android.google.cn/compose)

官方文档学习

#### **选择compose的理由**

更少代码，更直观。

可以构建不与特定 activity 或 fragment 相关联的小型无状态组件。

改进无障碍，布局。

动画更轻松。

#### **编程思想**

View框架：命令式UI模型。通过findViewById得到元素，直接修改界面元素的属性，修改了内部状态。

​	通过加载xml实例化widget树。每一个widget内部又有自己的状态，通过getter/setter直接与widget交互。



compose框架：整个行业都转向声明式UI模型。现在是，在概念上从头开始重新生成整个屏幕，然后仅执行必要的更改。这对框架提出的挑战是如何智能地减少重绘的资源消耗。

​	在声明式方法中，widget不暴露setter/getter函数，也是相对的没有状态。这样通过ViewModel观察数据变更后，Composable函数就可以将当前应用状态转变为UI。

**数据沿着可组合函数层次往下流动**：应用会向顶级composable函数提供数据，沿着层级结构向下传递：

<img src=".\pictures\compose数据向下变更UI.png" alt="compose数据向下变更UI" style="zoom:47%;" />

**事件沿着可组合函数层次向上传递**：用户与UI元素交互，触发事件，进而app使用新数据再次调用Composable函数刷新界面。这叫做*recomposition*(重组)。

<img src=".\pictures\compose用户事件向上传递.png" alt="compose用户事件向上传递" style="zoom:40%;" />

##### 简单的可组合函数(@Composable)

```kotlin
@Composable
fun SimpleTestUi(name: String) {
    Text("Simple Ui $name!")
}

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()
    setContent {
        SimpleTestUi("Hello compose!")
    }
}
```

<img src="./pictures/jetpack-simpleText.png" alt="jetpack-simpleText" style="zoom:50%;" />

(暂时不要管为什么顶在statusBar上面。android15的edgeToEdge默认沉浸式。后面学到如何padding再讲解。1️⃣)

* `@Composable`注解。交给编译器转变为界面。

* 可以入参。
* `Text()`是一个composable函数，还可以结合其他composable函数来生成UI层次结构。
* 没有返回值。因为我们期待的屏幕的状态，而不是一个组件。

##### Recomposition（重组）


```kotlin
@Composable
fun ClickCounter(clicks: Int, onClick: () -> Unit) {
    Button(onClick = onClick) {
        Text("I've been clicked $clicks times")
    }
}

//一个真实可以用的为如下代码。
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    enableEdgeToEdge()
    setContent {
        AndroidCompontsTheme {
            Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->
                MainUi(
                    name = "Android",
                    modifier = Modifier.padding(innerPadding)
                )
            }
        }
    }
}

@Composable
fun MainUi(name: String, modifier: Modifier = Modifier) {
    // 使用remember和mutableStateOf创建Compose状态变量
    var clicks by remember { mutableIntStateOf(0) }

    Column(
        modifier = modifier.background(
            color = Color.Transparent,
            shape = RoundedCornerShape(Dp(4f))
        ),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        ClickCounter(clicks) {
            clicks++
        }
    }
}
```

在命令式界面模型中，如需更改某个 widget，您可以在该 widget 上调用 setter 以更改其内部状态。在 Compose 中，您可以使用新数据再次调用可组合函数。

重组是指在输入更改时再次调用可组合函数的过程。当函数的输入更改时，会发生这种情况。当 Compose 根据新输入重组时，它仅调用可能已更改的函数或 lambda，而跳过其余函数或 lambda。通过跳过所有未更改参数的函数或 lambda，Compose 可以高效地重组。

**Side-Effect**（翻译：**附带效应** 或者 **副作用**） 避免在compose中进行如下操作：

**3个避免**：

* 写入共享对象的属性；
* 更新ViewModel中的可观察对象；
* 更新SharedPreferences。

为了尽可能高效的刷新UI，避免刷新不需要更新的其他组件部分。

所有可组合函数或 lambda 表达式的执行都应该无副作用。当你需要执行副作用操作时，请从回调函数中触发它。

当Compose认为可组合项的参数可能已发生变化时，重组就会开始。重组是*乐观的*，这意味着Compose期望在参数再次变化之前完成重组。如果某个参数在重组完成前*确实*发生了变化，Compose可能会取消当前重组，并使用新参数重新开始。当重组被取消时，Compose 会丢弃此次重组生成的 UI 树。如果您有任何依赖于正在显示的 UI 的副作用，即使重组被取消，这些副作用也会被应用。这可能会导致应用状态不一致。为了应对乐观重组，确保所有可组合函数和lambda表达式都是幂等的且无副作用。

如果您的可组合函数需要数据，它应该为这些数据定义参数。然后，您可以将耗时的工作转移到组合之外的另一个线程，并使用`mutableStateOf`或`LiveData`将数据传递给Compose。

**5个注意：**

- 重组会跳过尽可能多的可组合函数和 lambda。
- 重组是乐观的操作，可能会被取消。
- 可组合函数可能会像动画的每一帧一样非常频繁地运行。
- 可组合函数可以并行执行。
- 可组合函数可以按任何顺序执行。

> 3个避免：指的是，让composable函数不要跟耗时和副作用联系；
>
> 5个注意：指的是这些函数的执行是不可控的，可能是并行的，可能是任意顺序的，也可能很频繁，也可能不执行。这就要求我们写出保持快速、幂等且没有副作用的代码。



#### ~~构建自适应应用~~

~~根据应用显示的变化（主要是应用窗口的大小）来更改布局，还能适应可折叠设备折叠状态的变化（例如折叠状态），以及屏幕密度和字体大小的变化。还可以动态替换组件或者隐藏显示内容。比如双窗格变成列表之类。~~

- ~~使用窗口大小类别来做出布局决策~~
- ~~使用 Compose Material 3 自适应库进行构建~~
- ~~支持触控以外的输入方式~~

~~todo [构建自适应应用  | Jetpack Compose  | Android Developers](https://developer.android.google.cn/develop/ui/compose/build-adaptive-apps?hl=zh-cn)~~

~~暂时看来本章节，不用学习。后续再回头来做学习。~~



#### 第一课 todoList

第一课结束后，留下了大量问题：

| 问题列表                                                     |
| ------------------------------------------------------------ |
| Side-Effect （副作用）? “乐观”重组，这个事情看下来我们需要好好处理下副作用。 |
| 数据采用State (remember + mutableStateOf/...)方式承载。 State/mutableStateOf ? |
| flow如何结合？                                               |
| 前面看到了，该如何正确的做异步？似乎我们是有state来持有和更新数据，结合到flow，viewModel该如何做。 |
| Size相关，statusBar padding，边衬区....                      |

全屏相关：

[设置全屏显示  | Jetpack Compose  | Android Developers](https://developer.android.google.cn/develop/ui/compose/system/setup-e2e?hl=zh-cn#padding-modifiers)

safeDrawingPadding 用于Compose的边衬区：

https://developer.android.google.cn/develop/ui/compose/system/insets-ui?hl=zh-cn

而View使用：

[边衬区：应用圆角  | Views  | Android Developers](https://developer.android.google.cn/develop/ui/views/layout/insets/rounded-corners?authuser=4&hl=zh-cn)



### 第二课 界面架构

#### 生命周期

了解composable函数的生命周期和如何确定重组。





### 参考资料学习

[Jetpack Compose从入门到精通：构建高效Android应用-CSDN博客](https://blog.csdn.net/weixin_36299472/article/details/149265798)

4. 状态管理与UI更新
   4.1 State和mutableStateOf
   4.1.1 State类的作用与原理
   在Jetpack Compose中， State 类是构建响应式UI的核心组件之一。它允许你声明UI组件的状态，并且当这些状态更新时，依赖于该状态的UI能够自动刷新。这是通过响应式编程范式实现的，它是一种编程范式，主要关注于数据流和变更的传播。

State 对象在声明时会保持一个值，当这个值改变时，任何依赖于该 State 的Composable函数都会重新执行。这一机制能够确保UI总是反映当前最新的数据状态。

举个例子，如果你有一个计数器应用，每次点击按钮时，计数值都会增加。在这个场景中，计数值就是一个状态，而按钮的点击事件处理函数会更新这个状态。每当状态更新时，显示计数值的文本组件就会自动重新绘制，以显示新的值。

4.1.2 使用mutableStateOf管理可变状态
在实际应用中，状态通常需要修改，这就需要使用 mutableStateOf 来创建可变的状态对象。这个方法返回一个 MutableState<T> 对象，其中 T 是你想要存储的数据类型。

当使用 mutableStateOf 时，任何对返回值 value 属性的修改都会触发依赖于它的Composable函数的重组（recomposition）。重组是指重新执行Composable函数并更新UI以反映新的状态。

var count by mutableStateOf(0)
Button(onClick = { count++ }) {
    Text("Count is: $count")
}
一键获取完整项目代码
kotlin
在上面的代码中，每次点击按钮时，都会对 count 进行递增操作。由于 count 是用 mutableStateOf 创建的，每次它的值变化都会导致包含它的Button组件的重组，UI将反映出新的计数值。

4.1.3 State对象的生命周期
状态对象的生命周期应该遵守特定的规则。状态应该与UI组件的生命周期相匹配。在Jetpack Compose中，状态对象应该在UI树的相同层级或在更高的层级声明，以确保它们的生命周期覆盖它们依赖的Composable函数。

避免在较低层级的Composable函数中创建状态对象，这会导致在不需要的时候重新创建状态对象，并且无法正确追踪状态的更新。正确管理状态的生命周期可以帮助开发者构建更稳定、更高效的UI。

4.2 数据流与副作用
4.2.1 LiveData与StateFlow在Compose中的应用
在Compose中处理数据流的一个常见方式是使用LiveData和StateFlow，这两个都是在Android中广泛使用的响应式数据存储。LiveData是Android架构组件的一部分，而StateFlow是Kotlin协程的一部分。

LiveData和StateFlow都可以与Compose结合使用来管理UI状态，因为它们提供了一种观察数据变化并在变化时自动重组UI的方式。

LiveData主要通过 observe 方法来观察数据变化，而StateFlow则使用其 collect 方法。在Compose中，我们可以利用 LaunchedEffect 和 副作用 （如 SideEffect ）来与LiveData和StateFlow交互。

4.2.2 副作用处理与effect的使用
在Compose中，副作用（side-effect）是指任何影响UI以外的部分的操作，比如数据加载、更新本地存储或改变外部系统状态等。在Compose中处理副作用，应遵循声明式的副作用方法。

LaunchedEffect 是一个协程作用域，它在给定的键值改变时启动一个新的协程。这对于发起副作用非常有用，比如异步数据加载。而 SideEffect 是专门用来执行那些不依赖于Composable函数中的状态改变的副作用。

LaunchedEffect(key1 = someLiveData) {
    someLiveData.observeForever {
        // 这是一个副作用，因为这里会发起异步操作
        println("Data changed to $it")
    }
}

SideEffect {
    // 这是一个副作用，比如在这里更新一个外部的UI状态
    updateExternalState(someState.value)
}
一键获取完整项目代码
kotlin

4.3 状态提升与共享状态
4.3.1 状态提升的概念与实现
状态提升（Lifting State Up）是一种常见的设计模式，在这种模式中，状态（包括其相关的逻辑）从当前组件中“提升”到一个共同的祖先组件中。在Compose中，这通常意味着将状态变量和更新这些变量的逻辑移至更高层级的组件，通常是父组件或共同祖先。

这种方法有诸多优点。首先，它可以减少组件间的耦合，使得状态管理更加集中。其次，当多个组件需要响应同一状态变化时，状态提升可以避免冗余的代码，使得状态管理更为简洁。

4.3.2 使用Ambient共享状态
为了在多个组件之间共享状态，Jetpack Compose引入了一种名为Ambient的机制。Ambients是一种特殊的Composable，可以将值“注入”到UI树中。子组件可以消费这些值，而无需在它们之间显式传递。

Ambients使用 @Composable 注解函数创建，并且可以存储任何类型的值，包括状态。通过Ambients，状态可以跨越组件层次，实现共享。

一个典型的例子是使用 remember 和 provide 函数创建和提供一个Ambient值：

val LocalCounter = staticAmbientOf { 0 }

@Composable
fun CounterProvider(content: @Composable () -> Unit) {
    val count = remember { mutableStateOf(0) }
    provide(LocalCounter provides count) {
        content()
    }
}

@Composable
fun CounterDisplay() {
    val count = LocalCounter.current
    Text("Count: $count")
}

@Composable
fun App() {
    CounterProvider {
        CounterDisplay()
    }
}

上面的代码创建了一个 CounterProvider ，它提供了一个可变的计数状态，然后在应用的其它地方通过 LocalCounter 这个Ambient值来访问和展示这个状态。



数据同步机制
在 Jetpack Compose 中，Kotlin 的 Flow 与 collectAsState() 协同实现响应式状态更新。通过该组合函数，可将冷流自动收集并转化为可观察的 State 对象。

```kotlin 
val userFlow = remember { mutableStateFlow("Alice") }
val userName by userFlow.collectAsState()
Text(text = userName)
```

一键获取完整项目代码
上述代码中， userFlow 作为状态源，每次发射新值时， Text 组件会自动重组。使用 remember 避免重复初始化，提升性能。
性能优化建议
避免在循环或高频事件中调用 collectAsState()
对高频率发射的 Flow 使用 throttleFirst 或 debounce 限流
确保 Flow 在组合生命周期内被正确收集，防止内存泄漏。





