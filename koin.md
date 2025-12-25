## 一、依赖注入概念

### 什么是依赖注入

依赖注入是一种设计模式，在这种模式中，对象从外部源接收其依赖项，而不是在内部创建它们。这有助于实现松耦合、提高可测试性，并使代码架构更清晰。

举例，有一辆汽车需要发动机。不使用依赖注入的情况下：：

```kotlin
class Engine {
    fun start() {
        println("Engine starting...")
    }
}

class Car { //难以独立测试Car
    private val engine = Engine()  // Car与特定的Engine实现紧密耦合, 不方便更换Engine， Car控制着Engine的生命周期

    fun drive() {
        engine.start()
        println("Car is driving")
    }
}
```

改成依赖注入：

```kotlin
class Car(private val engine: Engine) {  // engine是被注入的，Car不用管哪里来的
    fun drive() {
        engine.start()
        println("Car is driving")
    }
}

// Now we can easily provide different engines
val gasolineCar = Car(GasEngine())
val electricCar = Car(ElectricEngine())
```

易于使用模拟测试。

灵活性高——可更换实现方式。

构造函数中可见清晰的依赖关系。

### 直接创建

 ```kotlin
val database = Database()
val apiClient = ApiClient()
val userRepository = UserRepository(database, apiClient)
val authRepository = AuthRepository(database, apiClient)
val userService = UserService(userRepository, authRepository)
val viewModel = UserViewModel(userService)
....
 ```

* 问题：

> 活动/片段间的重复代码
>
> 依赖顺序容易出错
>
> 随着应用程序的增长，维护变得困难
>
> 难以管理生命周期（单例、作用域对象）
>
> 无集中式配置

### 容器模式

```kotlin
object AppContainer {
    private val database by lazy { Database() }
    private val apiClient by lazy { ApiClient() }

    val userRepository by lazy { UserRepository(database, apiClient) }
    val authRepository by lazy { AuthRepository(database, apiClient) }

    fun createUserViewModel() = UserViewModel(
        UserService(userRepository, authRepository)
    )
}

// Usage
class MainActivity : AppCompatActivity() {
    private val viewModel = AppContainer.createUserViewModel()
}
```

* 问题：

> 依赖项的手动连接
>
> 没有自动的生命周期管理
>
> 全局状态（单例容器）
>
> 对于复杂图仍然重复



### 什么是好的架构呢？

简洁易用，灵活，跨平台，学习成本低。

* **最佳实践**

  ```kotlin
  // ✅ 好的做法
  class UserService(private val api: UserApi, private val db: UserDatabase)
  
  module {
      singleOf(::UserService)
  }
  
  // ❌ 避免
  class UserService : KoinComponent {
      private val api: UserApi by inject()
      private val db: UserDatabase by inject()
  }
  ```

* **可重用性和解耦**

  没有依赖注入，组件是耦合的：

  ```kotlin
  class EmailService {
      fun sendEmail(to: String, message: String) {
          // Send via Gmail API
      }
  }
  
  class UserRegistration {
      private val emailService = EmailService()  //耦合性高
  
      fun register(user: User) {
          emailService.sendEmail(user.email, "Welcome!")
      }
  }
  ```

  使用依赖注入时，组件是松耦合的：

  ```kotlin
  interface EmailService {
      fun sendEmail(to: String, message: String)
  }
  
  class GmailService : EmailService { /* ... */ }
  class SendGridService : EmailService { /* ... */ }
  
  class UserRegistration(
      private val emailService: EmailService  //接口依赖
  ) {
      fun register(user: User) {
          emailService.sendEmail(user.email, "Welcome!")
      }
  }
  
  // 易于切换子类
  val appModule = module {
      single<EmailService> { GmailService() }  // or SendGridService()
      single { UserRegistration(get()) }
  }
  ```
  
* **更轻松的重构**

  当你需要添加一个新的依赖项时：

  ```kotlin
  // 以前的代码
  class UserService(
      private val repository: UserRepository
  )
  
  // 新代码 - 只需要添加参数和更新模块
  class UserService(
      private val repository: UserRepository,
      private val analytics: Analytics  //新增注入对象
  )
  
  val appModule = module {
      single { UserService(get(), get()) } 
  }
  ```

* **简化测试**

	没有依赖注入的时候：

  ```kotlin
  class UserService {
      private val repository = UserRepository()  // 没法模拟
  
      fun getUser(id: String): User {
          return repository.findUser(id)
      }
  }
  
  // 难以测试- 不能替换仓库类
  @Test
  fun testGetUser() {
      val service = UserService()  // 总是使用固定的类
  }
  ```

  有依赖注入的时候：

  ```kotlin
  class UserService(
      private val repository: UserRepository
  ) {
      fun getUser(id: String): User {
          return repository.findUser(id)
      }
  }
  
  // 易于测试
  @Test
  fun testGetUser() {
      val mockRepository = mock<UserRepository>()
      val service = UserService(mockRepository)  // Full control
  
      every { mockRepository.findUser("123") } returns testUser
  
      val result = service.getUser("123")
      assertEquals(testUser, result)
  }
  ```

### koin优势

* **模块验证**

  Koin可以在运行前验证你的依赖图：

  ```kotlin
  @Test
  fun verifyKoinConfiguration() {
      koinApplication {
          modules(appModule, networkModule, dataModule)
      }.checkModules()  // 如果依赖不能解析，就报错
  }
  ```

* **作用域**

  Koin的作用域功能允许你为应用程序的特定部分隔离依赖项：

  ```kotlin
  module {
      scope<MyActivity> {
          scoped { MyActivityDependency() }
      }
  }
  ```

* **延迟模块加载**（koin4.2.0+）

  仅在需要时加载模块：

  ```kotlin
  startKoin {
      modules(coreModule)
      lazyModules(
          featureAModule,
          featureBModule
      )
  }
  
  // Modules 在第一次被访问的时候加载
  val featureA: FeatureA = get()  // featureAModule现在开始加载
  ```

* **DSL**

	直观、类型安全的配置：

  ```kotlin
  val appModule = module {
      // 单例
      single { Database() }
  
      // 每次创建的类
      factory { Presenter() }
  
      // 作用域 跟随Activity的生命周期
      scope<MainActivity> {
          scoped { ActivityPresenter() }
      }
  
      // ViewModel生命周期
      viewModel { UserViewModel(get()) }
  }
  ```

* **更少的样板代码**

  Koin vs 注解型框架:

  ```kotlin
  //koin
  val appModule = module {
      single { UserRepository(get()) }
  }
  
  //注解型框架
  @Module
  class AppModule {
      @Provides
      @Singleton
      fun provideUserRepository(database: Database): UserRepository {
          return UserRepository(database)
      }
  }
  ```

* **运行时灵活性**

	在运行时更改配置：
	

  ```kotlin
  // Development
  startKoin {
      modules(module {
          single<ApiClient> { MockApiClient() }
      })
  }
  
  // Production
  startKoin {
      modules(module {
          single<ApiClient> { RealApiClient() }
      })
  }
  ```



## 二、基础入门

### 构造函数注入（推荐）

依赖项通过构造函数传递：

```kotlin
class UserRepository(
    private val database: Database,
    private val apiClient: ApiClient
) {
    fun getUser(id: String): User {
        return database.query(id) ?: apiClient.fetchUser(id)
    }
}

//使用koin
val appModule = module {
    single { Database() }
    single { ApiClient() }
    single { UserRepository(get(), get()) }
}
```

构造函数注入是Koin中**首选的方法**。它使你的代码可测试，且在单元测试中不需要Koin。

### 字段注入

当你无法控制其构造的Android框架类（Activity、Fragment、Service）

当无法进行构造函数注入时：

```kotlin
class UserActivity : AppCompatActivity() {
    // Lazy injection - instance created when first accessed
    private val viewModel: UserViewModel by viewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        viewModel.loadUser()  // ViewModel instance created here
    }
}

//懒注入
val presenter: Presenter by inject()

// 饿汉注入
val presenter: Presenter = get()
```

### 方法注入(不常见)

```kotlin
class ReportGenerator {
    fun generateReport(data: DataSource) {
        // Use data to generate report
    }
}
```

何时使用：略过。

- 可选依赖项
- 对象生命周期内发生变化的依赖项
- 回调模式

### Koin的DSL 和 自动装配DSL

通过简洁的 DSL 提供自动化依赖解析：

```kotlin
// 1. 一次定义
//传统DSL：
val appModule = module {
    single { Database() }
    single { ApiClient() }
    single { UserRepository(get(), get()) }  // Manually call get() for each dependency
    single { AuthRepository(get(), get()) }
    single { UserService(get(), get()) }
    viewModel { UserViewModel(get()) }
}

//或者： 更推荐更优 自动装配DSL，如果是构造函数的类，就可以通过自动装配函数解决
val appModule = module {
    singleOf(::Database)
    singleOf(::ApiClient)
    singleOf(::UserRepository) // Koin 自动搞定，构造函数的参数!
    singleOf(::AuthRepository)
    singleOf(::UserService)
    viewModelOf(::UserViewModel)
}

// 2. Application一次启动
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            modules(appModule)
        }
    }
}

// 3. 到处使用 - Koin管理整个依赖图
class MainActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModel()
    // OK! Koin creates UserViewModel and all its dependencies
}
```

优势：

> 声明式依赖配置
>
> 自动依赖解析
>
> 生命周期管理（单例、工厂、作用域）
>
> 类型安全注入
>
> 易于测试和模块替换

### 可用的自动装配函数

| 自动装配函数                 | 等价于经典做法                     | 描述                               |
| :--------------------------- | :--------------------------------- | :--------------------------------- |
| `singleOf(::MyClass)`        | `single { MyClass(get(), ...) }`   | 带自动装配的**单例**模式           |
| `factoryOf(::MyClass)`       | `factory { MyClass(get(), ...) }`  | 通过自动装配，**每次都创建新对象** |
| `scopedOf(::MyClass)`        | `scoped { MyClass(get(), ...) }`   | 带自动装配的**作用域对象**         |
| `viewModelOf(::MyViewModel)` | `viewModel { MyViewModel(get()) }` | 带有自动装配的**ViewModel**        |

### 完整示例

```kotlin
// 使用构造函数注入的类
class Database
class ApiClient
class UserRepository(val database: Database, val apiClient: ApiClient)
class AuthRepository(val database: Database, val apiClient: ApiClient)
class UserService(val userRepo: UserRepository, val authRepo: AuthRepository)
class UserViewModel(val userService: UserService) : ViewModel()

//完整定义自动装配模块
val appModule = module {
    singleOf(::Database)
    singleOf(::ApiClient)
    singleOf(::UserRepository)
    singleOf(::AuthRepository)
    singleOf(::UserService)
    viewModelOf(::UserViewModel)
}
```

#### 带参数

自动装配DSL **完全支持**用于运行时参数的`parametersOf()`：

```kotlin
class UserPresenter(
    val userId: String,                    // 需要传入的参数
    val repository: UserRepository          // 依赖注入对象
)

val appModule = module {
    factoryOf(::UserPresenter)  // 自动处理了依赖注入和参数!
}

// 使用：参数被自动匹配到构造函数了
class UserActivity : AppCompatActivity() {
    val presenter: UserPresenter by inject { parametersOf("user123") }
    // 或者
    val presenter: UserPresenter = get { parametersOf("user123") }
}
```

> 自动装配DSL会检索`解析参数栈`，并通过`parametersOf()`传递的任意参数，自动将它们与构造函数参数匹配。

#### 自动装配DSL的适用范围

| 适用                                       | 自动装配DSL<br />支持与否                        |
| ------------------------------------------ | ------------------------------------------------ |
| 你的类使用**构造函数注入**                 | ✅ (::XXXClass) 等价于xxx { MyClass(get(), ...) } |
| 你可以通过`get()`或`by inject()`获取依赖项 | ✅                                                |
| 使用`parametersOf()`传递运行时参数         | ✅                                                |
| 对于可选依赖项，你需要`getOrNull()`        | ❌                                                |
| 自定义初始化逻辑                           | ❌                                                |
| 显式绑定接口（不过你可以与`bind`结合使用） | ❌                                                |

自动装配DSL仅适用于：

- ✅ `get()` - 立即检索 ✅ by inject() - 延迟委托 ✅ parametersOf() - 运行时参数
- ❌`getOrNull()` - 不支持（请改用传统DSL） 混合使用传统DSL和自动装配DSL您可以在同一模块中混合使用这两种风格



混合写法：

```kotlin
val appModule = module {
    // 简单的自动装配
    singleOf(::Database)
    factoryOf(::ApiClient)

    // 自动装配接口bind
    singleOf(::UserRepositoryImpl) bind UserRepository::class

    // 经典DSL 自定义一些逻辑
    single {
        OkHttpClient.Builder()
            .connectTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    // 经典DSL 可选依赖项
    factory {
        MyClass(
            required = get(),
            optional = getOrNull() // 无法自动依赖，？对象
        )
    }
}
```

对于使用构造函数注入的类，优先使用**自动装配DSL**（`singleOf`、`factoryOf`、`viewModelOf`、`scopedOf`）。它更简洁，减少了样板代码，并完全支持参数。仅当需要`getOrNull()`或自定义初始化逻辑时，才使用经典DSL。



### 服务定位器与依赖注入

（Service Locator vs Dependency Injection）

理解这些模式之间的区别很重要，因为Koin同时支持这两种模式。

#### 服务定位器模式

一个集中式注册表，您可以在其中主动请求依赖项：

当你使用`get()`或`by inject()`时：

```kotlin
class UserService : KoinComponent {
    // 请求依赖
    private val repository: UserRepository by inject()
    private val logger: Logger by inject()
  	private val service: MyService = get() 

    fun loadUser(id: String) {
        return repository.findUser(id)
    }
}
```

组件从容器中“拉取”依赖项； 构造函数中看不到依赖项；

使用`get()`或`inject()`来检索实例。

#### 依赖注入模式

依赖项由外部提供：

```kotlin
class UserService(
    private val repository: UserRepository,
    private val logger: Logger
) {
    fun loadUser(id: String) {
        return repository.findUser(id)
    }
}

val appModule = module {
    single { UserRepository(get(), get()) } // DI - Koin依赖注入
}

class UserRepository(
    private val database: Database, // 被koin注入
    private val apiClient: ApiClient
)
```

依赖项是被“推入”到组件中；构造函数中明确的依赖项；

更易于测试（无需框架）。

| 方面           | Service Locator 服务定位器 | Dependency Injection 依赖注入   |
| :------------- | :------------------------- | :------------------------------ |
| 依赖可见性     | 隐式依赖（类内部）         | 显式依赖（构造函数/属性）       |
| 测试           | 需要框架或模拟             | 易于传递的测试替身              |
| 耦合性         | 更紧密（取决于容器）       | 较松（依赖于接口）              |
| 在Koin中的使用 | `get()`，`by inject()`     | 模块中带有`get()`的构造函数参数 |
| 最适用于       | Android框架类              | 业务逻辑、仓库、服务            |

* **业务逻辑类首选构造函数注入**

```kotlin
// ✅ - koin可测试不用框架
class UserViewModel(
    private val userService: UserService
) : ViewModel() {
    // ...
}

val appModule = module {
    viewModel { UserViewModel(get()) }
}
```

* **仅在必要时使用服务定位器**（Android框架类）：

```kotlin
// 可接受： - Activity是android系统创建的对象
class UserActivity : AppCompatActivity() {
    private val viewModel: UserViewModel by viewModel()
}
```

* **避免在业务逻辑中使用`KoinComponent`**：

```kotlin
// ⚠️ - 难以测试，隐式依赖
class UserService : KoinComponent {
    private val repository: UserRepository = get()
}

// ✅ - 显式依赖，易于测试
class UserService(
    private val repository: UserRepository
)
```

虽然 Koin 为了灵活性（对 Android 而言尤其重要）支持这两种模式，但它强烈建议使用构造函数注入，以提高可测试性并使代码更清晰。




## 三、koin详细指南

Koin是一个面向Kotlin开发者的实用且轻量级的依赖注入框架。借助Kotlin语言的强大能力，Koin提供了一种领域特定语言（DSL）来帮助你描述应用程序的依赖注入容器。通过其Kotlin DSL，Koin提供了一个智能的函数式API，无需注解处理或代码生成就能准备好依赖注入。

Koin的DSL有两个主要部分：

**应用DSL** - 配置Koin容器本身（日志、模块、属性）

**模块DSL** - 声明组件及其依赖项



一个完整demo：

```kotlin
// 数据层
class ApiClient
class Database
class UserRepository(val api: ApiClient, val db: Database)

// 领域层
class GetUserUseCase(val repository: UserRepository)

// 表示层
class UserViewModel(val useCase: GetUserUseCase)

// 定义模块
val dataModule = module {
    single { ApiClient() }
    single { Database() }
    single { UserRepository(get(), get()) }
}

val domainModule = module {
    factory { GetUserUseCase(get()) }
}

val presentationModule = module {
    factory { UserViewModel(get()) }
}

// 入口申明
fun main() {
    startKoin {
        logger(Level.INFO)
      	// 多模块配置
        modules(dataModule, domainModule, presentationModule)
        // 加载配置
        environmentProperties()
        // 饥饿单例
        createEagerInstances()
    }
}
```



### Application DSL

一个`KoinApplication`实例代表您配置好的Koin容器。这使您可以设置日志、加载属性和注册模块。

在两种方法中选择：

* **`koinApplication { }`** 

  创建一个`KoinApplication`独立单例 （可以先跳过学习）

  一个可以直接控制的隔离实例，可以用作测试，用作多context应用

* **`startKoin { }`** 

  创建一个`KoinApplication`并将其注册到`GlobalContext`中

  在`GlobalContext`中注册容器，使其可以通过`KoinComponent`、`by inject()`和其他全局API访问

```kotlin
// 独立单例 (用于测试或者自定义内容)
val koinApp = koinApplication {
    modules(myModule)
}

// 全局实例（应用程序的标准方法） 常规做法，到处引用的根源
startKoin {
    logger()
    modules(myModule)
}
```

#### 配置函数

在`koinApplication`或`startKoin`中，你可以使用：

- `logger()` - 设置日志级别和日志记录器实现（默认：`EmptyLogger`）
- `modules()` - 将模块加载到容器中（接受列表或可变参数）
- `properties()` - 加载属性的HashMap
- `fileProperties()` - 从文件加载属性
- `environmentProperties()` - 从操作系统环境变量加载属性
- `createEagerInstances()` - 实例化所有标有`createdAtStart`的定义
- `allowOverride(Boolean)` - 启用/禁用定义覆盖（ 3.1.0默认true）



### 模块 DSL

Koin模块是定义的逻辑分组。它描述了要创建**什么**以及**如何**连接依赖项。

#### 创建模块

```kotlin
val myModule = module {
    // 具体定义在这里
}
```

#### 定义关键字

Koin提供了几个用于声明组件的关键字：

##### 1️⃣基于Lambda的定义

```kotlin
module {
    single { DatabaseHelper() }    // 单例（应用中共享一个实例）
    factory { NetworkRequest() }   // 工厂（每次都创建新实例）
    scoped { UserSession() }        //作用域（每个作用域生命周期一个实例）
}
```

##### 2️⃣自动装备定义（必须有构造函数）

```kotlin
class MyService(val repository: MyRepository)

module {
    singleOf(::MyRepository) //具有构造函数自动装配的单例
    singleOf(::MyService)    //具有构造函数自动装配的单例
  	factoryOf()              //具有构造函数自动装配的工厂
  	scopedOf()               //具有构造函数自动装配的作用域
}
```

> 更多参考《自动装配DSL》章节。

#### 解析与依赖注入

在定义中使用`get()`来解析依赖项：

```kotlin
class Controller(val service: Service)
class Service(val repository: Repository)

module {
    single { Repository() }
    single { Service(get()) }           // 注入Repository
    single { Controller(get()) }        // 注入Service
}
```

#### 绑定类型

##### 单类型绑定

默认情况下，定义绑定到其精确类型：

```kotlin
interface UserRepository
class UserRepositoryImpl : UserRepository

module {
    // 绑定成接口(推荐)
    single<UserRepository> { UserRepositoryImpl() }
    // 这样cast也行
    single { UserRepositoryImpl() as UserRepository }
}
```

##### 附加类型绑定

使用`bind`将一个定义绑定到多种类型：

```kotlin
interface Logger
interface DebugLogger

class ConsoleLogger : Logger, DebugLogger

module {
    // ConsoleLogger 和 Logger都可以
    single { ConsoleLogger() } bind Logger::class

    // 三个类型都可以
    single { ConsoleLogger() } binds arrayOf(
        Logger::class,
        DebugLogger::class
    )
}
```

#### 限定符（命名定义）

使用`named()`来区分同一类型的多个定义：

```kotlin
module {
    single<Database>(named("local")) { LocalDatabase() }
    single<Database>(named("remote")) { RemoteDatabase() }
}

//按限定符检索
val localDb: Database by inject(named("local"))
val remoteDb: Database by inject(named("remote"))
```

限定符可以是：

字符串：`named("qualifier")`

类型：`named<MyType>()`

枚举：`named(MyEnum.VALUE)`

#### 注入参数

将运行时参数传递给定义：

```kotlin
class UserPresenter(val userId: String, val view: UserView)

module {
    factory { (userId: String, view: UserView) ->
        UserPresenter(userId, view)
    }
}

// 解析时提供参数
val presenter: UserPresenter = get { parametersOf("user123", myView) }
```

> 在《注入参数》章节有更多内容。

#### 定义选项

##### 使用 `withOptions`

为一个定义应用多个选项：

```kotlin
module {
    single { MyService(get()) } withOptions {
        named("primary")
        createdAtStart()
        bind<ServiceInterface>()
    }
}
```

可用选项：

- `named()` - 分配限定符
- `bind<Type>()` - 绑定到其他类型
- `binds()` - 绑定到多种类型
- `createdAtStart()` - 在启动时立即创建 （**重点**）
- `override()` - 即使全局覆盖已禁用，也允许覆盖（4.2.0+）
- `onClose { }` - 注册清理回调函数

##### 立即实例化

启动时立即创建实例：

```kotlin
module {
    // 参数定义
    single(createdAtStart = true) { CacheManager() }

    // 或者 通过withOptions
    single { CacheManager() } withOptions {
        createdAtStart()
    }
}
```

你也可以标记整个模块：

```kotlin
module(createdAtStart = true) {
    single { ServiceA() }
    single { ServiceB() }
}
```

##### 覆盖

标记特定定义为允许覆盖：todo 什么是override什么作用

```kotlin
startKoin {
    allowOverride(false)  //严格模式
    modules(productionModule, testModule)
}

val productionModule = module {
    single<Service> { ProductionService() }
}

val testModule = module {
    //明确允许覆盖
    single<Service> { MockService() } withOptions {
        override()
    }
}
```

#### 作用域

为限定范围的实例定义逻辑分组：

```kotlin
module {
    scope<MyActivity> {
        scoped { ActivityPresenter() }
    }
}
```

> 在《作用域》章节中了解更多信息



#### 生命周期回调

使用`onClose`注册清理逻辑： todo 到底是什么时候清理

```kotlin
module {
    single {
        DatabaseConnection()
    } onClose { connection ->
        connection?.close()
    }
}
```

### 最佳实践

**按层组织** - 为数据层、领域层和表现层创建单独的模块

**使用自动装配DSL** - 为了代码更简洁，优先选择`singleOf(::MyClass)`，而非`single { MyClass(get()) }`

**利用限定符** - 当需要同一类型的多个实现时，请使用`named()`

**显式类型** - 为清晰起见，使用`single<Interface> { Implementation() }`

**模块组合** - 使用`includes()`从较小的模块组合成较大的模块 

​	todo includes还不了解





-----------



## 8、集成

```groovy
implementation(platform("io.insert-koin:koin-bom:$koin_version"))
implementation("io.insert-koin:koin-core")
implementation("io.insert-koin:koin-android")

//implementation("io.insert-koin:koin-core-coroutines")

// Java Compatibility
implementation("io.insert-koin:koin-android-compat")
// Jetpack WorkManager
implementation("io.insert-koin:koin-androidx-workmanager")
// Navigation Graph
implementation("io.insert-koin:koin-androidx-navigation")
// App Startup - Start Koin with AndroidX Startup
implementation("io.insert-koin:koin-androidx-startup")
```

```kotlin
class MainApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        startKoin {
            androidLogger()
            androidContext(this@MainApplication)
            modules(appModule)
        }
    }
}
```

* 其他集成，纯android compose

```groovy
implementation(platform("io.insert-koin:koin-bom:$koin_version"))
implementation("io.insert-koin:koin-androidx-compose")
implementation("io.insert-koin:koin-androidx-compose-navigation")
```

* compose multiplatform

```groovy
implementation(platform("io.insert-koin:koin-bom:$koin_version"))
implementation("io.insert-koin:koin-compose")
implementation("io.insert-koin:koin-compose-viewmodel")
implementation("io.insert-koin:koin-compose-viewmodel-navigation")
```

* nav3(alpha)

```groovy
// Navigation 3 support (alpha)
implementation("io.insert-koin:koin-compose-navigation3")
```

* Ktor

```groovy
implementation(platform("io.insert-koin:koin-bom:$koin_version"))
// Koin for Ktor
implementation("io.insert-koin:koin-ktor")
// SLF4J Logger
implementation("io.insert-koin:koin-logger-slf4j")
```

```kotlin
fun Application.main() {
    install(Koin) {
        slf4jLogger()
        modules(appModule)
    }
}
```

## 九、实践

记录实践中的一些用法。

### GlobalContext

我们通过`get()`/`inject()`直接在生命周期类注入, ViewModel类中可以拿到注入对象。比如:

```kotlin
class XXFragment : Fragment() {
    private val globalDiscovery : IDiscovery = get()
  	...
}
```

但是，不是生命周期类，可以通过：

```kotlin
GlobalContext.get().get<Api>()
```

还可以通过`继承KoinComponent`实现。

当然，最推荐直接构造函数使用。



### 继承KoinComponent

```kotlin
import org.koin.core.component.KoinComponent
import org.koin.core.component.inject
import org.koin.core.component.get

class MyRepository : KoinComponent {
    // 方式1: 使用 inject 延迟初始化
    private val apiService: ApiService by inject()
    
    // 方式2: 使用 get 直接获取
    fun doSomething() {
        val apiService = get<ApiService>()
        // 使用 apiService
    }
}
```

可以作用于object类，就是要注意持有的东西，生命周期跟随单例：

```kotlin
object ResourceManager : KoinComponent {
    // 注意：object 是全局单例，会持有所有注入的依赖
    private val heavyResource: HeavyResource by inject()
    
    fun cleanup() {
        // 需要手动释放资源
        heavyResource.release()
    }
}
```

### 自定义scope生命周期

用来实现自定义生命周期

```kotlin
// 创建自定义 Scope
class MyScope : Scope()

// 定义需要释放的资源
class DatabaseConnection : Closeable {
    override fun close() {
        println("Database connection closed")
    }
}

class NetworkClient : Closeable {
    override fun close() {
        println("Network client closed")
    }
}

// 模块配置
val appModule = module {
    // 在 MyScope 内创建资源
    scope<MyScope> {
        scoped { DatabaseConnection() }
        scoped { NetworkClient() }
    }
}

// 在 Activity 中使用
class MainActivity : AppCompatActivity() {
    private lateinit var myScope: MyScope
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 创建 Scope
        myScope = getKoin().createScope("myActivityScope", MyScope::class)
        
        // 获取资源
        val db = myScope.get<DatabaseConnection>()
        val network = myScope.get<NetworkClient>()
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // 关闭 Scope，自动释放所有 scoped 资源
        myScope.close()
    }
}
```

### ROOM

做成单例。

```kotlin
// 1. 定义数据库模块
val databaseModule = module {
    // 单例模式注入 AppDatabase
    single {
        Room.databaseBuilder(
            androidApplication(),  // 从 Koin 获取 Application Context
            AppDatabase::class.java,
            "app_database.db"
        )
        .fallbackToDestructiveMigration()  // 可选：破坏性迁移
        .build()
    }
    
    // 注入 DAO（也从单例数据库中获取）
    single { get<AppDatabase>().userDao() }
    single { get<AppDatabase>().productDao() }
}

// 2. 启动 Koin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        
        startKoin {
            androidContext(this@MyApp)
            modules(
                databaseModule,
                // 其他模块...
            )
        }
    }
}

// 3. 使用数据库
class UserRepository(
    private val userDao: UserDao
) {
    suspend fun getUsers(): List<User> {
        return userDao.getAll()
    }
}
```







### 多Activity共享ViewModel

```kotlin
// 一个文件搞定
object SharedSessionManager {
    private val viewModelStore = ViewModelStore()
    
    fun getSharedViewModel(owner: ViewModelStoreOwner = Application()): SharedViewModel {
        return ViewModelProvider(viewModelStore) { 
            SharedViewModelFactory() 
        }.get(SharedViewModel::class.java)
    }
    
    class SharedViewModel : ViewModel() {
        val data = MutableLiveData<String>()
        
        override fun onCleared() {
            // 所有 Activity 销毁时调用
            super.onCleared()
        }
    }
}

// 在任何 Activity 中使用
class AnyActivity : AppCompatActivity() {
    private val sharedViewModel = SharedSessionManager.getSharedViewModel()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        sharedViewModel.data.observe(this) { /* 更新 */ }
    }
}
```

或者使用koin官方推荐：

```kotlin
// 使用 Koin AndroidX 扩展
implementation "io.insert-koin:koin-androidx-scope:$koin_version"
implementation "io.insert-koin:koin-androidx-viewmodel:$koin_version"

// 创建共享作用域并绑定到 Activity 生命周期
class SharedActivity : AppCompatActivity() {
    // 创建作用域并绑定到生命周期
    private val sharedScope by activityScope(named("shared"))
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 获取共享数据
        val sharedData = sharedScope.get<SharedData>()
        
        // 当 Activity 销毁时，Koin 会自动检查引用并决定是否关闭
    }
    // 不需要手动 close()
}
```









### 一个好的MVVM设计

```kotlin

val appModule = module {
    // 1. 数据层 - 单例
    single { AppDatabase.getInstance(get()) }
    single { get<AppDatabase>().albumDao() }
    single { get<AppDatabase>().photoDao() }
    

    // 2. Repository 层 - 单例
    single { AlbumRepository(get()) }
    single { PhotoRepository(get()) }
    
    // 3. 网络层 - 单例
    single { Retrofit.Builder().build().create(ApiService::class.java) }
    
    // 4. ViewModel 层 - factory（每个 ViewModel 实例新创建）
    viewModel { MainViewModel(get(), get()) }
    viewModel { (albumId: String) -> DetailViewModel(albumId, get()) }
}
```

### androidApplication()获取context

在 Koin 中，androidApplication() 只能在 Koin 模块中使用，并且只能在 Koin 已经启动并且有 Android 上下文的情况下使用。因此，我们通常会在 Application 类的 onCreate 中启动 Koin，并传递 this（Application）给 Koin。

### KoinComponent & KoinScopeComponent

`KoinComponent` 只是一个**接口**，它让类能够访问 Koin 容器：

```kotlin
// KoinComponent 的简化定义
interface KoinComponent {
    // 默认实现提供了 Koin 实例的访问
    fun getKoin(): Koin = GlobalContext.get()
}

// 实现 KoinComponent 后，你就可以使用：
class MyClass : KoinComponent {
    // 现在可以访问 Koin 功能
    val dependency: MyDependency by inject()
}
```

并不是单例，也不是什么， 仅仅提供了能够`inject()`, `get()`等能力。通常用来标注`object类`，让他们能够注入代码。

`KoinScopeComponent` 是 Koin 中用于**管理带生命周期的依赖注入**的接口。它与 `KoinComponent` 类似，但提供了**作用域（Scope）管理**的能力。**同作用域内的单例**。

```kotlin
// KoinScopeComponent 的定义
interface KoinScopeComponent : KoinComponent {
    val scope: Scope  // 必须提供 Scope 实例
}

// KoinComponent 的定义（对比）
interface KoinComponent {
    fun getKoin(): Koin = GlobalContext.get()
}
```

使用示例：

：

```kotlin
// 1. 创建自定义作用域
class FeatureScope : Scope()

// 2. 模块中定义作用域依赖
val featureModule = module {
    // 作用域限定对象
    scope<FeatureScope> {
        scoped { FeatureRepository() } // 同一个 UserScope 内是单例
        scoped { FeatureViewModel(get()) } // 同一个 UserScope 内是单例
    }
}

// 3. 实现 KoinScopeComponent
class FeatureActivity : AppCompatActivity(), KoinScopeComponent {
    // 提供 Scope 实例
    override val scope: Scope by lazy {
        getKoin().createScope(
            scopeId = "feature_${this.hashCode()}",
            qualifier = FeatureScope::class
        )
    }
    
    // 使用作用域内的依赖
    private val viewModel: FeatureViewModel by scope.inject()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 直接使用注入的 viewModel
        viewModel.loadData()
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // 关闭作用域，释放资源
        scope.close()
    }
}
```

使用 `KoinScopeComponent` 的场景：

1. **有生命周期的对象**：如用户会话、临时任务
2. **需要隔离的数据**：不同用户/页面的数据隔离
3. **资源管理**：需要及时释放的资源
4. **复杂依赖关系**：嵌套的依赖关系

并不一定需要让类去继承它。



### scope

```kotlin
// 没有 KoinScopeComponent
class WithoutComponent {
    // 需要自己管理 Scope
    private lateinit var scope: Scope
    
    fun setupScope() {
        scope = getKoin().createScope("id", MyScope::class)
    }
    
    fun cleanup() {
        scope.close()
    }
}

// 有 KoinScopeComponent
class WithComponent : KoinScopeComponent {
    // 必须提供 scope
    override val scope: Scope = getKoin().createScope("id", MyScope::class)
    
    // 可以直接使用 scope 的属性
    val session: UserSession by scope.inject()
}
```

**什么是 `scope.close()`**

```kotlin
// 当调用 scope.close() 时：
// 1. ✅ 作用域内的所有 scoped 实例被释放
// 2. ✅ 作用域不能再使用
// 3. ✅ 其他使用同一作用域的地方都会出错

// ❌ 危险的代码：多个 Activity 共享同一个作用域
object GlobalScope {
    val sharedScope = getKoin().createScope("shared", SharedScope::class)
}

class ActivityA : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 使用共享作用域
        val data = GlobalScope.sharedScope.get<SharedData>()
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // 危险！第一个销毁的 Activity 就关闭了作用域
        GlobalScope.sharedScope.close()
    }
}

class ActivityB : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // ❌ 如果 ActivityA 先销毁了，这里会崩溃
        val data = GlobalScope.sharedScope.get<SharedData>()
    }
}


// 创建作用域
val myScope = koin.createScope("scope_id", named<MyType>())
// 获取已创建的作用域 //关闭作用域后，无法再从中注入实例，否则会抛出异常。
val existingScope = koin.getScope("scope_id")
// 创建或获取作用域
val scope = koin.getOrCreateScope("scope_id", named<MyType>())
 
// 关闭作用域
scope.close()


//重点：Koin提供KoinScopeComponent接口，方便将作用域关联到类：
class A : KoinScopeComponent {
    override val scope: Scope by lazy { createScope(this) }
    
    // 从作用域注入依赖
    val b: B by inject()
    
    fun closeScope() {
        scope.close() // 记得关闭作用域
    }
}
 
// 模块定义
module {
    scope<A> {
        scoped { B() }
    }
}
```

### android各种作用域

Android组件作用域API
Koin提供了专门的Android作用域API，如docs/reference/koin-android/scope.md所述，主要包括：

activityScope()：创建与Activity生命周期绑定的作用域
activityRetainedScope()：创建保留的作用域（基于ViewModel生命周期）
fragmentScope()：创建与Fragment生命周期绑定的作用域

activityScope:

```kotlin
// 模块定义
val appModule = module {
    activityScope {
        scoped { UserPresenter(get()) }
        scoped { UserAdapter(get()) }
    }
}
 
// Activity中使用
class UserActivity : AppCompatActivity(), AndroidScopeComponent {
    override val scope: Scope by activityScope()
    
    private val presenter: UserPresenter by inject()
    private val adapter: UserAdapter by inject()
    
    override fun onDestroy() {
        super.onDestroy()
        // 作用域会自动关闭，无需手动调用
    }
}
```

fragmentScope:

```kotlin
class UserFragment : Fragment(), AndroidScopeComponent {
    override val scope: Scope by fragmentScope()
    
    // 可以注入Activity作用域中定义的依赖
    private val presenter: UserPresenter by inject()
}
```

viewModelScope

ViewModel作用域是一种特殊的作用域，需注意ViewModel不能直接访问Activity或Fragment的作用域，以避免内存泄漏.

```kotlin
// 模块定义
module {
    viewModelScope {
        scoped { UserRepository() }
    }
    
    viewModelOf(::UserViewModel)
}
 
// ViewModel中使用
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    // ...
}
 
// 激活ViewModel作用域工厂
startKoin {
    options(
        viewModelScopeFactory()
    )
    modules(appModule)
}
```

作用域高级特性
作用域链接
作用域链接允许一个作用域访问另一个作用域中的依赖，实现跨作用域共享实例：

// 创建两个作用域
val scopeA = koin.createScope("scopeA", named("A"))
val scopeB = koin.createScope("scopeB", named("B"))

// 链接作用域
scopeA.linkTo(scopeB)

// 现在scopeA可以访问scopeB中的依赖
val dependency = scopeA.get<BDependency>()
kotlin
运行
作用域原型
Koin 4.1.0引入了作用域原型(Scope Archetypes)，可以为一类组件声明通用作用域：

module {
    // 为所有Activity声明通用作用域
    activityScope {
        scoped { AnalyticsTracker() }
    }
    
    // 为所有Fragment声明通用作用域
    fragmentScope {
        scoped { ImageLoader() }
    }
}
kotlin
运行

获取作用域源对象
在作用域定义中，可以通过getSource()获取作用域的源对象：

class A
class B(val a: A)

module {
    scope<A> {
        scoped { B(getSource()) } // 直接获取作用域源对象A
    }
}

// 使用
val a = koin.get<A>()
val scope = a.createScope()
val b = scope.get<B>() // b.a == a
kotlin
运行