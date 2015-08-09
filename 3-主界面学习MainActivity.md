# 开源中国源码学习（四）——主界面学习MainActivity

在AppStart中，我们看到在启动动画结束的时候，程序进行了一次redirectTo。完成了如下任务：

1. Intent to LogUploadService
2. Intent to MainActivity

这篇文章主要学习第二个任务：Intent to MainActivity。

```
Intent intent = new Intent(this, MainActivity.class);
startActivity(intent);
```

**涉及到的知识点**
1. 夜间模式和日间模式的切换
2. ButterKnife的使用
3. DrawerLayout的使用实现侧滑菜单
4. FragmentTabHost的使用及setUp方法的使用
5. IntentFilter的使用
6. BroadcastReceiver-registerReceiver
7. Service及其使用方式——Bind
8. checkUpdate——检查更新
9. AsyncHttpClient、AsyncHttpResponseHandler的使用
10. [enum 类的使用](http://developer.51cto.com/art/201110/295482.htm) http://my.oschina.net/yotoo/blog/266466 
[Java枚举类型enum的使用](http://blog.csdn.net/wgw335363240/article/details/6359614)
11. <intent-filter>
12. 广播的注册与反注册
13. Fragment相关的总结
 
 
 下面将按照上面所列出的点进行总结！



