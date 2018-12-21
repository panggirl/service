# service

#### startService
startService(Intent(this,MyService::class.java)时会调用该 MyService 中的onCreate()和onStartCommand()方法，对次 startService只有onStartCommand(intent: Intent?, flags: Int, startId: Int)方法且 startId 自增;
```
2018-12-18 15:08:48.433 27857-27857/com.dabo.aidlclient I/System.out: MyService onCreate
2018-12-18 15:08:48.433 27857-27857/com.dabo.aidlclient I/System.out: MyService onStartCommand ,flags = 0, startId = 1
2018-12-18 15:08:58.118 27857-27857/com.dabo.aidlclient I/System.out: MyService onStartCommand ,flags = 0, startId = 2
2018-12-18 15:09:00.040 27857-27857/com.dabo.aidlclient I/System.out: MyService onStartCommand ,flags = 0, startId = 3
2018-12-18 15:09:00.788 27857-27857/com.dabo.aidlclient I/System.out: MyService onStartCommand ,flags = 0, startId = 4
2018-12-18 15:09:01.537 27857-27857/com.dabo.aidlclient I/System.out: MyService onStartCommand ,flags = 0, startId = 5
````
startService方法无论启动多少次，onCreate方法只会调用一次，onStartCommand方法将会被调用多次（与startService的次数一致），且系统只会创建一个Service实例（结束该Service也只需要调用一次stopService），该Service会一直在后台运行直至调用stopService或调用自身的stopSelf方法。后台运行的Service系统优先级相对较低，在系统资源不足的情况下，服务有可能被系统结束（kill）；特点：一旦服务开启跟调用者(开启者)就没有任何关系了。开启者退出了，开启者挂了，服务还在后台长期的运行。开启者不能调用服务里面的方法。
 
#### bindService()
bindService(Intent(this, MyService::class.java), connection, BIND_AUTO_CREATE)
bindService 生命周期 ：onCreate() --->onBind()--->onunbind()--->onDestory()，特点：bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉，绑定者可以调用服务里面的方法。

```
// MyService.java
 class MyBinder : Binder() {
          fun  getService():MyService{
              return MyService()
        }

    }
 // XXXActivity.java
 object connection : ServiceConnection {
        override fun onServiceDisconnected(name: ComponentName?) {
            println("onServiceDisconnected name = $name")
        }

        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            println("onServiceConnected name = $name")
            val binder: MyService.MyBinder = service as MyService.MyBinder
            val myService = binder.getService()
        }

    }
```
如果是本地服务时OK，但更改 MyService 为远程服务：
```
<service android:name=".MyService" android:process=":remote" />
```
调用上面的 bindService(Intent(this, MyService::class.java), connection, BIND_AUTO_CREATE) 会崩溃
```
2018-12-18 17:04:37.494 30596-30596/com.dabo.aidlclient E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.dabo.aidlclient, PID: 30596
    java.lang.ClassCastException: android.os.BinderProxy cannot be cast to com.dabo.aidlclient.MyService$MyBinder
        at com.dabo.aidlclient.MainActivity$connection.onServiceConnected(MainActivity.kt:38)
        at android.app.LoadedApk$ServiceDispatcher.doConnected(LoadedApk.java:1818)
        at android.app.LoadedApk$ServiceDispatcher$RunConnection.run(LoadedApk.java:1847)
        at android.os.Handler.handleCallback(Handler.java:808)
        at android.os.Handler.dispatchMessage(Handler.java:101)
        at android.os.Looper.loop(Looper.java:166)
        at android.app.ActivityThread.main(ActivityThread.java:7425)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.Zygote$MethodAndArgsCaller.run(Zygote.java:245)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:921)
```


