#项目Q版本兼容处理

###Android O后台执行限制
前言: 应用进入缓存状态时候, 如果没有活动的组件, 系统将解除应用所具有的唤醒锁. 为提高性能, 系统会限制未在前台运行的应用的某些行为.

1:应用无法使用清单注册的大部分隐式广播, 这些限制仅适用于针对 O 的应用。不过，用户可以从 Settings 屏幕为任意应用启用这些限制，即使应用并不是以 O 为目标平台。

2:针对 Android 8.0 的应用尝试在不允许其创建后台服务的情况下使用 startService() 函数, 会引发IllegalStateException
    分析: 新的 Context.startForegroundService() 函数将启动一个前台服务。现在，即使应用在后台运行，系统也允许其调用 Context.startForegroundService()。不过，应用必须在创建服务后的五秒内调用该服务的 startForeground() 函数。

###应用置于前台的定义
- 具有可见的Activity
- 具有前台服务
- 另一个前台应用已关联到该应用(绑定服务或者使用内容提供者)
- IMI
- 壁纸服务
- 通知侦听器
- 语音文本服务
如果上诉条件均不满足应用将被视为后台

###关于服务限制的相关解释
1: 处于前台时，应用可以自由创建和运行前台服务与后台服务。 进入后台时，在一个持续数分钟的时间窗内，应用仍可以创建和使用服务。在该时间窗结束后，应用将被视为处于空闲(IDEL)状态。此时，系统将停止应用的后台服务，就像应用已经调用服务的“Service.stopSelf()”方法。

2: 后台应用将被置于一个临时白名单中并持续数分钟。 位于白名单中时，应用可以无限制地启动服务，并且其后台服务也可以运行。处理对用户可见的任务时，应用将被置于白名单中, 如:
Firebase云消息
接受广播: 短信/彩信消息
通知notification执行pendingIntent

3: JobScheduler 替代后台服务
在 Android 8.0 之前，创建前台服务的方式通常是先创建一个后台服务，然后将该服务推到前台。
* Android 8.0 有一项复杂功能；
	系统不允许后台应用创建后台服务。 因此，Android 8.0 引入了一种全新的方法，即 Context.startForegroundService()，以在前台启动新服务。
	在系统创建服务后，应用有五秒的时间来调用该服务的 startForeground() 方法以显示新服务的用户可见通知。如果应用在此时间限制内未调用 startForeground()，则系统将停止服务并声明此应用为 ANR

###使用时候出现的bug:
startService => startForegroundService
报错信息:
AndroidRuntime: android.app.RemoteServiceException: Context.startForegroundService() did not then call Service.startForeground() 

	Service启动的时候不会报错，在Service.stopSelf的时候报错，并且catch不到异常。5s内不停止服务还会有anr问题。
	
	原因: service startForground.api - ActivityManagerService.startForgourd.api中 int flags = 0 表示禁止,  Google本意就是让没有可见通知的应用不可以偷偷启动服务在后台

###解决办法:
1: startForegroundService(Intent)
2: MyDemoService:
`
onStartCommond() {
   createNotificationChannel
}
`

createNotificationChannel中:
```java 
Notification notification = new Notification.Builder(this)
               .setContentTitle("New Message") .setContentText("You've received new messages.")
                .setSmallIcon(R.drawable.ic_launcher_foreground)
                .setChannelId(CHANNEL_ID)
                .build();
        startForeground(1,notification)
```

Android O 后台应用想启动服务就老老实实的加个notification给用户看，表示你自己在后台占着资源，杀不杀由用户决定，偷偷地在后台跑，=> anr+crash。
1）activity： Context.startForegroundService()
2）Service：startForeground(int id, Notification notification)（id must not be 0）


###方案二: JobIntentService
1: Android强制开发者使用 JobScheduler 作业替换后台服务的意思，使用jobIntentService就可以比较方便的使用JobScheduler这个工具。
继承关系:
 java.lang.Object
    ↳   android.content.Context
       ↳    android.content.ContextWrapper
           ↳    android.app.Service
               ↳    android.support.v4.app.JobIntentService



2: 处理被加入到job或service任务的一个辅助工具，8.0以下被当作普通的Intent使用startSerivce()启动service来执行。8.0以上任务被作为job用jobScheduler.enqueue()方法来分发

3: Jobscheduler: APP调度任务的接口，根据APP要求构建JobInfo，系统会在适当的时间调用JobInfo指定的JobService来执行你的任务。

4: jobIntentService处理work就是实现onHandleWork，可以看到其使用线程池的AsyncTask来处理work的，所以不需要考虑主线程阻塞的问题。

5:
权限<"uses-permission android:name="android.permission.WAKE_LOCK"></uses-permission>
<"service android:name=".MyJobIntentService"></service>'

6:Android O的后台限制，创建后台服务需要使用JobScheduler来由系统进行调度任务的执行，而使用JobService的方式比较繁琐，8.0及以上提供了JobIntentService帮助开发者更方便的将任务交给JobScheduler调度，其本质是Service后台任务在他的OnhandleWork()中进行，子类重写该方法即可




 
