#OSC-China源码学习——Splash界面

前段时间学习了git-osc的源码，今天开始学习OSC-China的源码，按照之前的流程进行学习。

##功能介绍：

在应用启动的时候，出现一个启动的欢迎界面，在这个界面中完成的任务：

> 1. Log日志的上传；
> 2. 跳转到主页面
> 3. 动画——在动画结束的时候进行上述两项操作

##集成指南：

在自己开发应用的时候，Splash界面可以用来完成一些初始化工作，比如：

> 1. 日志信息的上传；
> 2. 资源的初始化（自己用过的经历——在Splash动画跳转的时候，将Assets文件夹中的内容拷贝到SD卡）

##详细介绍

1. AppStart.java —— 整个应用的入口
2. LogUploadService.java —— 在AppStart开启，完成上一次记录在本地的日志的上传
3. MainActivity.java —— 程序的主界面

---

###**AppStart分析**

在AppStart中通过一个动画来控制Splash界面中的图片优雅的展示出来，在动画结束的时候，完成两个动作：

> 1. 启动Service，上传日志；
> 2. 跳转到程序主界面。

###**LogUploadService分析**——启动Service

通过如下代码启动Service，需要在Mainfest文件中进行配置

```
	//在Mainfest文件中进行配置
  <service android:name="net.oschina.app.LogUploadService" />
  
 //启动Service
 Intent uploadLog = new Intent(this, LogUploadService.class);
 startService(uploadLog);
```
**LogUploadService**在下一篇文章中详细学习

###**MainActivity分析**——展示主界面

通过如下代码启动主界面Activity，需要在Mainfest文件中进行配置

```

	//在主界面中的配置
	<activity
            android:name=".ui.MainActivity"
            android:launchMode="singleTask"
            android:screenOrientation="portrait" >
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />

                <data
                    android:host="www.oschina.net"
                    android:scheme="http" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />

                <data
                    android:host="www.oschina.net"
                    android:scheme="https" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />

                <data
                    android:host="my.oschina.net"
                    android:scheme="http" />
            </intent-filter>
            <intent-filter>
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />

                <data
                    android:host="my.oschina.net"
                    android:scheme="https" />
            </intent-filter>
        </activity>
        
        //启动主界面
        Intent intent = new Intent(this, MainActivity.class);
        startActivity(intent);
```

## 总结收获

> 1. 在Splash界面中启动服务，上传日志Log。

## 目前为止，还不太明白的地方

* 在onCreate方法中有一个防止第三方跳转出现双实例的代码，还不太明白什么意思，代码如下：

```
			// 防止第三方跳转时出现双实例
        Activity aty = AppManager.getActivity(MainActivity.class);
        if (aty != null && !aty.isFinishing()) {
            finish();
        }
```

* 在onResume中有一系列的动作，还不明白干嘛的：

```
	  @Override
    protected void onResume() {
        super.onResume();
        int cacheVersion = PreferenceHelper.readInt(this, "first_install",
                "first_install", -1);
        int currentVersion = TDevice.getVersionCode();
        if (cacheVersion < currentVersion) {
            PreferenceHelper.write(this, "first_install", "first_install",
                    currentVersion);
            cleanImageCache();
        }
    }
```



