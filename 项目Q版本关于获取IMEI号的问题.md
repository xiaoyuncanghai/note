#项目Q版本关于获取IMEI号的问题(行为变更)
折叠屏增强项、新网络连接 API、全新的媒体解码器、摄像头新功能、NNAPI 扩展、Vulkan 1.1 图形支持等等

##用户隐私权限变更

###1:存储权限:
访问和共享外部存储设备中的文件的应用
adb shell sm set-isolated-storage on
1>:目的: 保证用户文件的隐私性，并有助于减少应用所需的权限数量
2>:分析:外部存储设备中为每个应用提供了一个应用专属文件夹，并且本应用访问这个文件夹无需权限（例如 /sdcard）。任何其他应用都无法直接访问您应用的沙盒文件. 只要本应用可以访问, 
3>API:
getExternalFilesDir(Environment.DIRECTORY_PICTURES)
4:适配:
- 访问自己文件：Q中READ_EXTERNAL_STORAGE 和 WRITE_EXTERNAL_STORAGE权限，并且无需特定权限，应用即可访问自己沙盒中的文件。
- 访问系统媒体文件：Q中引入了一个新定义媒体文件的共享集合，如果要访问沙盒外的媒体共享文件，比如照片，音乐，视频等，需要申请新的媒体权限:READ_MEDIA_IMAGES,READ_MEDIA_VIDEO,READ_MEDIA_AUDIO,申请方法同原来的存储权限。
- 访问系统下载文件：对于系统下载文件夹的访问，暂时没做限制，但是，要访问其中其他应用的文件，必须允许用户使用系统的文件选择器应用来选择文件。(无限制)

请判断当应用运行在Q平台上时，取消对READ_EXTERNAL_STORAGE 和 WRITE_EXTERNAL_STORAGE两个权限的申请。并替换为新的媒体特定权限。

###2:定位权限
在后台时请求访问用户位置信息的应用(Q版本始终启用)
ACCESS_BACKGROUND_LOCATION, 新权限仅会影响应用在后台运行时对位置信息的访问权, 应用需要在后台时也获得用户位置(比如滴滴)，就需要动态申请ACCESS_BACKGROUND_LOCATION权限。

targetSDK <= P 应用如果请求了ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION权限，Q设备会自动帮你申请ACCESS_BACKGROUND_LOCATION权限。

###3:后台启动Activity
不需要用户互动就启动 Activity 的应用
关闭允许系统执行后台活动开发者选项即可启用限制
即后台启动新的activity会给用户发送一个通知, 若干情况除外, 如:recent task的activity
https://developer.android.com/guide/components/activities/background-starts

###4:设备标识符(deviceID)
访问设备序列号或 IMEI 的应用
1: 从 Android Q 开始，应用必须具有 READ_PRIVILEGED_PHONE_STATE 签名权限才能访问设备的不可重置标识符,如果您的应用没有该权限，但您仍尝试查询标识符的相关信息。会返回空值或报错。
2:原来的READ_PHONE_STATE权限已经不能获得IMEI和序列号,且官方所说的READ_PRIVILEGED_PHONE_STATE权限只提供给系统app
3:设备唯一ID可以按照具体需求具体解决。给出一个不变和基本不重复的UUID方法。由于唯一标识符权限的更改会导致android.os.Build.getSerial()返回unknown,但是由于m_szDevIDShort是由硬件信息拼出来的，所以仍然保证了UUID的唯一性和持久性。
	代码块: 



###5:无线扫描权限
使用 WLAN API 和 Bluetooth API 的应用

###其他
android O的读取已安装应用权限（对应用内自动更新有影响）
android P的默认禁止访问http的API

Android Q Beta开发者文档链接：：https://developer.android.com/preview