### Charles抓https请求的爬坑路

第一步:[Android安装Charles证书（华为手机测试）](https://blog.csdn.net/weixin_42034554/article/details/86669159)

​	注意：可以使用电脑浏览器访问chls.pro/ssl，下载charles-proxy-ssl-proxying-certificate.pem文件然后adb push到sd卡

第二步:[Charles抓https显示unknown解决方法](https://www.jianshu.com/p/498884193013)

​	注意：Charles的SSL Proxying Settings，添加所有的域名这一步一定要有，否则就算信任了证书也全都是unknown

第三步:[如何解决 Android7.0之后部分手机无法抓包](https://blog.csdn.net/muranfei/article/details/89182997)
​	注意：关键代码

```
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" overridePins="true" />
            <certificates src="user" overridePins="true" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

### [Windows下用Charles对Android抓包HTTPS](https://blog.csdn.net/ybdesire/article/details/80636248)
> Android 7.0以上抓取Https请求还需要添加网络安全配置`网络安全配置`,具体可以参考[网络安全配置](https://developer.android.com/training/articles/security-config.html)

 ```java
 <?xml version="1.0" encoding="utf-8"?>
 <manifest ... >
	 <application android:networkSecurityConfig="@xml/network_security_config"
					 ... >
		 ...
	 </application>
 </manifest>
 ```
 其中network_security_config.xml具体为以下内容
 ```java
 <?xml version="1.0" encoding="utf-8"?>
 <network-security-config>
	 <base-config>
		 <trust-anchors>
			 <certificates src="user" />
		 </trust-anchors>
	 </base-config>
 </network-security-config>
 ```