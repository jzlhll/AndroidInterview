Android Framework源码开发揭秘 

## 系统启动流程分析

### 一、Android启动概览

drivers->HAL->ART->java framework->App



启动流程：

1. boot loader上电 

2. kernel 相关的进程

3. 用户空间进程**init进程**

​	pid1（

​	文件系统初始化挂载，SElinux等等；

​	解析init.rc, 

​	孵化一堆native进程mediaserver，servicemanager(有binder通信的管家, app 系统服务注册在这里)，surfaceflinger等等

​	孵化zygote进程

​	）

root             1          0 2317748   6588 0                   0 S init
root           603        1 2176420   3876 0                   0 S init

media            31      1     20944  3184  ffffffff afe0ca7c S /system/bin/mediaserver

system         673     1 2247796   4780 0                   0 S servicemanager
system         674     1 2231224   5712 0                   0 S hwservicemanager
system         675     1 2219576   3192 0                   0 S vndservicemanager
root           932        1 6620500  71700 0                   0 S zygote64
root           933        1   1820852  37288 0                 0 S zygote
system        1911   932 14044448 399540 0            0 S system_server

mediaServer进程：

service media /system/bin/mediaserve

user media

group system audio camera graphics inet net_bt net_bt_admin

audioFlinger，camera Service等等。



5. zygote进程

java应用程序和SystemServer都是从Zygote fork出来的。

它通过socket监听有人通知（主要是从AMS（AMS又是运行在SystemServer进程））

zygote进程里面preload了Framework.jar的资源和class。fork的时候，就更快地启动app。

监管systemServer进程死了，还得自杀并重启。

* fork过程

fork过程最好是单线程的进程，复制用户空间仅仅是复制了当前线程到子进程。

6. SystemServer

第一个zygote fork的进程。系统服务在里面。

startBootstrapServices(PMS, Recovery, DsiplayManagerService, AMS)...

startCoreServices(),

startOtherServices



### 二、binder解析

#### aidl / java binder

定义一个aidl接口。它会自动实现IInterface。

然后会有Stub和Proxy。从接口来看，都一样，但是同进程下，拿的就是真实执行对象；不同进程则是拿的binder的代理对象，通信的时候，就会通过transact，挂起等待，服务端那边收到onTransact后，还要回执唤醒binder线程继续往下执行。

> 进程间通信：socket（网络，没有权限系统设计，比较慢），共享内存（不用考虑，但是内存边界，难用，不安全），管道，消息队列

匿名service，和非匿名（AMS，PMS这些）。



用户空间的虚拟内存地址，映射到物理内存中，mmap。

1MB上限。

#### C++ binder

BpBinder（proxy，对应调用方的Client端）

BnXXXX（服务端实现的本身）

#### Anonymous Shared Memory匿名共享内存

服务端构造好地址空间，客户端会请求服务端返回的信息，映射好客户端进程的地址空间。

#### Camera流程

java层 Camera开启，会跑到JNI，跑到framework的camera c库，此时都是在app进程中；

当从systemServer拿到cameraService，就跨进程调用过去，创建一个代理的client存在cameraService中。

所以当时，也考虑过做一个虚拟的cameraClient层，就需要修改cameraService的代码。

如果想要同时2路camera，也需要在cameraService解禁。



### 三、Handler详解

handle简单地讲，就是跨线程通信的一种方式；你想在某个线程收消息，其实就通过HandlerThread，起了一个死循环的Loop MessageQueue。没消息的时候，它就等待在那（并不阻塞线程的等待，eventfd或epoll机制）。

等有消息了，就唤醒一下，执行下你的消息内容Runnable或者handleManager。

异步消息：消息屏障：其实是为了保证系统的那些动作优先执行，app都不允许设置。

插队，其实就是利用链表的特性，将消息提前挂到前面去，定时唤醒。

draw,requestLayout, invalidate都是异步消息。为了就是不卡界面。

Looper通过sThreadLocal来存储的。确保了线程唯一。



IdleHandler 空闲的时候，执行任务的时机，根据业务场景使用，不太可控。



### 四、ActivityManagerService解析

ActivityMangerService，四大组件的调度和管理器。



在system_server进程中启动 SystemServer.java #startBootstrapServices() 创建AMS实例对象，

创建Andoid Runtime，ActivityThread和Context对象； 

setSystemProcess：注册 AMS、meminfo、cpuinfo等服务到ServiceManager； 

installSystemProviderss，加载 SettingsProvider； 

启动SystemUIService，再调用一系列服务的systemReady()方法； 

发布Binder服务。



通过本地socket跟zygote进程通信。去fork新的app。

#### Application的启动过程

ActivityThread main函数执行，创建activityThread对象

handler Loop启动

attachApplication就是把自己ApplicationThread这个binder交给AMS(负责管理所有的进程，activity等)

再bindApplication调回去你的进程

​	ActivityThread：handlerBindApplication

1. 创建ContentImpl，2. Instrumentation，3. Application对象，4. 启动contentProvider，5. 最后Application onCreate





#### activity的启动过程

1. startActivity->Instrumentation.execStartActivity()其实就是找到ActivityManagerService在本app进程里面的代理对象activityManager这个binder对象，去调用startActivity；

2. AMS：ActivityStarter解析intent，权限等ActivityStackSupervisior/ActivityStack 
3. socket让zygote创建进程
4. 有了新进程后ActivityThread main函数。类似一个java实例。管理了四大组件的操作。ApplicationThread是ActivityThread的内部类，其实是公开给AMS去调用（即让其他APP调到这里），进而执行ActivityThread里面的代码。
5. ActivityThread ClassLoader加载Activity。 onCreate。



#### launcher的启动过程

当AMS准备完成以后，

systemReady，继续追查代码，

寻找到需要显示的区域，解析homeActivity的信息，

就会startHomeActivity。

最后才会发出bootCompleted广播。



### 五、WindowManagerService解析

activity只负责生命周期和事件的管理

window只控制视图

一个activity包含一个window，没有window就跟service差不多了

所有APP的activity都由AMS统一调度

WMS则是控制所有的window显示，隐藏，和区域



“Window”表明它是和窗口相关的，“窗口”是一个抽象的概念，从用户的角度来讲，它是一个“界面”；从
SurfaceFlinger的角度来看，它是一个Layer（buffer），承载着和界面有关的数据和属性；从WMS角度来看，它是一个
WIndowState，用于管理和界面有关的状态。



任何视图都是通过window呈现，setContentView也是通过window完成。

PhoneWindow。

#### 概念

* windowManager就是app，也是binder的客户端，用来管理窗口顺序，消息等。

* windowManagerService是真正的管理类，也就是在SystemServer阶段创建在该进程里面。
* token
* type 层级
* flags显示属性

### 六、PackageManagerService详解

SystemServer里面会启动PKMS。主要负责：

解析AndroidManifest.xml，各种receiver，contentProvider，Activity，service等等信息。

扫描apk，安装应用；安装，卸载，查询等等。



### 七、andorid图形系统

#### 1. window加载视图的过程

Activity(PhoneWindow (DecorView(titleBar, contentView)) )

activity不是窗口，不是视图，只是管理各种界面，key，touch。

window装载View的容器，处理窗口逻辑，PhoneWindow，被windowManager管理。

DecorView顶级视图。

token维护Activity和window的对应关系



activity的启动过程：

App1进程startActivity -> AMS startActivity -> AMS socket zyogte孵化app2进程 

-> app2进程触发activityThread main函数

->handlerThread，looper，context，contentProviders，Application准备好

->launcherActivity



在launcherActivity的过程中：

会通过classLoader创建出对应的activity。在attach函数里面，创建PhoneWindow。

一个activity或者Dialog会包含一个PhoneWindow，里面创建了DecorView，然后把app xml布局我们app层的view都是贴在decorView里面的。

进程里面的Window，ViewRootImpl都在WindowManagerGlobal里面管理。

它会跟WMS交互。

WMS里面addView，就是校验和分组token，type，创建WindowState，对window层级。

同时，ViewRootImpl里面setView，requestLayout，scheduleTraversals() 
![请添加图片描述](https://i-blog.csdnimg.cn/direct/ac5f757147894a1ba0619693edd612e8.png)

#### 2. View的绘制流程

performTraversal()

	performMeasue()
	
	performLayout()
	
	performDraw()



真实5个过程：

1. 预测量阶段performTraversals

    measureHierarchy最后也是调用了performMeasure过程，多了协商的过程。

2. 窗口布局阶段relayoutWindow

3. 测量过程performMeasure

   View的递归遍历View树

   getMeasuredWidth生效

4. 布局过程performLayout

   getWidth生效

5. 绘制过程performDraw()



#### 3. activity，window，view的关系

activity主要是管理生命周期

window是View的容器，这样的话，activity跟View解耦，windowManger才是真实管理View的。

activity与window就是生命周期与key，touch事件回调

window与view是处理管理



#### 4. Surface图形系统

其中每一层之间的数据传递是使用Buffer（缓冲区）作为载体, 上层和surfaceFlinger间的buffer为图形缓冲区，framework与显示屏间的buffer是硬件帧缓冲区。

ViewRootImpl发起performTravesal绘制View的流程：

measure，layout，draw，最后就是调用图形处理引擎。skia，或者opengl es， vulkan。

一个window，对应一个Surface，其实在ViewRootImpl里面，app进程中，hardwareRender。



surface->GLConsumer->SurfaceFlinger->HWComposer



##### Surface

上层app通过Surface获取buffer，供上层绘制，绘制过程通过Canvas来完成，底层实现是skia引擎，绘制完成后数据通过Surface被queue进BufferQueue, 然后监听会通知SurfaceFlinger去消费buffer, 接着SurfaceFlinger就acquire数据拿去合成, 合成完成后会将buffer release回BufferQueue。如此循环，形成一个Buffer被循环使用的过程。



所以Surface主要干两件事：获取Canvas来干绘制的活。申请buffer，并把Canvas最终生产的图形、纹理数据放进去。

##### SurfaceFlinger

System_server进程中的一个线程服务。接收所有surface作为输入。创建layer（BufferQueue）与Surface一一对应。计算每个layer最终合成图像，交给HWComposer或者openGL栅格化数据，最终framebuffer上。

##### Layer

Layer是SurfaceFlinger 进行合成的基本操作单元，其主要的组件是一个 BufferQueue。Layer在应用请求创建Surface的时候在SurfaceFlinger内部创建，因此一个Surface对应一个 Layer。Layer 其实是一个 FrameBuffer，每个 FrameBuffer 中有两个 GraphicBuffer 记作 FrontBuffer 和 BackBuffer。

##### HWComposer

Cpu合成还是GPU合成，显示。

##### Vsync & Buffer个数

一个buffer，撕裂，因为同时写入同时读取；

2个buffer，不会撕裂，但是卡顿，cpu、gpu利用率偏低；因为来一个vsync信号生成一次

3个buffer，避免1个buffer同时需要上屏显示使用不能被修改，2个buffer正在cpu和gpu处理中，弄出第三个可以在vsync来的时候，立刻开始合成。提升cpu、gpu的利用率，减少卡顿发生。但仍然需要引入延迟。

##### Choreographer

因为vsync其实是给硬件帧缓冲区发信号的。于是android加了上层也能接收屏幕vsync信号的。

Choreographer向surfaceFlinger注册DisplayEventReceiver。又配合handler里面的屏障机制，优先绘制UI，而不是我们其他的事件。



#### 5. app与surfaceFlinger服务连接

Activity.makeVisible，通过windowManage addView，将decorView。

ViewRootImpl里面new Surface()，是parcelable可以跨进程传递的；有windowSession也是binder对象，就是WMS在app进程的代理。

session.addToDisplay, WMS.addWindow，就将客户端的window，传递到了WMS中，并组装成windowState。

windowState.attach() -> windowAddedLocked() -> SurfaceSession，会调用到JNI，nativeCreate。



在JNI层，nativeCreate，SurfaceComposerClient，会去拿去SurfaceFlinger服务的代理接口。

ComposerService作为client 与 SurfaceFlinger server进行binder IPC , 获取到SurfaceFlinger创建的Client对象，它相当于是SurfaceFlinger内部对应用程序客户端的封装对象，而Client与SurfaceComposerClient又互为binder ipc的两端，SurfaceComposerClient为client端，Client为server端。



app进程最终是createSurface，到了SurfaceFlinger就是创建layer。

app通过Surface（GraphicBufferProducer，dequeueBuffer）向SurfaceFlinger申请buffer。

![image-20240815114118228](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815114118228.png)





![image-20240815104211941](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815104211941.png)

![image-20240815110205021](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815110205021.png)

![image-20240815110948555](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815110948555.png)

![image-20240815114154150](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815114154150.png)

#### 6. View ，Canvas，Surface

Surface通过dequeueBuffer流程（具体操作在此不多赘述）获取一块存放绘制数据的buffer。

View 在onDraw的时候，通过传入的Canvas进行绘制。（这里只是一个绘制的入口而已，本文是针对requestLayout流程来讲述的，当然你单独用Canvas实现绘制也是一样的）。

调用java层的CanvasAPI，实际真正负责绘制工作的是底层的Skia引擎，这里核心类包括SKCanvas（画家）以及SKBitmap（画布），绘制好的内容放入Surface 通过dequeueBuffer获取到的GraphicBuffer。

绘制完毕后，Surface通过queueBuffer将存放好绘制数据的buffer投递到队列中，并通知SurfaceFlinger消费。



#### SurfaceFlinger启动过程

从init进程fork而来，native进程。main函数。从ps -e就可以看到parent 进程id。

EGL：GPU api，绘制surface，配置framebuffer，swap绘制结果

HWComposer：硬件组合抽象层，处于SurfaceFlinger和HAL之间，处理部分合成工作和产生vsync信号。

EventThread：线程，接受vsync事件，分发到上层。



![image-20240815114850463](C:\Users\allan.jiang\AppData\Roaming\Typora\typora-user-images\image-20240815114850463.png)

### 八、进程管理

进程adj。是否前台Activity，是否有前台服务，后台service，reciever



### 九、虚拟录屏

Android DisplayManager提供了createVirtualDisplay接口，用于创建虚拟屏。MediaProjection也支持。

传入Surface。

转到DMS中，创建DisplayDevice，统一管理。

DMS调用SurfaceFlinger，创建Display。



* SurfaceView vs TextureView

  有自己的Surface，就有对应的WMS管理的Layer， windowState。

  TextureView没有，而是通过硬件加速，将buffer处理好后，转变成View。收到decorView统一的ViewTree管理

  
