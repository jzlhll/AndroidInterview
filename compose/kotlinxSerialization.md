替代Gson、fastJson等传统java的json解析工具。抛弃传统的反射类解析字段，利用kotlin的`inline+reified`特性和android可以预编译的特点，在编译阶段的时候，还原最后的类型，来实现的json序列化与反序列化。

性能效率：不做评价，2025年了，手机性能的提升和移动场景的数据量，json工具之间的时间消耗并非我们取舍的关键点。

日常使用难度：与gson相比，最重要是需要更多标注类的注解；序列化和反序列化相对简单；

优点：不用加混淆规则。

### 集成

```groovy
// 根目录 build.gradle.kts
plugins {
    kotlin("multiplatform") version "2.2.0" // 如果需要多平台支持
    kotlin("jvm") version "1.9.10" // 如果是 JVM 项目
}

// 直接声明依赖版本
val kotlinxSerializationJsonVersion = "1.9.0"
val kotlinReflectVersion = "1.9.10"

// 添加插件
plugins {
    id("org.jetbrains.kotlin.plugin.serialization") version "1.9.10"
}

// 模块目录 build.gradle.kts
plugins {
    kotlin("jvm") // 或 kotlin("multiplatform") 如果是多平台项目
    id("org.jetbrains.kotlin.plugin.serialization") version "1.9.10"
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:$kotlinxSerializationJsonVersion")
    implementation("org.jetbrains.kotlin:kotlin-reflect:$kotlinReflectVersion")
}
```

### 基本使用

* 定义全局单例，附带常用参数：

```kotlin
//单例
object Globals {
    kson = Json {
      ignoreUnknownKeys = true      // 后端多给字段也不报错
      encodeDefaults = true         // 输出默认值；关掉可减小体积
      prettyPrint = false           // 日常关；调试可开
      isLenient = true              // 宽松模式，允许非标准 JSON
      explicitNulls = false         // null 字段不主动输出
      //allowTrailingComma = true
      // classDiscriminator = "type" // 多态时的类型标记字段名（见下）
      // namingStrategy = JsonNamingStrategy.SnakeCase // 字段命名转换（版本要求较新）
   }
}
```

* 定义Bean类：

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable
import kotlinx.serialization.Transient

@Serializable
class User {
    var name: String = ""
    var age: Int = 0
    var email: String? = null //空安全
    var isActive: Boolean = true
    @SerializedName("created_at")
    var createdAt: String = "",
  	val sex: String = "male", //默认值
}

val user = User("Alice", 30, "alice@example.com")
// 序列化
val jsonString = Json.encodeToString(user)

// 反序列化  
val userFromJson = Json.decodeFromString<User>(jsonString)
// 输出: {"name":"Alice","age":30,"email":"alice@example.com","isActive":true,"created_at":"..."}
```

#### 常用注解

* `@Serializable`

  类名必须标记。否则运行报错。后续封装Util的泛型传入也必须是注解的类。

* `@SerialName`

  作用：JSON key映射，不论序列化或者反序列化都会改成注解后的字段。

* `@JsonNames`

  ```kotlin
  // 用于为同一字段提供多个可能的JSON键名
  @Serializable
  data class User(
      @JsonNames("user_name", "username", "userName")
      val name: String,
      
      @JsonNames("user_email", "email_address")
      val email: String? = null
  )
  
  // 使用示例
  val json1 = """{"user_name": "Alice", "user_email": "alice@test.com"}"""
  val json2 = """{"username": "Bob", "email_address": "bob@test.com"}"""
  kson.decodeFromString<User>(json1) ✅
  ...
  val bean1 = User("Alice", "alice@test.com")
  kson.encodeToString(bean1)
  //得到： {"name":"Alice", "email": "alice@test.com"}
  ```

  能够识别任意JsonNames，从json字符串，反序列化为对象；

  序列化，还是转成现有字段；并且`@SerialName`优先级更高。

* `@Transient` 

  忽略字段。注意是`kotlinx.serialization.Transient`。不论序列化或者反序列化，都当做不存在一样。

* `@Contextual`

​	进阶`自定义解析器`章节。

####  特性

* **可空字段**： 其中定义了`email:String?`字段，就是空安全，允许，jsonString不存在该内容，反序列化后的对象该变量就是null

* **默认值**：`sex ="male"`,  如果jsonString不存在该内容，反序列化后的对象该变量就是male。

  需要给`json{}`添加参数：

  ```kotlin
  json {
  	//...
  	encodeDefaults = true
  }	
  ```

#### 泛型API

我们一般遇到的是这种case：

```kotlin
@Serializable
sealed class BaseBean {
    abstract val code: String
    abstract val message: String?
    abstract val status: Boolean
}

@Serializable
@SerialName("result")
data class ResultBean<T>(
    override val code: String,
    override val message: String?,
    override val status: Boolean,
    val data: T? = null
) : BaseBean()
```

直接使用，只要你的嵌套data的Class使用注解即可。



#### 类似org.json.JSONObject的使用

```kotlin
//org.json.JSONObject
var jsonObject = JSONObject(result)
val code = jsonObject.optString("code")

//Kson使用方式
val element = Json.parseToJsonElement("""{"age":18,"data":["x","y"]}""")
println(element.jsonObject["age"]?.jsonPrimitive?.int) // 18

val element = JsonParser.parseString("""{"age":18,"data":["x","y"]}""")
println(element.asJsonObject["age"].asInt) // 18

val obj = buildJsonObject { 
    put("name", "Xiao ming") 
    put("classes", buildJsonArray { 
            add("Math")
            add("Chinese") 
            } ) 
    }
```

#### 配合Retrofit、Ktor

```kotlin
object NetworkClient {
    private val json = Json {
        ignoreUnknownKeys = true
        isLenient = true
    }
    
    val instance: Retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(json.asConverterFactory("application/json".toMediaType()))
            .build()
    }
}


val client = HttpClient(OkHttp) {
    install(ContentNegotiation) { json(Json {
        ignoreUnknownKeys = true
        // classDiscriminator = "type"
    }) }
}
```



### 进阶使用

#### 自定义解析器（必须掌握）

对于某些字段，我们需要自定义非普通类型，又无法自定义Class，比如系统的Cookie类，UUID类，Color类等，必须自定义解析器后，注解`@Serializable(with = xxx::class)`标记。

* 方法一：

  先编写`KSerializer<XXX>` 转换器比如UUIDAsString。

  然后给需要转化的字段使用`@Serializable(with=UUIDAsString::class)`

```kotlin
object UUIDAsString : KSerializer<UUID> {
    override val descriptor: SerialDescriptor =
        PrimitiveSerialDescriptor("UUIDAsString", PrimitiveKind.STRING)
 
    override fun serialize(encoder: Encoder, value: UUID) {
        encoder.encodeString(value.toString())
    }
    override fun deserialize(decoder: Decoder): UUID =
        UUID.fromString(decoder.decodeString())
}
 
@Serializable
data class WithUuid(
    @Serializable(with = UUIDAsString::class) //方法一
    val id: UUID
)

@Serializable
data class WithUuid(
    @Contextual //方法二
    val id: UUID
)
val module = SerializersModule {
    contextual(UriSerializer)
}
val json = Json { serializersModule = module }
```

* 方法二：

先编写`KSerializer<XXX>` 转换器，如上。

然后将它在`Json{}`申明的时候，注册；

字段使用`@Contextual`注解标注。



#### sealed class（功能强大，后补学习）

```kotlin
@Serializable
sealed class Message {
    @Serializable @SerialName("text")
    data class Text(val content: String, val sender: String) : Message()
    
    @Serializable @SerialName("image")
    data class Image(val url: String, val width: Int, val height: Int) : Message()
    
    @Serializable @SerialName("video")
    data class Video(val url: String, val duration: Int, val thumbnail: String) : Message()
}

// 测试代码
fun testSealedClass() {
    val json = Json { 
        classDiscriminator = "type"  // 指定类型标识字段名
        prettyPrint = true
    }
    
    // 创建不同子类的实例
    val messages = listOf(
        Message.Text("Hello World", "Alice"),
        Message.Image("https://example.com/img.jpg", 800, 600),
        Message.Video("https://example.com/video.mp4", 120, "thumb.jpg")
    )
    
    // 序列化
    messages.forEach { msg ->
        val jsonStr = json.encodeToString(msg)
        println("序列化结果:\n$jsonStr")
        
        // 反序列化
        val decoded = json.decodeFromString<Message>(jsonStr)
        println("反序列化结果: $decoded\n")
    }
    
    // 测试JSON示例
    val jsonText = """{"type":"text","content":"Hi there","sender":"Bob"}"""
    val jsonImage = """{"type":"image","url":"test.png","width":100,"height":100}"""
    
    val textMsg = json.decodeFromString<Message>(jsonText)
    val imageMsg = json.decodeFromString<Message>(jsonImage)
    
    // 使用when处理不同类型
    when (textMsg) {
        is Message.Text -> println("收到文本消息: ${textMsg.content}")
        is Message.Image -> println("收到图片: ${textMsg.url}")
        is Message.Video -> println("收到视频: ${textMsg.url}")
    }
}

```

```
序列化结果:
{
    "type": "text",
    "content": "Hello World",
    "sender": "Alice"
}
反序列化结果: Text(content=Hello World, sender=Alice)

序列化结果:
{
    "type": "image",
    "url": "https://example.com/img.jpg",
    "width": 800,
    "height": 600
}
反序列化结果: Image(url=https://example.com/img.jpg, width=800, height=600)
```

如果你们后端，采用的是这样平铺在type同级的字段方式。可以采用sealed class的做法。

如果你们是通过这下设计的嵌套字段：

```kotlin
@Serializable
data class MediaResultBean<T>(
  	val type:String,
    val data: T? = null
)

@Serializable
data class Image(val url: String, val width: Int, val height: Int)
@Serializable
data class Video(val url: String, val duration: Int, val thumbnail: String)
```

无需特殊处理。



#### 开放层级 多态SerializersModule 和 polymorphic（完全可跳过）

完全可以跳过，你可以不需要这种设计方案。

```kotlin
import kotlinx.serialization.*

// 开放多态：基类不是sealed class或不是@Serializable
open class Msg(val id: String)

@Serializable
@SerialName("text")
data class TextMsg(
    val content: String,
    override val id: String = ""  // 必须覆盖基类属性
) : Msg(id)

@Serializable
@SerialName("image")
data class ImageMsg(
    val url: String,
    val size: Int,
    override val id: String = ""
) : Msg(id)

// 创建序列化模块
val messageModule = SerializersModule {
    // 注册多态类型
    polymorphic(Msg::class) {
        // 为每个子类指定序列化器和类型名
        subclass(TextMsg::class, TextMsg.serializer())
        subclass(ImageMsg::class, ImageMsg.serializer())
        // 可以动态添加更多子类
    }
    
    // 可以同时注册多个多态基类
    polymorphic(Any::class) {
        // 为Any类型注册序列化器
        subclass(String::class, serializer())
        subclass(Int::class, serializer())
    }
}

// 使用模块创建Json实例
val jsonWithModule = Json {
    serializersModule = messageModule
    classDiscriminator = "msg_type"  // 类型标识字段名
    ignoreUnknownKeys = true
}

// 测试用例
fun testPolymorphic() {
    val messages: List<Msg> = listOf(
        TextMsg("Hello", "1"),
        ImageMsg("pic.jpg", 1024, "2")
    )
    
    // 序列化
    messages.forEach { msg ->
        val json = jsonWithModule.encodeToString(
            PolymorphicSerializer(Msg::class),  // 必须使用多态序列化器
            msg
        )
        println("序列化: $json")
        
        // 反序列化
        val decoded = jsonWithModule.decodeFromString<Msg>(json)
        println("反序列化: ${decoded::class.simpleName}")
    }
    
    // JSON格式示例
    val jsonStr = """
        [
            {"msg_type":"text","id":"1","content":"Hello"},
            {"msg_type":"image","id":"2","url":"pic.jpg","size":1024}
        ]
    """.trimIndent()
    
    val listSerializer = ListSerializer(PolymorphicSerializer(Msg::class))
    val decodedList = jsonWithModule.decodeFromString(listSerializer, jsonStr)
}

// 动态添加子类（运行时注册）
fun dynamicRegistration() {
    // 创建可变的模块
    val mutableModule = SerializersModule { }
    
    // 运行时动态添加
    @Serializable
    @SerialName("audio")
    data class AudioMsg(val url: String, val duration: Int, val id: String = "") : Msg(id)
    
    // 注意：动态添加比较麻烦，通常建议在模块构建时就注册所有类型
}
```



### 封装

**使用AI做了测试25条用例，做了覆盖日常开发常用13种class定义方式，涵盖泛型，嵌套泛型，List，Map，嵌套List等，超过60条各类测试条目。**

最终整合出一个日常通用的Util类做各类解析：

```kotlin
//单例
object Globals {
    kson = Json {
      ignoreUnknownKeys = true      // 后端多给字段也不报错
      encodeDefaults = true         // 输出默认值；关掉可减小体积
      prettyPrint = false           // 日常关；调试可开
      isLenient = true              // 宽松模式，允许非标准 JSON
      explicitNulls = false         // null 字段不主动输出
      //allowTrailingComma = true
      // classDiscriminator = "type" // 多态时的类型标记字段名（见下）
      // namingStrategy = JsonNamingStrategy.SnakeCase // 字段命名转换（版本要求较新）
   }
}

/**
 * 专攻List<Any>, Map<String, Any?>的toString。
 */
@Deprecated("极度受限，使用上位版本[toKsonString]")
fun Any.toKsonStringLimited() : String = Globals.kson.encodeToString(this.toKsonElementLimited())

@Deprecated("极度受限，使用上位版本[toKsonString]")
internal fun Any?.toKsonElementLimited(): JsonElement = when (this) {
    null -> JsonNull
    is JsonElement -> this
    is Number -> JsonPrimitive(this)
    is Boolean -> JsonPrimitive(this)
    is String -> JsonPrimitive(this)
    is Array<*> -> JsonArray(map { it.toKsonElementLimited() })
    is List<*> -> JsonArray(map { it.toKsonElementLimited() })
    is Map<*, *> -> JsonObject(map { it.key.toString() to it.value.toKsonElementLimited() }.toMap())

    //经过测试，其实这里也是要求this这个class，必须是使用@ Serializable注解的。因此尽量使用toJsonStringTyped
    //同时对于嵌套泛型的解析，这是无法支持的。因为这里仅是拿了第一层。
    else -> Globals.kson.encodeToJsonElement(serializer(this::class.createType()), this)
}

/**
 * inline+KSerializer让编译时就确定类型准确无误。
 * 嵌套类型比如：loginBean.toJsonStringTyped(ResultBean.serializer(LoginResponse.serializer()))
 */
inline fun <reified T> T.toKsonString() : String = Globals.kson.encodeToString(this)

inline fun <reified T> String.fromKson() = ignoreError { Globals.kson.decodeFromString<T>(this) }
                                                        
inline fun <T:Any> ignoreError(
    block: () -> T?
): T? {
    return try {
        block.invoke()
    } catch (e: Throwable) {
        e.printStackTrace()
        null
    }
}
```

整个测试下来，我们只需要如上3个函数，就能覆盖99%的工作：

因为全部kson库，toString和fromString，要求的都是`inline+reified`，因为一般情况我们不需要传入解析器。所以：

`toKsonString()`: 用来序列化成字符串；

`fromKson()` : 用来反序列化成T对象，已经做了try-catch；

`toKsonStringLimited()`:只用来将`Map<String, Any>， List<Any>`转成String，这适用于给后端传参的场景。



### 混淆 / 体积
一般不需要额外 keep（因编译期生成并显式引用）。

如果你自定义了 KSerializer 或用了反射式工厂，才可能需要补充 keep。

体积优化：关闭 prettyPrint，必要时 explicitNulls=false，并按需关闭 encodeDefaults。

### 常见坑与排查
忘记加插件：id("org.jetbrains.kotlin.plugin.serialization")。

字段名对不上：用 @SerialName 或开 namingStrategy。

接口偶发多字段：ignoreUnknownKeys = true。

后端大小写不一致：JsonNamingStrategy.SnakeCase 或 @JsonNames(...) 做兼容。

多态无法解析：确认 classDiscriminator 字段名与后端一致；开放多态要注册 SerializersModule。

时间类型：优先 kotlinx-datetime；或写自定义 KSerializer。
