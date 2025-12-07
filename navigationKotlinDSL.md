https://developer.android.google.cn/guide/navigation?hl=zh-cn

# Android 传统 Navigation 组件使用指南

## 官方文档学习

### 1. 指引

https://developer.android.google.cn/guide/navigation?hl=zh-cn

在不同 fragment 之间切换和交互的组件。可以从简单的按钮点击到更复杂的模式（应用栏和抽屉导航栏）。

> 官方的文档提到了，可以将Dialog和抽屉方式，和底部选项卡方式都加入到navigation中。
>
> 但是文档相对缺乏，使用者寥寥，不建议初学者把他们加入到navigation中，你当做进阶去学习。让我们从简单的纯粹的学起。

包括如下概念和类：

* **NavHostFragment 宿主**。单 Activity 设计思想。宿主指的是包含当前导航目的地的界面元素，fragment就是显示在它上面的内容。
* **NavGraph 导航图**。一种数据结构。定义所有目的地和连接。
* **NavController 控制器**。管理目的地之间的协调，提供方法，导航，处理 deep links，back stacks返回栈等。
* **NavDestination 目的地，图中的节点**。就是内容界面。
* **Route路线**。唯一标识目的地及其所需的任何数据，可序列化的数据类型。

优势和功能：

- **动画和过渡**：为动画和过渡提供标准化资源。
- **deep link**：实现并处理可将用户直接转到目的地的深层链接。
- **界面模式**：只需极少的额外工作即可支持抽屉式导航栏和底部导航等模式。
- **类型安全**：支持在目的地之间传递[类型安全](https://developer.android.google.cn/guide/navigation/design/type-safety?hl=zh-cn)的数据。
- **ViewModel 支持**：允许将 `ViewModel` 的作用域限定为导航图，以便在导航图的目的地之间共享与界面相关的数据。
- **fragment 事务**：全面支持和处理 fragment 事务。
- **返回和向上**：默认情况下，系统会正确处理返回和向上操作。

>  提到了预测性返回导航功能 ，todo 如何支持？

### 2. Navigation Contoller

因为还没有学习到如何创建他的 xml，所以这里介绍的是一些函数用来调度NavGraph 的。

可以根据当前所在的类中，使用下列方法之一来找到**NavController**：

```kotlin
//常规使用
Framgent.findNavController()
View.findNavController()
Activity.findNavController(viewId:Int)

//其实上面的方法，他们内部实现中去找了navHostFragment，进而拿到的navController。因此你的初始化方式是使用 FragmentContainerView 创建 NavHostFragment 时或使用 FragmentTransaction 手动向 activity 添加 NavHostFragment 时，是找不到的，应该采用下面的方案：
val navHostFragment =
    supportFragmentManager.findFragmentById(R.id.nav_host_fragment) as NavHostFragment
val navController = navHostFragment.navController
```



### 3. 设计NavGraph

NavGraph 导航图。是一种数据结构。定义所有目的地和连接。

> **注意：**导航图不同于[返回堆栈](https://developer.android.google.cn/guide/navigation/backstack?hl=zh-cn)，后者是 `NavController` 内用于保存用户近期所到目的地的堆栈。

有三种常规类型的目的地（ nav destinations）：托管、对话框和 activity。下表概述了这三种目的地类型及其用途。

| 类型     | 说明                                                         | 用例                                                         |
| :------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 托管     | 填充整个导航宿主。也就是说，托管目的地的大小与导航宿主的大小相同，之前的目的地不会显示。 | 主界面和详情界面。                                           |
| 对话框   | 显示叠加界面组件。此界面与导航宿主的位置或大小无关。之前的目的地会显示在该目的地下方。 | 提醒、选择、表单。                                           |
| activity | 表示应用中的独特界面或功能。                                 | 充当导航图的退出点，启动与 Navigation 组件分开管理的新activity。在 Modern Android Development 中，应用由 1 个 activity 组成。因此，当与第三方 activity 互动或作为[迁移过程](https://developer.android.google.cn/guide/navigation/migrate?hl=zh-cn)的一部分时，最适合使用 activity 目的地。 |

使用 **NavHostFragment** 作为宿主。可以写kotlin DSL，也可以写 xml，还可以直接使用 android studio 编辑器。

* 1. **XML方式**：相对简单和便于传统的 Xml View 思维下的理解，这里不做学习，原因后面会讲。

* 2. **编辑器方式**：是xml方式的编辑模式，不推荐，不好直接理解代码。

* 3. **kotlin DSL方式**：麻烦，需要自行编写 fragment{}, activity{}等createGraph 来实现， navigation 泛型跳转等。但是十分接近 Compose写法。

  优点：

  类似 compose方便以后学习 compose 迁移；

  相对简洁和更现代；

  可动态从服务器下发配置，动态构建导航图。

接下来，原文章是的讲compose，讲xml的不适用的部分就略掉了。只学习 DSL 方式。

#### 3.1 引入

https://developer.android.google.cn/guide/navigation/design/kotlin-dsl?hl=zh-cn

```groovy
dependencies {
    def nav_version = "2.9.6"
    api "androidx.navigation:navigation-fragment-ktx:$nav_version"
  //无需其他了，不需要xml方式就只这一个就够了。
}
```

#### 3.2 想好界面

首先，想好你的所有目的地，（约等于以前你有多少个Activity的界面），以及相应的跳入参数和操作。

比如示例中，有个程序有2个界面，**home**和**detail**：

```
Destination //home界面
ID: home
Fragment: HomeViewPagerFragment


Destination  //详情界面
ID: plant_detail
Fragment: PlantDetailFragment
Argument: (name: plant_id, type: String) //进入该界面的参数

动作：
Action
ID: to_plant_detail
Destinationld: plant_detail
```

#### 3.3 在 activity 中引入 NavHostFragment

第一步在主 Activity 上，只放了一个 **FragmentContainerView**，注意不要有 app:navGraph 标签。

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        app:defaultNavHost="true"                                      
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</FrameLayout>
```

在 XML 中，action用于将 destination ID 与一个或多个参数绑定在一起。不过，使用Navigation DSL 时，route（路由）可以包含实参作。这意味着，在使用 DSL 时，不存在 action的概念。

todo 多 Activity 之间是否还需要再定义？看下来是嵌套navigation的意思。



#### 3.4 构建 NavGraph和Route

在 xml 构建中，编译过程生成了静态 id。现在使用DSL是运行时，所以使用**可序列化类型**来表示每一条路由。

##### 1) 注册fragment界面(destination)

```kotlin
//0. 定义Route，可序列化类型
@Serializable data object Home //代表Home界面
@Serializable data class PlantDetail(val id: String) //代表Detail详情页带参数

val navController = findNavController(R.id.nav_host_fragment) //1. 找到navController
navController.graph = navController.createGraph( //2. 创建NavGraph
    startDestination = Home
) {
  	//2.1 创建了一个Home界面
    fragment<HomeFragment, Home> {
        label = resources.getString(R.string.home_title)
    }
  	//2.2 创建了一个Detail二级页面
    fragment<PlantDetailFragment, PlantDetail> {
        label = resources.getString(R.string.plant_detail_title)
    }
}
```

其中，`fragment{}`是构建函数：

```kotlin
fragment<MyFragment, MyRoute> {
   label = getString(R.string.fragment_title)
   // custom argument types, deepLinks
}
```

`fragment{}`构建函数：

泛型：

​	第一个为 Fragment 类；

​	第二个是Route，必须是 Any 继承的可序列化的类型（包含了跳转需要的参数）

lambda：

​	label，自定义参数，deeplink。



todo 除了泛型以外custom argument types指的是什么？



##### 2) dialog destinations todo 先跳过学习

指的应用导航图中以对话框窗口的形式叠加在应用界面元素和内容之上。**需要处理 NavController 的返回栈。**

与 fragment{} 差不多。

```kotlin
// Define destinations with serializable classes or objects
@Serializable
object Home
@Serializable
object Settings

// Add the graph to the NavController with `createGraph()`.
navController.graph = navController.createGraph(
    startDestination = Home
) {
    // Associate the home route with the HomeFragment.
    fragment<HomeFragment, Home> {
        label = "Home"
    }

    // Define the settings destination as a dialog using DialogFragment.
    dialog<SettingsFragment, Settings> {
        label = "Settings"
    }
}
```

todo FloatingWindow 接口继承问题？似乎不用管，只需要使用` dialog{}`包裹正常的 DialogFragment 即可。

todo 如何处理返回栈？还是说，处理返回栈的意思是已经交给了 navigation 框架处理。

##### 3) activity destinations

泛型：

​	只有唯一的，就是路由，必须是 Any 继承的可序列化的类型（包含了跳转需要的参数和类型）

lambda：

​	label，自定义参数，deeplink。

​	多了一个activityClass。

```kotlin
activity<MyRoute> {
   label = getString(R.string.activity_title)
   // custom argument types, deepLinks...

   activityClass = MyActivity::class
}
```

通过这种方式来实现，我们将无需再找到 Activity 类名来显示启动。而且还能够自定义传参，不需要start activity+ bundle的方式了。

##### 4) 定义泛型参数

前面已经写了是通过构建函数的泛型来定义一个目的地的参数类型：

```kotlin
fragment<MyFragment, MyRoute> //作为泛型传入
activity<MyRoute> { //作为泛型传入
   activityClass = MyActivity::class
}
```

* 常规数据类型

支持`String, int, List<String>...`这些。

```kotlin
@Serializable
data class MyRoute(
  val id: String,
  val myList: List<Int>,
  val optionalArg: String? = null
)
```

* 自定义类型

通过自定义 **NavType** 类。 怎么说呢，就类似 Gson 的转换器。懂了吧。

比如，如下定义了搜索路由的自定义SearchParameters。

```kotlin
//搜索界面的，SearchRoute
@Serializable
data class SearchRoute(val parameters: SearchParameters)

//自定义类型
@Serializable
@Parcelize
data class SearchParameters(
  val searchQuery: String,
  val filters: List<String>
)

//复杂类型转换器
val SearchParametersType = object : NavType<SearchParameters>(
  isNullableAllowed = false
) {
  override fun put(bundle: Bundle, key: String, value: SearchParameters) {
    bundle.putParcelable(key, value)
  }
  override fun get(bundle: Bundle, key: String): SearchParameters {
    return bundle.getParcelable(key) as SearchParameters
  }

  override fun serializeAsValue(value: SearchParameters): String {
    // Serialized values must always be Uri encoded
    return Uri.encode(Json.encodeToString(value))
  }

  override fun parseValue(value: String): SearchParameters {
    // Navigation takes care of decoding the string
    // before passing it to parseValue()
    return Json.decodeFromString<SearchParameters>(value)
  }
}

//注册：携带了 typeMap 做映射转换
fragment<SearchFragment, SearchRoute>(
    typeMap = mapOf(typeOf<SearchParameters>() to SearchParametersType)
) {
    label = getString(R.string.plant_search_title)
}


//如何跳转
val params = SearchParameters("rose", listOf("available"))
navController.navigate(route = SearchRoute(params))

//目的地获取
val searchRoute = navController().getBackStackEntry<SearchRoute>().toRoute<SearchRoute>()
val params = searchRoute.parameters
```

> 复杂类型，麻烦的很咯。我建议还是自己搞gson，我们传入String，反解析就好了，效率差不多，更简单。



##### 5) 跳转和获取参数

使用 `navigate() `函数跳转:

```kotlin
private fun navigateToPlant(plantId: String) {
   findNavController().navigate(route = PlantDetail(id = plantId))
}
```

进入后，可以在目标Fragment中，可以通过如下代码获取参数：

```kotlin
val plantDetailRoute = findNavController().getBackStackEntry<PlantDetail>().toRoute<PlantDetail>()
val plantId = plantDetailRoute.id

//viewModel中的方式
val plantDetailRoute = savedStateHandle.toRoute<PlantDetail>()
val plantId = plantDetailRoute.id
```

todo，back stack怎么管理？

#### 3.5 嵌套 NavGraph

思考一个app，用户首次启动有 guide_screen，register 界面。主界面是 home，然后有二级界面 detail，settings 等。

后续注册使用，用户应该直接进入 home。

最佳实践是将 home 界面设置为顶级导航图的起始 Destination。

并将guide_screen，register 移到嵌套图中。home 界面启动后检查是否有注册用户，没有注册就跳转到注册界面。

```kotlin
@Serializable data object HomeGraph
@Serializable data object Home

navigation<HomeGraph>(startDestination = Home) {
   // label, other destinations, deep links
}
```

使用`navigation<>()`函数实现嵌套导航图。

泛型：一张新的 Graph；

参数：startDestination 入口的界面；

lambda：与其他一致。



todo：navigation函数应该放在哪里执行？

todo：如何定义一张新的 Graph呢？

todo：home 界面启动后检查是否有注册用户，没有注册就跳转到注册界面。听起来像是需要在home界面的onCreate函数里面，检查是否登录了，没有的话，立刻navigate到注册界面，但是是否要销毁呢，back stack如何管理？

todo：条件导航？



#### 3.6 deeplink todo

首先要明确，deeplink指的是：**能从别的应用跳过来，打开某个界面，带或者不带参数的方式**。

而对于那种不公开给外部的，我们就只需要做成fragment，通过第二个泛型传参即可。

对于有外部需要的，我们需要定义成Activity。在android标准中，我们已经有action intent-filter scheme的方式。

如果是单一架构的Activity，可以定义多个intent-filter + scheme来实现多deeplink的传入。

> 如果是 xml 方式使用的时候，有显式和隐式两种方式。
>
> 显式深层链接是深层链接的一个实例，该实例使用 PendingIntent 将用户定向到应用内的特定位置。例如，您可以在通知或应用 widget 中显示显式深层链接。
>
> 当用户通过显式深层链接打开您的应用时，任务返回堆栈会被清除，并被替换为相应的深层链接目的地。当嵌套图表时，每个嵌套级别的起始目的地（即层次结构中每个 <navigation> 元素的起始目的地）也会添加到相应堆栈中。也就是说，当用户从深层链接目的地按下返回按钮时，他们会返回到相应的导航堆栈，就像从入口点进入您的应用一样。

如果是 xml 方式使用的时候，有显式和隐式两种方式。它是通过给 navigation.xml 和 Android manifest Activity 标签修改内容，最终 navigation 插件在编译阶段修改成了传统代码。而我们学习的 DSL。就不利用插件了，也不支持。

所以我们需要自行给 Activity 添加传统的Intent方式。

```xml
<activity
    android:name="com.example.android.GizmosActivity"
    android:label="@string/title_gizmos" >
    <intent-filter android:label="@string/filter_view_http_gizmos">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "http://www.example.com/gizmos” -->
        <data android:scheme="http"
              android:host="www.example.com"
              android:pathPrefix="/gizmos" />
        <!-- note that the leading "/" is required for pathPrefix-->
    </intent-filter>
    <intent-filter android:label="@string/filter_view_example_gizmos">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- Accepts URIs that begin with "example://gizmos” -->
        <data android:scheme="example"   
              android:host="gizmos" 
              android:mimeType="image/*"/>
    </intent-filter>
</activity>
```

```kotlin
              
//读取：
val action: String? = intent?.action
val data: Uri? = intent?.data

// 获取 plantId 参数
val plantId = arguments?.getString("plantId") ?: "unknown"
        
```

然后如何注册？在申明 desitination 的地方追加 deeplink：

todo试一试能不能添加多个deeplink？

```kotlin
@Serializable data object Home

fragment<HomeFragment, Home>{
  deepLink<Home>(basePath = "www.example.com/home"){
    // Optionally, specify the action and/or mime type that this destination
    // supports
    action = "android.intent.action.MY_ACTION"
    mimeType = "image/*"
  }
}
```

这里可以看到，传统的几个要素的对应关系：

```
<action android:name="android.intent.action.VIEW" />
对应 action

<data android:scheme="example"   
      android:host="gizmos" 
      android:mimeType="image/*"/>
basePath 就是 scheme/host
mimeType 对应 mimeType
```

**额外 URI 参数**

比如我们定义了一个 route 它的类型是：

```kotlin
@Serializable data class PlantDetail(
  val id: String,
  val name: String,
  val colors: List<String>,
  val latinName: String? = null,
)
```

```
// 方法1: 使用 URI 格式
val deepLinkUri = "https://www.example.com/plants/$plantId"
findNavController().navigate(Uri.parse(deepLinkUri))

// 方法2: 直接使用路由（更简单）
findNavController().navigate("plantDetail/$plantId")
```

不要使用 Safe Args。不支持 DSL。

todo 需要进一步看一下deeplink 传参到底是如何。是否我们可以忽略deeplink章节的学习，继续使用标准android来做。我推测只需要不在注册fragment函数中提到deeplink就不会有这个了。 



#### 3.7 自定义 Destination todo

```kotlin
// The NavigatorProvider is retrieved from the NavController
val customDestination = navigatorProvider[CustomNavigator::class].createDestination().apply {
    route = Graph.CustomDestination.route
}
addDestination(customDestination)
```

https://developer.android.google.cn/guide/navigation/design/add-new?hl=zh-cn

todo 需要配合这里没看懂，似乎是挺进阶的使用方法。navigator，activityNavigator，嵌套的 navigatorProvider 等，addNavigator()等。



### 4. 使用 NavGraph

#### 4.1 跳转

使用` NavController`来实现跳转，调用`navigate()`函数即可，对于不同的架构函数是不同的。对于 compose 是`navigate<T>(route= 序列化类型)`

```kotlin
@Serializable
object FriendsList
navController.navigate(route = FriendsList)
```

如果是 xml 方式，则是`navigate(int)`:

```kotlin
viewTransactionsButton.setOnClickListener { view ->
  view.findNavController().navigate(R.id.viewTransactionsAction)
}
```

而我们重点，是学习 kotlin DSL 方式。则是

`navigate(route= MyRoute())`

todo



# AI 





传统 Navigation 组件是 Android Jetpack 的一部分，专门用于在基于 View 和 Fragment 的应用中管理导航。以下是详细的使用方式：

## 1. 添加依赖

在 `build.gradle` 文件中添加依赖：



```gradle
dependencies {
    def nav_version = "2.9.6"
    
    // Java 版本
    implementation "androidx.navigation:navigation-fragment:$nav_version"
    implementation "androidx.navigation:navigation-ui:$nav_version"
    
    // Kotlin 版本
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
    
    // 可选：功能模块支持
    implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"
}
```



## 2. 创建导航图 (Navigation Graph)

在 `res/navigation/` 目录下创建导航图 XML 文件：

```xml
<!-- res/navigation/nav_graph.xml -->
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <!-- 定义 Fragment 目的地 -->
    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        android:label="Home"
        tools:layout="@layout/fragment_home">
        
        <!-- 定义跳转动作 -->
        <action
            android:id="@+id/action_home_to_detail"
            app:destination="@id/detailFragment" />
            
        <action
            android:id="@+id/action_home_to_profile"
            app:destination="@id/profileFragment" />
    </fragment>

    <fragment
        android:id="@+id/detailFragment"
        android:name="com.example.app.DetailFragment"
        android:label="Detail"
        tools:layout="@layout/fragment_detail">
        
        <!-- 带参数的跳转 -->
        <argument
            android:name="itemId"
            app:argType="integer"
            android:defaultValue="0" />
            
        <argument
            android:name="itemName"
            app:argType="string" />
    </fragment>

    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.app.ProfileFragment"
        android:label="Profile"
        tools:layout="@layout/fragment_profile" />

    <!-- Activity 目的地 -->
    <activity
        android:id="@+id/settingsActivity"
        android:name="com.example.app.SettingsActivity"
        android:label="Settings" />

</navigation>
```



## 3. 设置 NavHost

在主 Activity 的布局中添加 NavHostFragment：

xml

```
<!-- activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- 可选的顶部导航栏 -->
    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize" />

    <!-- 底部导航栏 -->
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu" />

    <!-- NavHostFragment 容器 -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

</LinearLayout>
```



## 4. 配置主 Activity

kotlin

```
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    
    private lateinit var navController: NavController
    private lateinit var appBarConfiguration: AppBarConfiguration
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 设置 Toolbar
        setSupportActionBar(findViewById(R.id.toolbar))
        
        // 获取 NavController
        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController
        
        // 配置 AppBarConfiguration
        appBarConfiguration = AppBarConfiguration(
            setOf(R.id.homeFragment, R.id.profileFragment)
        )
        
        // 设置 ActionBar 与导航的关联
        setupActionBarWithNavController(navController, appBarConfiguration)
        
        // 设置底部导航
        setupBottomNavigation()
    }
    
    private fun setupBottomNavigation() {
        val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav)
        bottomNav.setupWithNavController(navController)
    }
    
    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
    }
}
```



## 5. 基本导航操作

### 在 Fragment 中进行导航：

kotlin

```
// HomeFragment.kt
class HomeFragment : Fragment() {
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // 简单的导航跳转
        btn_go_to_detail.setOnClickListener {
            // 方式1：使用 action ID
            findNavController().navigate(R.id.action_home_to_detail)
        }
        
        // 带参数的导航
        btn_go_with_args.setOnClickListener {
            // 方式2：使用目的地 ID 和参数 Bundle
            val args = Bundle().apply {
                putInt("itemId", 123)
                putString("itemName", "示例项目")
            }
            findNavController().navigate(R.id.detailFragment, args)
        }
        
        // 使用 Safe Args（推荐）
        btn_safe_args.setOnClickListener {
            val direction = HomeFragmentDirections.actionHomeToDetail(
                itemId = 456,
                itemName = "安全参数示例"
            )
            findNavController().navigate(direction)
        }
    }
}
```



## 6. Safe Args 的使用

### 在项目的 `build.gradle` 中添加插件：

gradle

```
// 项目级别的 build.gradle
buildscript {
    dependencies {
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:2.7.7"
    }
}

// 模块级别的 build.gradle
plugins {
    id 'androidx.navigation.safeargs.kotlin'
}
```



### 使用生成的 Safe Args 类：

kotlin

```
// 发送参数
val direction = HomeFragmentDirections.actionHomeToDetail(
    itemId = 123,
    itemName = "测试项目"
)
findNavController().navigate(direction)

// 接收参数
class DetailFragment : Fragment() {
    private val args: DetailFragmentArgs by navArgs()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        val itemId = args.itemId
        val itemName = args.itemName
        
        // 使用参数更新 UI
        textView.text = "ID: $itemId, 名称: $itemName"
    }
}
```



## 7. 深层链接 (Deep Links)

### 配置深层链接：

xml

```
<!-- 在 nav_graph.xml 中 -->
<fragment
    android:id="@+id/detailFragment"
    android:name="com.example.app.DetailFragment"
    android:label="Detail">
    
    <argument
        android:name="itemId"
        app:argType="integer" />
    
    <!-- 添加深层链接 -->
    <deepLink
        android:id="@+id/deepLink"
        app:uri="exampleapp://detail/{itemId}" />
</fragment>
```



### 在 Manifest 中声明：

xml

```
<activity
    android:name=".MainActivity"
    android:exported="true">
    
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="exampleapp" />
    </intent-filter>
</activity>
```



## 8. 导航回退栈管理

kotlin

```
// 自定义导航行为
val action = HomeFragmentDirections.actionHomeToDetail().apply {
    // 弹出回退栈直到指定目的地
    popUpTo(R.id.homeFragment) {
        inclusive = true  // 是否包含 homeFragment 本身
    }
    
    // 启动单例模式（清除所有中间页面）
    launchSingleTop = true
}

findNavController().navigate(action)

// 手动处理返回按钮
requireActivity().onBackPressedDispatcher.addCallback(viewLifecycleOwner) {
    // 自定义返回逻辑
    if (shouldHandleBack) {
        handleBackPress()
    } else {
        isEnabled = false
        requireActivity().onBackPressed()
    }
}
```



## 9. 条件导航

kotlin

```
fun navigateBasedOnCondition() {
    val navController = findNavController()
    
    when {
        isUserLoggedIn() -> {
            navController.navigate(R.id.profileFragment)
        }
        hasPendingNotifications() -> {
            navController.navigate(R.id.notificationsFragment)
        }
        else -> {
            navController.navigate(R.id.homeFragment)
        }
    }
}
```



这就是传统 Navigation 组件的基本使用方法。这套系统非常成熟稳定，适合所有基于 View 和 Fragment 的 Android 应用，提供了强大的导航管理和类型安全的参数传递机制。







## 创建底部导航菜单

### 1. 创建菜单资源文件

在 `res/menu/` 目录下创建 `bottom_nav_menu.xml`：

xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/homeFragment"
        android:icon="@drawable/ic_home"
        android:title="首页" />
        
    <item
        android:id="@+id/dashboardFragment"
        android:icon="@drawable/ic_dashboard"
        android:title="仪表板" />
        
    <item
        android:id="@+id/notificationsFragment"
        android:icon="@drawable/ic_notifications"
        android:title="通知" />
        
    <item
        android:id="@+id/profileFragment"
        android:icon="@drawable/ic_profile"
        android:title="个人资料" />
</menu>
```



### 2. 更新导航图

确保导航图中包含对应的 Fragment：

xml

```
<!-- res/navigation/nav_graph.xml -->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        android:label="首页"
        tools:layout="@layout/fragment_home" />
        
    <fragment
        android:id="@+id/dashboardFragment"
        android:name="com.example.app.DashboardFragment"
        android:label="仪表板"
        tools:layout="@layout/fragment_dashboard" />
        
    <fragment
        android:id="@+id/notificationsFragment"
        android:name="com.example.app.NotificationsFragment"
        android:label="通知"
        tools:layout="@layout/fragment_notifications" />
        
    <fragment
        android:id="@+id/profileFragment"
        android:name="com.example.app.ProfileFragment"
        android:label="个人资料"
        tools:layout="@layout/fragment_profile" />

</navigation>
```



### 3. 完整的 Activity 实现

kotlin

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    
    private lateinit var navController: NavController
    private lateinit var appBarConfiguration: AppBarConfiguration
    private lateinit var bottomNavigationView: BottomNavigationView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 初始化视图
        setupToolbar()
        setupNavigation()
    }
    
    private fun setupToolbar() {
        setSupportActionBar(findViewById(R.id.toolbar))
    }
    
    private fun setupNavigation() {
        // 获取 NavController
        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController
        
        // 配置 AppBarConfiguration - 指定顶级目的地
        appBarConfiguration = AppBarConfiguration(
            setOf(
                R.id.homeFragment,
                R.id.dashboardFragment, 
                R.id.notificationsFragment,
                R.id.profileFragment
            )
        )
        
        // 设置 ActionBar
        setupActionBarWithNavController(navController, appBarConfiguration)
        
        // 设置底部导航
        setupBottomNavigation()
    }
    
    private fun setupBottomNavigation() {
        bottomNavigationView = findViewById(R.id.bottom_nav)
        
        // 方法1：自动处理导航（推荐）
        bottomNavigationView.setupWithNavController(navController)
        
        // 方法2：手动处理导航（更灵活的控制）
        // setupManualBottomNavigation()
    }
    
    /**
     * 手动处理底部导航的点击事件
     * 这种方式可以添加自定义逻辑
     */
    private fun setupManualBottomNavigation() {
        bottomNavigationView.setOnItemSelectedListener { menuItem ->
            when (menuItem.itemId) {
                R.id.homeFragment -> {
                    // 可以在这里添加额外逻辑
                    if (navController.currentDestination?.id != R.id.homeFragment) {
                        navController.navigate(R.id.homeFragment)
                    }
                    true
                }
                R.id.dashboardFragment -> {
                    // 检查权限或其他条件
                    if (hasDashboardAccess()) {
                        navController.navigate(R.id.dashboardFragment)
                        true
                    } else {
                        showAccessDeniedMessage()
                        false
                    }
                }
                R.id.notificationsFragment -> {
                    navController.navigate(R.id.notificationsFragment)
                    true
                }
                R.id.profileFragment -> {
                    navController.navigate(R.id.profileFragment)
                    true
                }
                else -> false
            }
        }
        
        // 监听导航变化，更新底部导航选中状态
        navController.addOnDestinationChangedListener { _, destination, _ ->
            when (destination.id) {
                R.id.homeFragment -> bottomNavigationView.selectedItemId = R.id.homeFragment
                R.id.dashboardFragment -> bottomNavigationView.selectedItemId = R.id.dashboardFragment
                R.id.notificationsFragment -> bottomNavigationView.selectedItemId = R.id.notificationsFragment
                R.id.profileFragment -> bottomNavigationView.selectedItemId = R.id.profileFragment
            }
        }
    }
    
    private fun hasDashboardAccess(): Boolean {
        // 检查用户是否有权限访问仪表板
        return true // 实际实现中这里会有具体的权限检查逻辑
    }
    
    private fun showAccessDeniedMessage() {
        Toast.makeText(this, "无权访问此功能", Toast.LENGTH_SHORT).show()
    }
    
    override fun onSupportNavigateUp(): Boolean {
        return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
    }
}
```



### 4. 图标资源

您需要在 `res/drawable/` 目录下添加对应的图标：

- `ic_home.xml` (首页图标)
- `ic_dashboard.xml` (仪表板图标)
- `ic_notifications.xml` (通知图标)
- `ic_profile.xml` (个人资料图标)

示例图标文件 (`ic_home.xml`)：

xml

```
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
  <path
      android:fillColor="@color/colorPrimary"
      android:pathData="M10,20v-6h4v6h5v-8h3L12,3 2,12h3v8z"/>
</vector>
```



### 5. 高级用法 - 显示徽章

kotlin

```
// 在 Fragment 或 Activity 中显示通知徽章
fun showNotificationBadge(count: Int) {
    val badge = bottomNavigationView.getOrCreateBadge(R.id.notificationsFragment)
    badge.number = count
    badge.backgroundColor = ContextCompat.getColor(this, R.color.red)
    badge.badgeTextColor = ContextCompat.getColor(this, R.color.white)
    badge.isVisible = count > 0
}

// 隐藏徽章
fun hideNotificationBadge() {
    bottomNavigationView.removeBadge(R.id.notificationsFragment)
}
```



### 6. 处理深层链接时的底部导航

kotlin

```
// 确保从深层链接进入时底部导航状态正确
navController.addOnDestinationChangedListener { _, destination, _ ->
    // 更新底部导航选中状态
    when (destination.id) {
        R.id.homeFragment,
        R.id.dashboardFragment,
        R.id.notificationsFragment,
        R.id.profileFragment -> {
            bottomNavigationView.selectedItemId = destination.id
        }
    }
    
    // 隐藏底部导航（如果需要）
    when (destination.id) {
        R.id.homeFragment,
        R.id.dashboardFragment,
        R.id.notificationsFragment,
        R.id.profileFragment -> {
            bottomNavigationView.visibility = View.VISIBLE
        }
        else -> {
            bottomNavigationView.visibility = View.GONE
        }
    }
}
```



现在 `bottom_nav_menu` 就完全集成到导航系统中了！当用户点击底部导航项时，会自动导航到对应的 Fragment，并且底部导航的选中状态也会随着导航变化自动更新。
