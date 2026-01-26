### 修改点

android10增加了**UnSpecialized Application Process**机制，提前预支一个fork好的进程。

修改MAX_CACHE_PROCESS 调整缓存数量。

调整oom_adj。

锁颗粒度，AMS大量的锁。android10+ ATMS



系统开机加速

android整个阶段包括：bootloader，kernel，init.rc，zygote，SystemServer，Launcher，bootCompleted。

bootLoader、kernel略：有一些策略比如日志减少，精简内核，异步一些初始化；

到了init.rc阶段：裁剪里面的执行脚本；

camera;

没有电话，就telephony相关移除，apk移除；

android.software.print.xml

是否需要传感器等等。手势唤醒；锁屏通知等等。双击唤醒。

zygote-start加速；

SystemServer裁剪；并发；拆分一些AMS，WMS的初始化过程；异步完成。

桌面：SplashScreen尽快可见；减少冷启动的逻辑，application的初始化；延迟加载。

PMS扫描线程加速。



trace、perfetto，来研究哪里比较慢。再分析耗时：

```
1.VibratorService          震动器服务  
2.ClipboardService         粘贴板服务  
3.FingerprintService       指纹  
4.BatteryService           电池服务，当电量不足时发广播  
5.AlarmManagerService      闹钟服务   
6.WallpaperManagerService  壁纸管理服务  
7.StatusBarManagerService  状态栏管理服务  
StartCountryDetectorService
```





bootChart分析时间消耗图。

从软件的角度看主要系统上主要有两个地方：preload classes resource和scan packages。

Android zygote 启动的时候会预加载一些类，预加载的类根据frameworks\base\preloaded-classes 。这个是下载android代码的时候已经确定的。是google 给出的参考。和这个文件相关：framework/base/tools/preload/WritePreloadedClassFile.java 在这个文件中定义了需要预加载类的规则并且生成frameworks\base\preloaded-classes。预加载类的规则如下：

/*

​     \* The set of classes to preload. We preload a class if:

​     *

​     \* a) it's loaded in the bootclasspath (i.e., is a system class)

​     \* b) it takes > MIN_LOAD_TIME_MICROS us to load, and

​     \* c) it's loaded by more than one process, or it's loaded by an

​     \*   application (i.e., not a long running service)

​     */

就是a)系统级的类， 加载时间大于 b) MIN_LOAD_TIME_MICROS的类 c) 被应用加载次数大于两次的类。

MIN_LOAD_TIME_MICROS 定义如下：

 static final int MIN_LOAD_TIME_MICROS = 1250;

 





