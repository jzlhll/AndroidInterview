android近期新特性和我的一些兼容处理。
### android16
* 新功能： Live Updates 实时活动，类似国产手机实现的点餐、打车通知卡片；

* 新功能： App Functions API 增强 Gemini 助手的应用交互能力。

  https://zhuanlan.zhihu.com/p/1902819098624259608

  > App 提供 AppFunctionService 并在 AndroidManifest.xml 中注册
  > Android 系统检测到新的 AppFunctionService，并使用 App Search 索引其功能元数据
  > 服务消费者（例如 AI 助手）使用 App Search 搜索实现了特定 schema 的 App Functions
  > 消费者获取目标 App Function 的 functionIdentifier
  > 消费者通过 AppFunctionManager 构建包含目标包名和 functionIdentifier 的 ExecuteAppFunctionRequest
  > AppFunctionManager 将请求发送给 Android 系统
  > 系统识别出提供服务的应用，并绑定到其 AppFunctionService。
  > 系统调用提供者应用中 AppFunctionService 的 onExecuteFunction 方法。
  > 提供者应用执行请求的功能，并通过 OutcomeReceiver 回调返回 ExecuteAppFunctionResponse 给系统
  > 系统将响应结果通过消费者应用在调用 executeAppFunction 时提供的 OutcomeReceiver 传递回去

* JobScheduler 配额优化改动；
	背景：Android 16 对 scheduleAtFixedRate进行优化，应用后台时错过的任务，恢复到前台后最多仅执行 1 次。
适配方案
需严格按时执行的任务（如定时同步数据）：替换为 WorkManager，通过 PeriodicWorkRequestBuilder构建任务，设置约束条件，使用 WorkManager.getInstance(context).enqueueUniquePeriodicWork入队执行，实现 Worker类处理具体任务逻辑。
可容忍延迟的任务（如日志上报）：使用 JobScheduler，构建 JobInfo并设置相关参数，通过 jobScheduler.schedule(jobInfo)调度任务，在服务中处理任务，如 LogUploadService中的 onStartJob和 onStopJob方法。

* 最新的ART优化；
  
* 16kb so正式要求兼容。网上太多帖子，这里不做赘述。

* 强制全屏，不要使用windowOptOutEdgeToEdgeEnforcement属性，无边框实现，使用edgeToEdge。

  ```
  <!-- 移除所有以下配置（如果存在） --><item name="windowOptOutEdgeToEdgeEnforcement">true</item>
  ```

  ```kotlin
  //View体系如下： Compose自查
  // 启用无边框模式
  WindowCompat.setDecorFitsSystemWindows(window, false)
  
  // 设置状态栏和导航栏透明
  window.statusBarColor = Color.TRANSPARENT
  window.navigationBarColor = Color.TRANSPARENT
  
  // 设置状态栏文字颜色为深色
  WindowCompat.getInsetsController(window, window.decorView).apply {
      isAppearanceLightStatusBars = true
      isAppearanceLightNavigationBars = true
  }
  
  // 给根布局添加系统内边距
  ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.root)) { view, insets ->
      val systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
      view.setPadding(
          systemBars.left,
          systemBars.top,
          systemBars.right,
          systemBars.bottom
      )
      insets
  }
  ```

  

* 预测性返回，不回调ActVivity`onBackPressed`和`KeyEvent.KEYCODE_BACK`。

  androidx:activity库和OnBackPressedDispatcher是完美支持的；相信大部分开发者已经迁移。不会还是在Activity中接受onBackPressed吧？也可选择暂时屏蔽该影响：方法是在应用的 `AndroidManifest.xml` 文件的 `<application>` 或 `<activity>` 标记中将 [`android:enableOnBackInvokedCallback`](https://developer.android.com/guide/topics/manifest/activity-element?hl=zh-cn#enableOnBackInvokedCallback) 属性设置为 `false`。

  适配方案：

  ```kotlin
  private val invokedBack:Any? = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
      OnBackInvokedCallback {
          finish()
      }
  } else {
      null
  }
  
  //onCreate函数
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
      requireActivity().onBackInvokedDispatcher.registerOnBackInvokedCallback(OnBackInvokedDispatcher.PRIORITY_DEFAULT, invokedBack as OnBackInvokedCallback)
  } else {
      requireActivity().onBackPressedDispatcher.addCallback(viewLifecycleOwner, object: OnBackPressedCallback(true) {
          override fun handleOnBackPressed() {
              finish()
          }
      })
  }
  
  //onDestroy函数
  if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
      requireActivity().onBackInvokedDispatcher.registerOnBackInvokedCallback(OnBackInvokedDispatcher.PRIORITY_DEFAULT, invokedBack as OnBackInvokedCallback)
  }
  ```

  

* 开始忽略屏幕方向限定

	对于以 Android 16（API 级别 36）为目标平台的应用，Android 16 包含对系统管理屏幕方向、尺寸调整能力和宽高比限制的方式的变更。**在最小宽度大于或等于 600dp 的显示屏上，这些限制不再适用。**

此选择停用是临时性的，在未来的 Android 版本中以 API 级别 37 为目标平台时，此选择停用将不再适用。也就是说，对于以 API 级别 37 为目标的应用，在至少为 `sw600dp` 的显示屏上，系统会忽略屏幕方向、尺寸调整能力和宽高比限制。继续苟着吧。

```xml
<application ...>
  <property android:name="android.window.PROPERTY_COMPAT_ALLOW_RESTRICTED_RESIZABILITY" android:value="true" />
</application>
```



### android15

* 16kb so要求兼容。网上太多帖子，这里不做赘述。

* 投屏也有状态栏提醒

* 后台访问网络会被限制抛出`UnknownHostException`或者`IOException`。在离开有效的进程生命周期时取消。如果您非常重视即使用户离开应用也要发出网络请求，请考虑使用 [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager?hl=zh-cn) 调度网络请求，或使用[前台服务](https://developer.android.com/develop/background-work/services/fgs?hl=zh-cn)继续执行对用户可见的任务。

* 前台服务变更，之前有申明前台服务的类型，现在15添加了dataSync/mediaProcessing类型超时行为，限制运行一天6小时。

  其他类型还有dataSync/mediaPlayback/camera/phoneCall/mediaProjection/microphone等。

* android核心库支持LTS openJDK，即jdk17。

* 15开始安全TLS1.0和1.1彻底不允许使用。

* 限制后台启动Activity。针对的就是PendingIntent。[针对从后台启动 activity 的限制  | App architecture  | Android Developers](https://developer.android.com/guide/components/activities/background-starts?hl=zh-cn)

* edge-to-edge默认开启。

### android14

* 蓝牙相关：在调用 `BluetoothAdapter` [`getProfileConnectionState()`](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter?hl=zh-cn#getProfileConnectionState(int)) 方法时，Android 14 会强制执行 [`BLUETOOTH_CONNECT`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#BLUETOOTH_CONNECT) 权限。

* 广播接收器注册的时候，需要指定EXPORTED标志。

  兼容方案如下：

  ```kotlin
  fun Context.registerReceiverFix(receiver: BroadcastReceiver, filter: IntentFilter, receiverSystemOrOtherApp:Boolean = true) {
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
          registerReceiver(
              receiver, filter,
              if (receiverSystemOrOtherApp) Context.RECEIVER_EXPORTED else Context.RECEIVER_NOT_EXPORTED
          )
      } else {
          registerReceiver(receiver, filter)
      }
  }
  ```

* 前台服务类型，必须写出来。

* JobScheduler会触发ANR。

  此 ANR 可能是由以下 2 种情况造成的： 1.有工作阻塞主线程，阻止回调 `onStartJob` 或者`onStopJob`在预期时间内执行并完成。 2. 开发者在 JobScheduler 中运行阻塞工作 回调 `onStartJob` 或 `onStopJob`，阻止从 在预期的时限内完成。

  还有调用setRequiredNetworkType, setRequireedNetwork，需要申明网络权限。否则报错。

  推荐使用WorkManager。

* MediaProjection需要用户同意

  您的应用必须在每次捕获会话之前征求用户同意。单次捕获会话是对 `MediaProjection#createVirtualDisplay` 的单次调用，并且每个 `MediaProjection` 实例只能使用一次。

* 照片和视频的部分访问权限。

  基于android13的细分权限，又追加了一个`READ_MEDIA_VISUAL_USER_SELECTED`。

    >  这里推荐不是做图库类似强需求的应用，统统采用照片选择器picker，以提供一致的图片和视频选择体验，同时增强用户隐私保护，而无需请求任何存储权限。这里我实名五星好评。代码参考：AndroidComponts，NewPhotoPickerFragment以及相关的调用逻辑。

* 隐式Intent会报错。

  ```java
  //错误：
  context.startActivity(Intent("com.example.action.APP_ACTION"))
  
  
  //正确：
  val explicitIntent =
          Intent("com.example.action.APP_ACTION")
  explicitIntent.apply {
      package = context.packageName
  }
  context.startActivity(explicitIntent)  
  ```

* 通知以前设置`Notification.Builder.setOngoing(true)`通知是不可关闭的。

  现在android14，也可以划掉了。

* 默认拒绝设定精确的闹钟。即`SCHEDULE_EXACT_ALARM`权限。换句话说，就是不能动态申请权限了。需要按照如下。

  ```kotlin
  if (!alarmManager.canScheduleExactAlarms()) {
      //弹窗确认引导跳转
      ...
      startActivity(Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM))
  }
  ```

* 从 Android 14 开始，当您的应用调用 `killBackgroundProcesses()`时，该 API 只能终止您自己应用的后台进程。

* BLE开发，蓝牙GATT MTU默认517。

  曾经导致过我公司的app bug，我还给google提了issue。[Gatt requestMtu onMtuChanged](https://issuetracker.google.com/issues/310471171)

  其实google的变更写的不是很清楚。

  主要原因如下：

  ```kotlin
  class InnerGattCallback extends BluetoothGattCallback 
  
      public void onDescriptorWrite(BluetoothGatt gatt, BluetoothGattDescriptor descriptor, int status) {
          boolean success = gatt.requestMtu(mtu) //这里的mtu，是blufi框架默认值251
      }
  
      public void onMtuChanged(BluetoothGatt gatt, int mtu, int status) {
          //回调协商好的mtu就是251
         if (status == BluetoothGatt.GATT_SUCCESS) {
         }
      }
  }
  ```

  blufi的通信框架的逻辑为：

  1. 嵌入式和android端会约定分包大小，这是前提，251；

  2. onDescriptorWrite中调用gatt.requestMtu(251)；

  3. 回调onMtuChanged得到也是251，mBlufiMTU=251，证明协商好了，blufi就用251作为分包size；
  4. 后续发包就按照这个size做分包，如下逻辑所示：

  ```java
  private boolean postContainData(boolean encrypt, boolean checksum, boolean requireAck, int type, byte[] data)
              throws InterruptedException {
      	//mPackageLengthLimit是默认-1；如果mtu协商失败就定为20。协商成功就使用mBlufiMTU
      
          int pkgLengthLimit = mPackageLengthLimit > 0 ? mPackageLengthLimit :
                  (mBlufiMTU > 0 ? mBlufiMTU : DEFAULT_PACKAGE_LENGTH);
          int postDataLengthLimit = pkgLengthLimit - PACKAGE_HEADER_LENGTH;
          byte[] dataBuf = new byte[postDataLengthLimit];
          while (true) {
              //...
              //分包发送逻辑
          }
  ```

  `requestMtu()`函数描述：

  > 请求用于给定连接的MTU大小。
  >
  > 蓝牙通信，执行无响写入请求操作时，发送的数据被截断为MTU大小。
  >
  > 那么， 我们可以使用此函数请求更大的MTU大小，以便能够一次发送更多数据。
  >
  > 同时，onMtuChanged()回调将会告诉我们这个动作是否成功。
  >
  > 返回值：true表示，此操作是否成功。

	这里的理解会出现偏差。

	我们一直认为`requestMtu()`请求多少，`onMtuChanged()`就应该返回多少。比如在android13上，我请求251，`onMtuChanged`就会返回251。

但是在android14的加持下，`requestMtu(251)`会被默认修改成`requestMtu(517)`调用下去，返回true表示操作成功。同时`onMtuChanged()`也会给出设置成功的响应，但是返回的mtu变成了500，这个是你们嵌入式开发的BLE设备返回的值不一定是我这里的500。而且推测返回了500，并不代表他固件就是按照500分包的，而继续按照以前约定的251去分包。
而上面说的，blufi标准流程是按照`onMtuChanged()`返回的mtu来当做分包依据。

**那么， 再兼容android14的时候，怎么处理呢？**
**以你requestMtu的值为准，不要以`onMtuChanged()`返回值为准。**


### android13

* 通知的运行时权限`POST_NOTIFICATIONS`。

  简单来讲，就是你必须动态申请他并授予，才能发送Notification。

  ```kotlin
  <uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
  
  
  class MainActivity {
      private val requestPermissionLauncher = registerForActivityResult(
          ActivityResultContracts.RequestPermission()
      ) { isGranted ->
          if (isGranted) {
              showToast("通知权限已授予")
              // 这里可以开始发送通知
          } else {
              showToast("通知权限被拒绝")
          }
      }
  
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
  
          checkAndRequestNotificationPermission()
      }
  
      private fun checkAndRequestNotificationPermission() {
          val isCanNotify = (Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU
                  || ContextCompat.checkSelfPermission(
              this,
              Manifest.permission.POST_NOTIFICATIONS
          ) == PackageManager.PERMISSION_GRANTED)
  
          if (!isCanNotify) {
              requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
          }
      }
  
      private fun showToast(message: String) {
          Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
      }
  }
  ```

* 隐私复制到剪切板ClipboardManager#setPrimaryClip()

* android:sharedUserId 不得再使用。

* 前台服务的通知可以被关闭。

* 新的wifi运行时ACCESS_FILE_LOCATION权限。主要是用来通过wifi推导物理位置。类似android12的通过蓝牙获取位置。

  主要有ACCESS_FILE_LOCATION和NEARBY_WIFI_DEVICES。自行研究。

* 媒体权限细化，READ_MEDIA_IMAGES, READ_MEDIA_VIDEO, READ_MEDIA_AUDIO, 再结合android14的`READ_MEDIA_VISUAL_USER_SELECTED`。

  > 这里推荐不是做图库类似强需求的应用，统统采用照片选择器picker，以提供一致的图片和视频选择体验，同时增强用户隐私保护，而无需请求任何存储权限。这里我实名五星好评。

* BOOT_COMPLETED/LOCKED_BOOT_COMPLETED可能会根据电池计划把你限制，不再广播给你。

* 播放器相关的`PlaybackState` 的变更。详情自己研究。

* WebView通过`prefers-color-scheme`标准查询。

* 通过BluetoothAdapter调用disable()和enable()弃用。

* googlePlay 广告ID权限。

  如果您的应用以 Android 13 或更高版本为目标平台且未声明此权限，系统会自动移除广告 ID 并将其替换为一串零。

  如果您的应用使用的 SDK 在库的清单中声明了 `AD_ID` 权限，则默认情况下，该权限会与应用的清单文件合并。在这种情况下，您无需在应用的清单文件中声明该权限。

```
  <!-- Required only if your app targets Android 13 or higher. -->
      <uses-permission android:name="com.google.android.gms.permission.AD_ID"/>
```

* app和系统语言不同的设置方案。

  >  值得好好研究，之前的代码存在一些太多的代码设置。

  在许多情况下，多语言用户会将其系统语言设置为某一种语言（例如英语），但又想为特定应用选择其他语言（例如荷兰语、中文或印地语）。为了帮助应用为这些用户提供更好的体验，Android 13 针对支持多种语言的应用引入了以下功能：

  - **系统设置**：用户可以在这个集中位置为每个应用选择首选语言。

    您的应用必须在应用的清单中声明 `android:localeConfig` 属性，以告知系统它支持多种语言。如需了解详情，请参阅有关[创建资源文件并在应用的清单文件中声明资源](https://developer.android.com/guide/topics/resources/app-languages?hl=zh-cn#app-language-settings)的说明。

  - **其他 API**：借助这些公共 API（例如 [`LocaleManager`](https://developer.android.com/reference/android/app/LocaleManager?hl=zh-cn) 中的 [`setApplicationLocales()`](https://developer.android.com/reference/android/app/LocaleManager?hl=zh-cn#setApplicationLocales(android.os.LocaleList)) 和 [`getApplicationLocales()`](https://developer.android.com/reference/android/app/LocaleManager?hl=zh-cn#getApplicationLocales()) 方法），应用可以在运行时设置不同于系统语言的其他语言。

    这些 API 会自动与系统设置同步；因此，使用这些 API 创建自定义应用内语言选择器的应用将确保用户获得一致的用户体验，无论他们在何处选择语言偏好设置。公共 API 还有助于减少样板代码量、支持拆分 APK，并且支持[应用自动备份](https://developer.android.com/guide/topics/data/autobackup?hl=zh-cn)，以存储应用级的用户语言设置。

* 支持JDK11。



### android12

* 启动画面。

  SplashScreen 兼容。参考我以前的帖子https://blog.csdn.net/jzlhll123/article/details/136628746。在未来某一天android最低开始支持android12的时候，就可以不做兼容了。

* 废弃一些获取屏幕的办法

  这里推荐如下2个万能公式：

  ```kotlin
  /**
   * 无需等待界面渲染成功，即在onCreate就可以调用，而且里面已经做了低版本兼容，感谢jetpack window库
   * 获取的就是整个屏幕的高度。包含了statusBar，navigationBar的高度一起。与wm size一致。
   * 这个方法100%可靠。虽然我们看api上描述说低版本是近似值，但是也是最接近最合理的值，不会是0的。
   */
  fun Activity.getScreenFullSize() : Pair<Int, Int> {
      val m = WindowMetricsCalculator.getOrCreate().computeCurrentWindowMetrics(this)
      //computeMaximumWindowMetrics(this) 区别就是多屏，类似华为推上去的效果。不分屏就是一样的。
      return m.bounds.width() to m.bounds.height()
  }
  
  /**
   * 必须在activity已经完全渲染之后，一般地，我们是通过
   * ViewCompat.setOnApplyWindowInsetsListener(decorView) { _, insets ->
   *         val navHeight = insets.getInsets(WindowInsetsCompat.Type.navigationBars()).bottom
   *         val statusHeight = insets.getInsets(WindowInsetsCompat.Type.statusBars()).top
   *
   *  来得到结果的。但是它并不一定会回调，必须调用WindowCompat.setDecorFitsSystemWindows(this, false)。
   *
   *  想要获取，要么，如上，setOnApplyWindowInsetsListener的回调。
   *  要么，同View.post里面再调用本函数获取。
   */
  fun Activity.currentStatusBarAndNavBarHeight() : Pair<Int, Int>? {
      val insets = ViewCompat.getRootWindowInsets(window.decorView) ?: return null
      val nav = insets.getInsets(WindowInsetsCompat.Type.navigationBars()).bottom
      val sta = insets.getInsets(WindowInsetsCompat.Type.statusBars()).top
      return sta to nav
  }
  ```

* ACESS_FINE_LOCATION/ACCESS_COARSE_LOCATION位置权限的变更，自行研究，主要是限制app获取位置。

* 自定义通知，标准android的通知样式。

  自行研究，[`Notification.DecoratedCustomViewStyle`](https://developer.android.com/reference/android/app/Notification.DecoratedCustomViewStyle?hl=zh-cn)是什么？

  自定义通知样式又是什么：[`setCustomContentView(RemoteViews)`](https://developer.android.com/reference/android/app/Notification.Builder?hl=zh-cn#setCustomContentView(android.widget.RemoteViews))、[`setCustomBigContentView(RemoteViews)`](https://developer.android.com/reference/android/app/Notification.Builder?hl=zh-cn#setCustomBigContentView(android.widget.RemoteViews)) 和 [`setCustomHeadsUpContentView(RemoteViews)`](https://developer.android.com/reference/android/app/Notification.Builder?hl=zh-cn#setCustomHeadsUpContentView(android.widget.RemoteViews)) 的应用。

* Activity和Service，广播，需要设置标签`android:exported`。除了申明`LAUNCHER`类别的，设置true，其他一般都考虑false。

* 精确闹钟权限，请在清单中请求 [`SCHEDULE_EXACT_ALARM`](https://developer.android.com/reference/android/Manifest.permission?hl=zh-cn#SCHEDULE_EXACT_ALARM) 权限。变成了特殊权限。需要自行跳转到设置界面。

  ```java
  if (!alarmManager.canScheduleExactAlarms()) {
      //弹窗确认引导跳转
      ...
      startActivity(Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM))
  }
  ```

* 蓝牙权限，wifiManager等一些变更。

[行为变更：以 Android 12 为目标平台的应用  | Android Developers](https://developer.android.com/about/versions/12/behavior-changes-12?hl=zh-cn)

其实还有挺多的，不过没有太多适配问题不再这里提出。

### 更早的一些变更

* android11，单次授权，位置，拍照，录音等。对开发者其实没有太多影响，只是系统弹窗的差异。

* storage相关

android10-11，storage的分区存储变更。

引入**MANAGE_EXTERNAL_STORAGE**权限。

还可以通过[`requestLegacyExternalStorage`](https://developer.android.google.cn/reference/android/R.attr?hl=zh-cn#requestLegacyExternalStorage)属性，来屏蔽带来的影响。

[`ACTION_MANAGE_STORAGE`](https://developer.android.google.cn/reference/kotlin/android/os/storage/StorageManager?hl=zh-cn#action_manage_storage) 

>  但是对于APP开发，最好的做法是，不要往外部存储去存储。只调用自己的沙盒空间。

```kotlin
/**
 * 选择合适的cacheDir
 */
val goodCacheDir : File by unsafeLazy { app.externalCacheDir ?: app.cacheDir }

/**
 * 选择合适的filesDir
 */
val goodFilesDir : File by unsafeLazy { app.getExternalFilesDir(null) ?: app.filesDir }
```

* 涉及读取图片/视频，使用ActivityResultContracts+ActivityResultContracts.PickVisualMedia()/ActivityResultContracts.PickMultipleVisualMedia方式，五星推荐，不要任何权限。

* 涉及读取文档，可以通过ActivityResultContracts+GetContent/OpenDocument/OpenDocumentTree：

  ```kotlin
  /**
   *StartActivityForResult: 通用的Contract,不做任何转换，Intent作为输入，ActivityResult作为输出，这也是最常用的一个协定。
   *RequestMultiplePermissions： 用于请求一组权限。
   *RequestPermission: 用于请求单个权限。
   *TakePicturePreview: 调用MediaStore.ACTION_IMAGE_CAPTURE拍照，返回值为Bitmap图片。
   *TakePicture: 调用MediaStore.ACTION_IMAGE_CAPTURE拍照，并将图片保存到给定的Uri地址，返回true表示保存成功。
   *TakeVideo: 调用MediaStore.ACTION_VIDEO_CAPTURE 拍摄视频，保存到给定的Uri地址，返回一张缩略图。
   *PickContact: 从通讯录APP获取联系人。
   *GetContent: 提示用选择一条内容，返回一个通过。ContentResolver#openInputStream(Uri)访问原生数据的Uri地址（content://形式） 。默认情况下，它增加了Intent#CATEGORY_OPENABLE, 返回可以表示流的内容。
   *CreateDocument: 提示用户选择一个文档，返回一个(file:/http:/content:)开头的Uri。
   *OpenMultipleDocuments: 提示用户选择文档（可以选择多个），分别返回它们的Uri，以List的形式。
   *OpenDocumentTree: 提示用户选择一个目录，并返回用户选择的作为一个Uri返回，应用程序可以完全管理返回目录中的文档。
   *上面这些预定义的Contract中，除了StartActivityForResult和RequestMultiplePermissions之外，基本都是处理的与其他APP交互，返回数据的场景，比如，拍照，选择图片，选择联系人，打开文档等等。使用最多的就是StartActivityForResult和RequestMultiplePermissions了。
   */
  ```

* 涉及往外导出文件，则通过如下一些函数来解决：

```kotlin
/**
 * 导出到download目录。
 * 似乎是能支持。而且不要权限。
 */
suspend fun exportFileToDownload(outputFileName: String, sourceFile: File): String {
    // 验证源文件是否存在
    if (!sourceFile.exists()) {
        return "error: Source file does not exist."
    }

    // 获取下载目录
    val has = hasPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE)
    val directory = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS)

    val generateUniqueFileName = fun(directory: File, originalName: String): File {
        // 分离文件名和扩展名
        val (name, ext) = if (originalName.contains('.')) {
            val lastDotIndex = originalName.lastIndexOf('.')
            originalName.substring(0, lastDotIndex) to originalName.substring(lastDotIndex)
        } else {
            originalName to ""
        }

        var counter = 0
        var uniqueFile = File(directory, originalName)

        // 循环查找可用的文件名
        while (uniqueFile.exists()) {
            counter++
            val newName = if (ext.isNotEmpty()) {
                "${name}-${counter}${ext}"
            } else {
                "${name}-${counter}"
            }
            uniqueFile = File(directory, newName)
        }

        return uniqueFile
    }

    // 创建目标文件
    val destFile = generateUniqueFileName(directory, outputFileName)
    delay(0)
    try {
        // 使用文件流拷贝
        sourceFile.inputStream().use { input ->
            destFile.outputStream().use { output ->
                input.copyTo(output)
            }
        }
        return "Success! " + destFile.absolutePath + "\n" + destFile.absolutePath
    } catch (e: IOException) {
        e.printStackTrace()
        return "error: ${e.localizedMessage}"
    }
}

/**
 * 保存文件（支持大文件）
 */
fun saveFileToPublicDirectory(
    context: Context,
    origFile: File,
    deleteOldFile: Boolean = true,
    path: String,
    setContentValues: Function1<ContentValues, Unit>? = null
) : Uri?{
    if (origFile.isDirectory) {
        return null
    }

    val uri = insertFileToContentResolverFile(context, origFile, origFile.name, path, setContentValues)
    var isSuc = false
    if (uri != null) {
        context.contentResolver.openOutputStream(uri)?.use { outputStream ->
            origFile.inputStream().buffered().use { inputStream ->
                val buffer = ByteArray(1024 * 32)
                var bytesRead: Int
                while (inputStream.read(buffer).also { bytesRead = it } != -1) {
                    outputStream.write(buffer, 0, bytesRead)
                    isSuc = true
                }
            }
        }

        ignoreError {
            if (isSuc && deleteOldFile) {
                origFile.delete()
            }
        }
    }
    return if(isSuc) uri else null
}

fun deleteFromContentResolver(mediaUri: Uri) {
    Globals.app.contentResolver.delete(mediaUri, null, null)
}

/**
 * 获取想要插入到的目标位置的uri
 *
 * @param mediaType 文件类型 使用函数：mediaTypeOf2 来解析。
 * @param mimeType 文件类型
 * @param displayName 目标文件名
 * @param subPath 目标在媒体文件夹下的子目录
 * @param contentValuesAction 给你额外操作的可能。
 * @return 返回文件uri
 */
private fun insertFileToContentResolver(
    context: Context,
    mediaType: MediaType,
    mimeType: String,
    displayName: String,
    subPath: String,
    contentValuesAction: Function1<ContentValues, Unit>? = null
) : Uri? {
    return ignoreError {
        //设置保存参数到ContentValues中
        val contentValues = ContentValues()
        //执行insert操作，向系统文件夹中添加文件
        //EXTERNAL_CONTENT_URI代表外部存储器，该值不变
        val contentResolver = context.contentResolver
        val saveUri = when (mediaType) {
            MediaType.Video -> {
                //设置文件名
                contentValues.put(MediaStore.Video.Media.DISPLAY_NAME, displayName)
                contentValues.put(MediaStore.Video.Media.TITLE, displayName)
                //设置文件类型
                contentValues.put(MediaStore.Video.Media.MIME_TYPE, mimeType)
                //兼容Android Q和以下版本
                if (!isExternalStorageLegacy) {
                    //android Q中不再使用DATA字段，而用RELATIVE_PATH代替
                    //RELATIVE_PATH是相对路径不是绝对路径
                    //DCIM是系统文件夹，关于系统文件夹可以到系统自带的文件管理器中查看，不可以写没存在的名字
                    contentValues.put(MediaStore.Video.Media.RELATIVE_PATH, "Movies/${subPath}")
                } else {
                    contentValues.put(MediaStore.Video.Media.DATA, getOldSdkPath(displayName, subPath, "Movies"))
                }
                MediaStore.Video.Media.EXTERNAL_CONTENT_URI
            }

            MediaType.Image -> {
                contentValues.put(MediaStore.Images.Media.DISPLAY_NAME, displayName)
                contentValues.put(MediaStore.Images.Media.TITLE, displayName)
                contentValues.put(MediaStore.Images.Media.MIME_TYPE, mimeType)
                if (!isExternalStorageLegacy) {
                    contentValues.put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/${subPath}")
                } else {
                    contentValues.put(MediaStore.Images.Media.DATA, getOldSdkPath(displayName, subPath, "Pictures"))
                }
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI
            }

            MediaType.Audio -> {
                contentValues.put(MediaStore.Audio.Media.DISPLAY_NAME, displayName)
                contentValues.put(MediaStore.Audio.Media.TITLE, displayName)
                contentValues.put(MediaStore.Audio.Media.MIME_TYPE, mimeType)
                if (!isExternalStorageLegacy) {
                    contentValues.put(MediaStore.Audio.Media.RELATIVE_PATH, "Music/${subPath}")
                } else {
                    contentValues.put(MediaStore.Audio.Media.DATA, getOldSdkPath(displayName, subPath, "Music"))
                }
                MediaStore.Audio.Media.EXTERNAL_CONTENT_URI
            }

            MediaType.Other -> {
                contentValues.put(MediaStore.Downloads.DISPLAY_NAME, displayName)
                contentValues.put(MediaStore.Downloads.TITLE, displayName)
                contentValues.put(MediaStore.Downloads.MIME_TYPE, mimeType)
                if (!isExternalStorageLegacy) {
                    contentValues.put(MediaStore.Downloads.RELATIVE_PATH, "Download/${subPath}")
                    MediaStore.Downloads.EXTERNAL_CONTENT_URI
                } else {
                    contentValues.put(MediaStore.Files.FileColumns.DATA, getOldSdkPath(displayName, subPath, "Download"))
                    MediaStore.Files.getContentUri("external")
                }
            }
        }
        contentValuesAction?.invoke(contentValues)
        return contentResolver.insert(saveUri, contentValues)
    }
}

/**
 * 获取想要插入到的目标位置的uri
 *
 * 将原来的origUri，转成mediaType和mimeType
 * 再调用insertFileToContentResolver
 */
fun insertFileToContentResolverUri(
    context: Context,
    origUri:Uri,
    displayName: String,
    subPath: String,
    contentValuesAction: Function1<ContentValues, Unit>? = null
) : Uri? {
    val mimeType = origUri.getUriMimeType(context.contentResolver)
    val mediaType = MediaHelper.mediaTypeOfMimeType(mimeType)
    return insertFileToContentResolver(context, mediaType, mimeType, displayName, subPath, contentValuesAction)
}

/**
 * 获取想要插入到的目标位置的uri
 *
 * 将原来的origFile，转成mediaType和mimeType
 * 再调用insertFileToContentResolver
 */
fun insertFileToContentResolverFile(
    context: Context,
    origFile: File,
    displayName: String,
    subPath: String,
    contentValuesAction: Function1<ContentValues, Unit>? = null
) : Uri? {
    val origUri = Uri.fromFile(origFile)
    return insertFileToContentResolverUri(context, origUri, displayName, subPath, contentValuesAction)
}

/**
 * 是否是用原来的存储
 * true:没有启用分区存储
 * false:当前是分区存储
 */
val isExternalStorageLegacy: Boolean
    get() = Build.VERSION.SDK_INT < Build.VERSION_CODES.Q || Environment.isExternalStorageLegacy()
```

android8权限系统变更。
