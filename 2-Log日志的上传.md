# OSC-China源码学习——Log日志上传

在`AppStart`中开启了一个服务`LogUploadService`用来上传应用程序的日志。

采用的是`start`的方式开启服务，代码如下：

```
Intent uploadLog = new Intent(this, LogUploadService.class);
startService(uploadLog);
```

##一、功能介绍：

在服务`LogUploadService`被开启后，根据情况进行如下几种操作：

> 1. 读取osc本地文件夹下的日志信息
> 2. 如果日志信息为空，服务停止—— `LogUploadService.this.stopSelf()`
> 3. 如果日志信息不位空，上传日志；
 
##二、详细介绍
 
 
###停止服务
 
 在该服务操作中，无论什么情况下都需要停止服务的（日志信息为空、日志上传成功、日志上传失败），停止服务时，使用如下代码：
 
 ```
 LogUploadService.this.stopSelf()
 ```
 关于stopSelf，在Google官方文档中如下描述：
 
 > If a component starts the service by calling startService() (which results in a call to onStartCommand()), then the service remains running until it stops itself with stopSelf() or another component stops it by calling stopService().
 如果一个组件开启通过startService()开启了服务（其实是调用onStartCommand())方法，接着这个Service一直在运行知道它调用stopSelf()停止了自己或者另一个组件通过调用`stopService`方法停止该服务。
 > onStartCommand()
The system calls this method when another component, such as an activity, requests that the service be started, by calling startService(). Once this method executes, the service is started and can run in the background indefinitely. If you implement this, it is your responsibility to stop the service when its work is done, by calling stopSelf() or stopService(). (If you only want to provide binding, you don't need to implement this method.)
当某一个组件比如Activity，通过调用startService()方法来开启一个服务时，系统会调用onStartCommand()方法。一旦这个方法执行之后，服务就会被开启并在后台独立的运行。如果你实现了这个方法，你必须在任务完成后通过调用`stopSelf()`或者`stopService()`来停止该服务。（如果你仅仅只想提供绑定，你不需要实现这个方法）


### 上传日志

日志上传的动作，封装在了OSChinaApi中，借助**android-async-http**完成该操作，代码如下：
 
 ```
 OSChinaApi.uploadLog(data, new AsyncHttpResponseHandler() {
                @Override
                public void onSuccess(int arg0, Header[] arg1, byte[] arg2) {
                    //上传成功，删除本地记录的日志，并且，停止服务
                    log.delete();
                    LogUploadService.this.stopSelf();
                }

                @Override
                public void onFailure(int arg0, Header[] arg1, byte[] arg2,
                        Throwable arg3) {
                    //上传失败停止服务
                    LogUploadService.this.stopSelf();
                }
            });
 ```
 
 这块的封装是非常巧妙的，在OSChinaApi中，进行了如下封装：
 
 > 上传分为：上传日志`uploadLog`和上传反馈意见`feedback`，根据report来进行区分上传的类别
 
 ```
 private static void uploadLog(String data, String report,
            AsyncHttpResponseHandler handler) {
        RequestParams params = new RequestParams();
        params.put("app", "1");
        params.put("report", report);
        params.put("msg", data);
        ApiHttpClient.post("action/api/user_report_to_admin", params, handler);
    }

    /**
     * BUG上报
     * 
     * @param data
     * @param handler
     */
    public static void uploadLog(String data, AsyncHttpResponseHandler handler) {
        uploadLog(data, "1", handler);
    }

    /**
     * 反馈意见
     * 
     * @param data
     * @param handler
     */
    public static void feedback(String data, AsyncHttpResponseHandler handler) {
        uploadLog(data, "2", handler);
    }
 ```
 
 
##三、总结收获
 
 > 1. start 方式开启服务，调用onStartCommand方法，并且在任务结束后要 stop 服务——stopSelf或者stopService
 
 > 2. 上传日志和反馈意见的代码封装 OSChinaApi.java
 
 > 3. android-async-http 的使用


