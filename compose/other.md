收集了一些好帖子。

## Jetpack Compose从入门到精通：构建高效Android应用

[Jetpack Compose从入门到精通：构建高效Android应用-CSDN博客](https://blog.csdn.net/weixin_36299472/article/details/149265798)

前几章忽略。第四章开始。

### 4. 状态管理与UI更新

#### 4.1 `State`和`mutableStateOf`

##### 4.1.1 `State`类的作用与原理

在Jetpack Compose中，` State` 类是构建响应式UI的核心组件之一。它允许你声明UI组件的状态，并且当这些状态更新时，依赖于该状态的UI能够自动刷新。这是通过响应式编程范式实现的，它是一种编程范式，主要关注于数据流和变更的传播。

State 对象在声明时会保持一个值，当这个值改变时，任何依赖于该 State 的Composable函数都会重新执行。这一机制能够确保UI总是反映当前最新的数据状态。

举个例子，如果你有一个计数器应用，每次点击按钮时，计数值都会增加。在这个场景中，计数值就是一个状态，而按钮的点击事件处理函数会更新这个状态。每当状态更新时，显示计数值的文本组件就会自动重新绘制，以显示新的值。

#### 4.1.2 使用`mutableStateOf`管理可变状态

在实际应用中，状态通常需要修改，这就需要使用` mutableStateOf `来创建可变的状态对象。这个方法返回一个 `MutableState<T> `对象，其中 T 是你想要存储的数据类型。

当使用` mutableStateOf` 时，任何对返回值 value 属性的修改都会触发依赖于它的Composable函数的重组（recomposition）。重组是指重新执行Composable函数并更新UI以反映新的状态。

```kotlin
var count by mutableStateOf(0)
Button(onClick = { count++ }) {
    Text("Count is: $count")
}
```


在上面的代码中，每次点击按钮时，都会对 count 进行递增操作。由于 count 是用 mutableStateOf 创建的，每次它的值变化都会导致包含它的Button组件的重组，UI将反映出新的计数值。

#### 4.1.3 State对象的生命周期

状态对象的生命周期应该遵守特定的规则。状态应该与UI组件的生命周期相匹配。在Jetpack Compose中，状态对象应该在UI树的相同层级或在更高的层级声明，以确保它们的生命周期覆盖它们依赖的Composable函数。

避免在较低层级的Composable函数中创建状态对象，这会导致在不需要的时候重新创建状态对象，并且无法正确追踪状态的更新。正确管理状态的生命周期可以帮助开发者构建更稳定、更高效的UI。

### 4.2 数据流与副作用

#### 4.2.1 LiveData与StateFlow在Compose中的应用

在Compose中处理数据流的一个常见方式是使用LiveData和StateFlow，这两个都是在Android中广泛使用的响应式数据存储。LiveData是Android架构组件的一部分，而StateFlow是Kotlin协程的一部分。

LiveData和StateFlow都可以与Compose结合使用来管理UI状态，因为它们提供了一种观察数据变化并在变化时自动重组UI的方式。

LiveData主要通过 observe 方法来观察数据变化，而StateFlow则使用其 collect 方法。



在Compose中，我们可以利用 `LaunchedEffect` 和 副作用 （如 `SideEffect` ）来与LiveData和StateFlow交互。

#### 4.2.2 副作用处理与effect的使用

在Compose中，副作用（`side-effect`）是指任何影响UI以外的部分的操作，比如数据加载、更新本地存储或改变外部系统状态等。在Compose中处理副作用，应遵循声明式的副作用方法。

`LaunchedEffect`是一个协程作用域，它在给定的键值改变时启动一个新的协程。这对于发起副作用非常有用，比如异步数据加载。而 `SideEffect `是专门用来执行那些不依赖于Composable函数中的状态改变的副作用。

```kotlin
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
```

### 4.3 状态提升与共享状态

#### 4.3.1 状态提升的概念与实现

状态提升（`Lifting State Up`）是一种常见的设计模式，在这种模式中，状态（包括其相关的逻辑）从当前组件中“提升”到一个共同的祖先组件中。在Compose中，这通常意味着将状态变量和更新这些变量的逻辑移至更高层级的组件，通常是父组件或共同祖先。

这种方法有诸多优点。首先，它可以减少组件间的耦合，使得状态管理更加集中。其次，当多个组件需要响应同一状态变化时，状态提升可以避免冗余的代码，使得状态管理更为简洁。

#### 4.3.2 使用Ambient共享状态

为了在多个组件之间共享状态，Jetpack Compose引入了一种名为`Ambient`的机制。Ambients是一种特殊的Composable，可以将值“注入”到UI树中。子组件可以消费这些值，而无需在它们之间显式传递。

Ambients使用 @Composable 注解函数创建，并且可以存储任何类型的值，包括状态。通过Ambients，状态可以跨越组件层次，实现共享。

一个典型的例子是使用 remember 和 provide 函数创建和提供一个Ambient值：

```kotlin val LocalCounter = staticAmbientOf { 0 }
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
```

上面的代码创建了一个 CounterProvider ，它提供了一个可变的计数状态，然后在应用的其它地方通过 LocalCounter 这个Ambient值来访问和展示这个状态。

### 5. 布局系统

Jetpack Compose提供了多种布局组件，如 `Box `、 `Column `和 `Row `等，它们被设计为符合声明式编程范式的特性，极大地简化了布局的实现。

`Box `允许开发者在垂直或水平方向上层叠多个组件，这在创建自定义布局时尤其有用。

 `Column `与 `Row `是两个主要的线性布局组件，分别用于垂直和水平方向上排列子组件。它们都是 `Arrangement `类的扩展函数，允许灵活地控制子项的排列方式。

`ConstraintLayout`:

 是Android传统XML布局中非常强大的布局方式，而在Compose中也有着对应的实现。它允许通过定义组件之间的约束关系，实现复杂的布局排列。

下面是一个简单的例子，演示如何在Compose中使用 ConstraintLayout ：
```kotlin
@Composable
fun ConstraintLayoutExample() {
    ConstraintLayout {
        val (button, text) = createRefs()

    		Button(
            onClick = { /* Do something */ },
            content = { Text("Button") },
            modifier = Modifier.constrainAs(button) {
                top.linkTo(parent.top, margin = 16.dp)
                end.linkTo(parent.end, margin = 16.dp)
            }
        )
     
        Text(
            "This is a text",
            modifier = Modifier.constrainAs(text) {
                top.linkTo(button.bottom, margin = 16.dp)
                start.linkTo(parent.start, margin = 16.dp)
            }
        )
    }
}
```

### 6. Material Design

在Compose中构建遵循Material Design风格的UI，从引入必要的库开始，然后使用 `MaterialTheme `包裹你的UI内容。 `MaterialTheme `提供了一个主题上下文，该上下文包含了颜色、字体和形状等属性。

```kotlin
@Composable
fun MaterialThemeExample() {
    MaterialTheme {
        Surface(color = MaterialTheme.colors.background) {
            // 添加组件
            Column {
                Text(
                    text = "Hello, Material!",
                    style = MaterialTheme.typography.h5,
                    color = MaterialTheme.colors.onBackground,
                    modifier = Modifier.padding(16.dp)
                )
                Button(
                    onClick = { /* TODO: Handle button click */ },
                    modifier = Modifier.align(Alignment.CenterHorizontally)
                ) {
                    Text(text = "Click Me")
                }
            }
        }
    }
}
```

#### 使用Scaffold组织布局

Scaffold 是Material Design布局中一种高级组件，它提供了一种方便的方式来实现典型的布局结构。 Scaffold 允许你设置一个应用程序的上层结构，包括顶部的 TopAppBar ，底部的 BottomAppBar ，浮动的动作按钮 FloatingActionButton ，以及抽屉 Drawer 等。

```kotlin
@Composable
fun ScaffoldExample() {
    Scaffold(
        topBar = { TopAppBar(title = { Text("Scaffold TopAppBar") }) },
        bottomBar = { BottomAppBar { Text("Scaffold BottomAppBar") } },
        floatingActionButton = { FloatingActionButton(onClick = { /* TODO */ }) {
            Icon(Icons.Filled.Add, contentDescription = "Add")
        } },
        content = {
            // 主内容区域
            Text("Hello, Scaffold!")
        }
    )
}
```

在这个例子中， `Scaffold `组织了顶部的 `TopAppBar `，底部的 `BottomAppBar `，以及浮动的 `FloatingActionButton `。所有的组件都被放置在 `content `参数中，它们将在屏幕空间内按照层次结构进行布局。



#### 应用BottomAppBar、FloatingActionButton
BottomAppBar 和 FloatingActionButton 都是Material Design中的组件，通常用于底部操作区域。 BottomAppBar 是一个容器，用于放置一组按钮，而 FloatingActionButton 则提供了一个可以悬浮在界面上的圆形按钮，用于触发主要操作。

将 BottomAppBar 和 FloatingActionButton 整合到 Scaffold 中，可以实现一个典型的用户交互区域布局：
```kotlin
Scaffold(
    bottomBar = {
        BottomAppBar {
            // 底部按钮
            IconButton(onClick = { /* TODO */ }) {
                Icon(Icons.Filled.Home, contentDescription = "Home")
            }
            IconButton(onClick = { /* TODO */ }) {
                Icon(Icons.Filled.Favorite, contentDescription = "Favorite")
            }
            IconButton(onClick = { /* TODO */ }) {
                Icon(Icons.Filled.Person, contentDescription = "Profile")
            }
        }
    },
    floatingActionButton = {
        FloatingActionButton(onClick = { /* TODO */ }) {
            Icon(Icons.Filled.Add, contentDescription = "Add")
        }
    }
    // 其他参数
)
```


### 5. 数据同步机制

在 Jetpack Compose 中，Kotlin 的 Flow 与 collectAsState() 协同实现响应式状态更新。通过该组合函数，可将冷流自动收集并转化为可观察的 State 对象。

```kotlin 
val userFlow = remember { mutableStateFlow("Alice") }
val userName by userFlow.collectAsState()
Text(text = userName)
```

上述代码中， userFlow 作为状态源，每次发射新值时， Text 组件会自动重组。使用 remember 避免重复初始化，提升性能。

### 6. 性能优化建议

避免在循环或高频事件中调用 collectAsState()
对高频率发射的 Flow 使用 throttleFirst 或 debounce 限流
确保 Flow 在组合生命周期内被正确收集，防止内存泄漏。
